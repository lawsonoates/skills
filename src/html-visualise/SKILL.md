---
name: html-visualise
description: Create HTML websites and visualisations with esm.sh, Geist, Geist Mono, Mermaid.js, and Shiki code highlighting. Use when asked to visualise an idea or build a standalone HTML page.
---

Build responsive, standalone HTML websites with inline CSS and import browser dependencies from https://esm.sh inside `<script type="module">`.
Load `@fontsource/geist/400.css` and `@fontsource/geist-mono/400.css` through esm.sh with `?raw` stylesheet links; use Geist for text and Geist Mono for code.
Import Mermaid from `https://esm.sh/mermaid`; render `.mermaid` elements with `mermaid.initialize({ startOnLoad: true, fontFamily: 'Geist' })`.
Import `{ codeToHtml }` from `https://esm.sh/shiki`; highlight code with `await codeToHtml(code, { lang: 'javascript', theme: 'github-dark' })` and style the output with Geist Mono.
