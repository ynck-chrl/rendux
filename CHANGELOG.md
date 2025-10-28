# Changelog

## v0.93.2

- fixed nested x-for rendering

## v0.93.1

- x-key diffing algorithm is now reusing DOM nodes thus greatly optimzing rendering (up to 50% faster!)

## v0.93.0

- remove plugins, focusing on core library
- Fixed state and rendering target coupling, now state and rendering target element are completely decoupled. The function requires both state and target now : rendux(state, element)

## v0.92.0

- Mini Evaluator is now directly embeded in `rendux`

## v0.91.0

- Added conditional logging system for plugins and core features
- Removed x-click-outside plugin
- Fixed plugin attribute handling and re-rendering
- Improved plugin logging control via `logs` attribute

## v0.86.0

- Inlined plugin core into `rendux.js`, removing dependence on external raw-render-core.js.
- Added chainable plugin API: `.use()`, `.process()`, `plugins`, and `parsePluginCall` directly on `rendux`.
- Fixed plugin attribute lookup to avoid invalid CSS selector issues for `render.plugin`.

## v0.84.0

- Hidden elements (`x-if`) now reside in an in-memory `DocumentFragment` instead of a `<template>`, preventing hidden nodes from cluttering the DOM or shadow DOM.
- `<template x-for>` loops are now tracked from both the live DOM and the hidden fragment for correct re-rendering.
