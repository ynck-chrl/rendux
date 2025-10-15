## Changelog

### v0.92
- Mini Evaluator is now directly embeded in `renderx`

### v0.91
- Added conditional logging system for plugins and core features
- Removed x-click-outside plugin
- Fixed plugin attribute handling and re-rendering
- Improved plugin logging control via `logs` attribute

### v0.86
- Inlined plugin core into `renderx.js`, removing dependence on external raw-render-core.js.
- Added chainable plugin API: `.use()`, `.process()`, `plugins`, and `parsePluginCall` directly on `renderx`.
- Fixed plugin attribute lookup to avoid invalid CSS selector issues for `render.plugin`.

### v0.84
- Hidden elements (`x-if`) now reside in an in-memory `DocumentFragment` instead of a `<template>`, preventing hidden nodes from cluttering the DOM or shadow DOM.
- `<template x-for>` loops are now tracked from both the live DOM and the hidden fragment for correct re-rendering.