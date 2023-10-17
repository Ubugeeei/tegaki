# API Reference

## @tecack/frontend

### createTecack()

Creates Tecack instance for drawing on canvas.

- #### Type

  ```ts
  function createTecack(document: Document): Tecack;

  interface Tecack {
    init: (id: string) => void | InitializeError;
    deleteLast: () => void | CanvasCtxNotFoundError;
    erase: () => void | CanvasCtxNotFoundError;
    getStrokes: () => Readonly<Array<Stroke>>;
    restoreFromStrokes: (strokesMut: Array<Stroke>) => void;

    // docs is coming soon...
    // redraw: () => void | CanvasCtxNotFoundError;
    // normalizeLinear: () => void;
    // draw: (color?: string) => void | CanvasCtxNotFoundError;
    // copyStuff: () => void;
  }
  ```

- #### Example

  ```ts
  import { createTecack } from "@tecack/frontend";

  const tecack = createTecack(document);
  ```

### Tecack.init()

Mounts Tecack instance to canvas element.

- #### Type

  ```ts
  function init(id: string): void | InitializeError;
  ```

- #### Example

  ```ts
  import { createTecack, InitializeError } from "@tecack/frontend";

  const tecack = createTecack(document);
  const res = tecack.init("my-canvas");
  if (res instanceof InitializeError) {
    // handle error
  }
  ```

### Tecack.deleteLast()

Deletes last stroke from canvas and instance internal data.

- #### Type

  ```ts
  function deleteLast(): void | CanvasCtxNotFoundError;
  ```

- #### Example

  ```ts
  import { createTecack, CanvasCtxNotFoundError } from "@tecack/frontend";

  const tecack = createTecack(document);
  tecack.init("my-canvas");

  const deleteLastButton = document.getElementById("delete-last");
  deleteLastButton.addEventListener("click", () => {
    const res = tecack.deleteLast();
    if (res instanceof CanvasCtxNotFoundError) {
      // handle error
    }
  });
  ```

### Tecack.erase()

Erases all strokes from canvas and instance internal data.

- #### Type

  ```ts
  function erase(): void | CanvasCtxNotFoundError;
  ```

- #### Example

  ```ts
  import { createTecack, CanvasCtxNotFoundError } from "@tecack/frontend";

  const tecack = createTecack(document);
  tecack.init("my-canvas");

  const eraseButton = document.getElementById("erase");
  eraseButton.addEventListener("click", () => {
    const res = tecack.erase();
    if (res instanceof CanvasCtxNotFoundError) {
      // handle error
    }
  });
  ```

### Tecack.getStrokes()

Get strokes data from instance internal.  
You can use this data for restore strokes or backend recognition.

- #### Type

  ```ts
  function getStrokes(): Readonly<Array<Stroke>>;
  ```

- #### Example

  ```ts
  import { createTecack } from "@tecack/frontend";

  const tecack = createTecack(document);
  tecack.init("my-canvas");

  const strokes = tecack.getStrokes();
  console.log(strokes);
  ```

  ```ts
  import { recognize } from "@tecack/backend";

  const strokes = tecack.getStrokes();
  const candidates = recognize(strokes);
  ```

### Tecack.restoreFromStrokes()

Restores canvas strokes and internal data from existing strokes data.

- #### Type

  ```ts
  function restoreFromStrokes(strokesMut: Array<Stroke>): void;
  ```

- #### Example

  ```ts
  const strokes = JSON.parse(localStorage.getItem("strokes"));
  tecack.restoreFromStrokes(strokes);
  ```

## @tecack/backend

### recognize()

Recognizes strokes data and returns candidates.

- #### Type

  <!-- prettier-ignore -->
  ```ts
  function recognize(
    input: Readonly<Array<Stroke>>, 
    dataset: Readonly<Array<TecackStroke>>
  ): string[];
  ```

- #### Example

  ```ts
  import { recognize } from "@tecack/backend";
  import { KANJI_DATASET } from "@tecack/dataset";

  const candidates = recognize(strokes, [...KANJI_DATASET, ...myDataset]);
  ```

## @tecack/shared

### encodeStroke() / decodeStroke()

Encodes `Array<TecackStroke>` data to minimize size.  
And, decodes encoded string.

- #### Type

  ```ts
  function encodeStroke(strokes: Array<Stroke>): string;
  function decodeStroke(encoded: string): Array<Stroke> | DecodeError;
  ```

- #### Example

  ```ts
  // on client
  import { encodeStroke } from "@tecack/shared";

  const recBtn = document.getElementById("recognize-btn");
  recBtn.addEventListener("click", async () => {
    const strokes = tecack.getStrokes();
    const encoded = encodeStroke(strokes); // here
    const res = await fetch("http://my-api.dev/recognize", {
      method: "POST",
      body: JSON.stringify({ strokes: encoded }),
    });
  });
  ```

  ```ts
  // on server
  import { decodeStroke } from "@tecack/shared";
  import { recognize } from "@tecack/backend";
  import { KANJI_DATASET } from "@tecack/dataset";

  // ex1: recognize
  app.post("/recognize", async c => {
    const req = await c.req.json();
    const decoded = decodeStroke(req.strokes); // here
    if (decoded instanceof DecodeError) {
      return c.json({ error: decoded.message });
    }
    const candidate = recognize(decoded, KANJI_DATA_SET);
    return c.json(candidate);
  });

  // ex2: save to database as encoded text data
  app.post("/save-stroke", async c => {
    const req = await c.req.json();
    const decoded = decodeStroke(req.strokes); // here
    if (decoded instanceof DecodeError) {
      return c.json({ error: decoded.message });
    }
    await myDb.save({ decoded }); // save to database as encoded text data
    return c.json({ ok: true });
  });
  ```