The current design is tried to match the
[Readonly Collections proposal](https://github.com/tc39/proposal-readonly-collections). But `ArrayBuffer` is not a
a collection so the design won't be the same.

# Design goal

1. Freeze the `ArrayBuffer`.
2. Read-only `TypedArray`/`DataView` to an `ArrayBuffer`.
3. Unescapable slice of `TypedArray`/`DataView` to an `ArrayBuffer`.
4. Not creating new code branches on a hot path when it is unnecessary.
5. Have similar API design with [Readonly collections proposal](https://github.com/tc39/proposal-readonly-collections)

# ArrayBuffer.prototype.freeze()

Note: [issue #9](https://github.com/tc39/proposal-limited-arraybuffer/issues/9), suggests removing this function based on the following reasoning: **If all read-write view to an ArrayBuffer has been GC, the engine can assume the underlying buffer is read-only.** But I'm not favoring this because it is an implicit opt-in.

After calling this function on **ArrayBuffer** _A_:

## Any TypedArray _T_ where it's [[ViewedArrayBuffer]] is _A_

1. [[[GetOwnProperty]]](https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-integer-indexed-exotic-objects-getownproperty-p):
   Return `[[Writable]]: false, [[Configurable]]: false` for existing numeric keys.
2. [[[DefineOwnProperty]]](https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-integer-indexed-exotic-objects-defineownproperty-p-desc):
   Change the behavior to accept descriptor with `[[Writable]]: false` or `[[Configurable]]: false`; Throw when trying to
   define a different `[[Value]]`.
3. [[[Set]]](https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-integer-indexed-exotic-objects-set-p-v-receiver):
   Throw.
4. [[[Delete]]](https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-integer-indexed-exotic-objects-delete-p):
   Throw.

## Any DataView _V_ where it's [[ViewedArrayBuffer]] is _A_

1. **setBigInt64, setBigUint64, setFloat32, setFloat64, setInt8, setInt16, setInt32, setUint8, setUint16, setUint32**:
   Throw.

## Host integration

1. If a Host API need to write an **ArrayBuffer** _A_,
    1. if _A_ is frozen, throws.
    2. Prevent _A_ to be frozen.
    3. Host may re-allow freeze _A_ after the host no longer needs the write access.

## SharedArrayBuffer integration

Possible choices:

1. Throw? (do not allow freeze on SABs).
2. Froze the SAB? (May impact usability).

## ResizableArrayBuffer integration

1. Freeze the **ResizableArrayBuffer**.
2. Throw when trying to resize.

# get ArrayBuffer.prototype.isFrozen()

1. Return **true** if the **this** value has been frozen.

# %TypedArray%.prototype.readOnlyView()

This API design is trying to match the
[Readonly collection proposal](https://github.com/tc39/proposal-readonly-collections#snapshotdivergereadonlyview-methods-for-all-collections).

Note: It provides a read-only view to a possibly mutable **ArrayBuffer**.

1. `%TypedArray%.prototype.readOnlyView()` returns a new `%TypedArray%` object with same **offset** and **length**. (Or
   we can create a new `ReadonlyTypedArray` type for it).
2. If a `%TypedArray%` is already a readonly view, calling `readonlyView()` on it will return it self.
3. `[[GetOwnProperty]]`: Return `[[Writable]]: false, [[Configurable]]: true` for existing numeric keys. (**NOT SAME**
   as `[[GetOwnProperty]]` above.)
4. `[[DefineOwnProperty]]`: Throw.
5. `[[Set]]`: Throw.
6. `[[Delete]]`: Throw.
7. get `%TypedArray%.prototype.buffer`: Return undefine or throw. (If we have `ArrayBufferSlice`, return a new slice that is read-only).
8. Any new `%TypedArray%` created based on this one is also read-only.

# DataView.prototype.readOnlyView()

Same design as `%TypedArray%.prototype.readOnlyView()`.

# ArrayBuffer.prototype.snapshot()

This API design is trying to match the
[Readonly collection proposal](https://github.com/tc39/proposal-readonly-collections#snapshotdivergereadonlyview-methods-for-all-collections).

Clone a new read-only `ArrayBuffer`. If the current `ArrayBuffer` is already read-only, it will return itself.

# %TypedArray%.prototype.snapshot()

This API is required if we do not have `ArrayBufferSlice` because `%TypedArray%.prototype.buffer` will return undefined/throw therefore it's impossible to create a snaphost on a read-only typed array view.

```js
readonlyU8Array.buffer.snapshot()
// Throw: Cannot read property snapshot of undefined
```

# ArrayBuffer.prototype.diverge()

This API design is trying to match the
[Readonly collection proposal](https://github.com/tc39/proposal-readonly-collections#snapshotdivergereadonlyview-methods-for-all-collections).

Clone a new mutable `ArrayBuffer` (even if the current `ArrayBuffer` is read-only).

# %TypedArray%.prototype.diverge()

This API is required if we do not have `ArrayBufferSlice` because `%TypedArray%.prototype.buffer` will return undefined/throw therefore it's impossible to create a diverge on a read-only typed array view.

```js
readonlyU8Array.buffer.diverge()
// Throw: Cannot read property snapshot of undefined
```

## Why not have `readonlyView` on `ArrayBuffer`?

`ArrayBuffer` itself is not a view to a collection. It needs to be viewed with `TypedArray` or `DataView`.

# (Possible) New primitive type

1. New primitive type `arraybuffer`.
2. Always immutable.
3. Can generate from `ArrayBuffer.prototype.toPrimitive()`
4. Has prototype `ArrayBuffer.prototype`.
5. `ToObject` make it an `ArrayBuffer` object.

# (Possible) ArrayBufferSlice

This part is intended to resolve the use case of [issue #11](https://github.com/tc39/proposal-limited-arraybuffer/issues/11).

> One wants the slice to be read-only in order to prevent writing to the memory, and one doesn't want to move around the entire ArrayBuffer object, as that would allow reading into the memory at practically arbitrary locations.

1. Can be created by `ArrayBuffer.prototype.placeholder_name_to_create_a_new_slice(offset, length)`
1. Calling `.freeze()` on an `ArrayBufferSlice` will throw.
1. `%TypedArray%`, `DataView` and host APIs can also accept `ArrayBufferSlice` when `ArrayBuffer` is accepted.
1. Cannot get the wider view based on the `ArrayBufferSlice`.
1. Can create a read-only slice and keep the underlying `ArrayBuffer` mutable.
