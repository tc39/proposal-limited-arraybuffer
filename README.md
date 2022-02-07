# Limited ArrayBuffer

## Status

Champion(s): *[Jack Works](https://github.com/Jack-Works)*

Author(s): *Jack Works*

Stage: 1

## Presentations

- [For stage 1 on 82th tc39 meeting (Apr 2021)](https://docs.google.com/presentation/d/1TGLvflOG63C5iHush597ffKTenoYowc3MivQEhAM20w/edit?usp=sharing)
- [TC39 meeting notes on 82th tc39 meeting](https://github.com/tc39/notes/blob/master/meetings/2021-04/apr-21.md#read-only-arraybuffer-and-fixed-view-of-arraybuffer-for-stage-1)

## Problem to be resolved

All of the following are helpful to archive the minimal permission/information principle.

1. Cannot make an `ArrayBuffer` read-only.
2. Cannot give others a read-only view to the `ArrayBuffer` and keep the read-write permission internally.
3. Cannot give others a view that range limited (only a small area of the whole buffer is visible).

## Design goal

1. Freeze the `ArrayBuffer`.
    1. Like `Object.freeze`, there is no way back once frozen.
    2. Any `TypedArray`/`DataView` to the freezed `ArrayBuffer` is read-only too.
    3. [Optional] Keep frozen when sent across Realm (HTML intergration).
2. Read-only `TypedArray`/`DataView` to a read-write `ArrayBuffer`.
    1. Must not be able to construct a read-write view from a read-only view.
3. [Optional] Range-limited `TypedArray`/`DataView` to a read-write `ArrayBuffer` ([CrimsonCodes0](https://github.com/CrimsonCodes0)'s [use case on WebAssembly](https://github.com/tc39/proposal-limited-arraybuffer/issues/11)).
    1. Must not be able to construct a bigger view range from a smaller view range.
4. Not adding too much complexity to the implementor.

## Pros

1. Minimal permission/information principle works on `ArrayBuffer`.
2. Embedded JS engines can represent ROMs as read-only `ArrayBuffer`.

## API design

See [design.md](./design.md)
