# Freeze ArrayBuffer and Readonly view to ArrayBuffer

## Motivation

TBD. Performance & security. Maybe intergrate with Record & Tuple?

## Freeze ArrayBuffer

Add a `ArrayBuffer.prototype.freeze()` to freeze any future modification on the ArrayBuffer. Any view to the ArrayBuffer becomes readonly.

### Example

```js
const buffer = new ArrayBuffer(8);
const view = new Int32Array(buffer);

view[0] = 42 // OK
buffer.freeze();

view[0] = 42 // TypeError
```

## Readonly view to ArrayBuffer

Add a new option to the `TypedArray` and `DataView` constructor (or `.prototype.freeze()`). To create a readonly view.

Note: Readonly view can points to a mutable ArrayBuffer.

To avoid re-create mutable view of the ArrayBuffer via `typedArray.buffer`, it will be special handled.

Replace `[[ViewedArrayBuffer]]` with a new ArrayBuffer but points to the same `[[ArrayBufferData]]` ``[[ArrayBufferByteLength]]`` and ``[[ArrayBufferDetachKey]]``. (Need to be careful when any of the internal slot has modified (detached or resized)).

### Example

```js
const buffer = new ArrayBuffer(8);
// or roView.freeze()
const roView = new Int32Array(buffer, { readonly: true });
const rwView = new Int32Array(buffer);

roView[0] = 42 // TypeError
rwView[0] = 42 // OK

const tryEscape = roView.buffer
const newView = new Int32Array(tryEscape)
newView[0] = 42 // still TypeError

roView.buffer !== rwView
```
