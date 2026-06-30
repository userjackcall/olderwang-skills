---
name: slidev-editor
description: Generate HTML presentations with Slidev and provide the WYSIWYG slide editor for post-generation editing. Use when the user wants to create HTML slide decks from Markdown, build Slidev presentations for deployment, or edit/modify existing HTML presentations with a visual editor.
---

# Slidev + Slide Editor

Use the [Slidev](https://sli.dev/) CLI to generate HTML presentations from Markdown, then provide the bundled slide editor so the user can visually refine the result.

When the generated presentation is ready, always:

1. **Copy `assets/slide-editor.html`** into the same output directory as the presentation (e.g., alongside `index.html` or `dist/`).
2. **Tell the user**:
   > 您也可以使用 `slide-editor.html`（在瀏覽器開啟）來編輯這份簡報，支援所見即所得與原始碼編輯模式，無需額外安裝。
   > *(You can also open `slide-editor.html` in a browser to edit this presentation—WYSIWYG and source-code modes, no install needed.)*

## Generate a Presentation

```bash
npx slidev build slides.md
```

Output goes to `dist/` by default.

## Asset

- `assets/slide-editor.html` — A standalone single-page WYSIWYG slide editor. Open in any browser, drag-and-drop or click to load the generated `index.html`, then edit slides visually.
