# Freeze ArrayBuffer and Readonly view to ArrayBuffer

## Motivation

### Security

We can have frozen objects (via `Object.freeze`) but not for binary data today.

### Performance

-   Developers might copy the whole ArrayBuffer if they want to prevent external code to modify that ArrayBuffer. The copy brings the performance lost.
-   Engines can safely share the memory across different Realms/processes if the ArrayBuffer is read-only.

## Target

1. Add a new way to freeze the ArrayBuffer.
    1. One-way. Once it froze, there is no way back.
    2. All views to the frozen ArrayBuffer are read-only too.
    3. If it is sent across Realms/processes, it is still frozen.
2. Add a new way to create a read-only view to a read-write ArrayBuffer.
    1. Cannot construct the read-write view from `readOnlyView.buffer`

## Possible API design

### Freeze ArrayBuffer

```js
const buffer = new ArrayBuffer(8)
const view = new Int32Array(buffer)

view[0] = 42 // OK
buffer.freeze()

view[0] = 42 // TypeError
```

### Readonly view to ArrayBuffer

<!-- Replace `[[ViewedArrayBuffer]]` with a new ArrayBuffer but points to the same `[[ArrayBufferData]]` ``[[ArrayBufferByteLength]]`` and ``[[ArrayBufferDetachKey]]``. (Need to be careful when any of the internal slot has modified (detached or resized)). -->

```js
const buffer = new ArrayBuffer(8)

// This one?
function createROInt32Array(buffer) {
    return new Int32Array(buffer, { readonly: true })
}
// This one?
function createROInt32Array(buffer) {
    const view = new Int32Array(buffer)
    view.freeze()
    return view
}
// Or this one?
function createROInt32Array(buffer) {
    const view = new Int32Array(buffer.frozenView)
    // view.buffer === buffer.frozenView
    // view.buffer.buffer === view.buffer
    return view
}

const roView = createROInt32Array(buffer)
const rwView = new Int32Array(buffer)

roView[0] = 42 // TypeError
rwView[0] = 42 // OK

const tryEscape = roView.buffer
const newView = new Int32Array(tryEscape)
newView[0] = 42 // still TypeError

roView.buffer !== rwView
```

## Integrate with Tuple & Records?

```js
const buffer = new ArrayBuffer(8)
fillBuffer(buffer)
buffer.freeze()

const data = #{
    binary: buffer
    // yay!
}
```
