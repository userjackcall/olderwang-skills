# slidev-editor

## 用途

用 [Slidev](https://sli.dev/) CLI 從 Markdown 產生 HTML 簡報，並附帶獨立的 WYSIWYG 所見即所得編輯器，方便後續修改。

## 使用流程

1. 建立 Slidev Markdown 簡報檔（`slides.md`）
2. 執行 `npx slidev build slides.md` 產生 HTML（預設輸出 `dist/`）
3. 將 `assets/slide-editor.html` 複製到輸出目錄
4. 提醒使用者：可用瀏覽器開啟 `slide-editor.html`，載入簡報檔後以視覺化模式編輯

## 觸發時機

- 使用者要求建立 HTML 簡報／投影片
- 使用者希望用 Slidev 製作並部署簡報
- 使用者需要編輯已存在的 HTML 簡報

## 結構

```
slidev-editor/
├── SKILL.md                    # 技能主說明
├── agents/openai.yaml          # UI 後設資料
├── README.md                   # 本檔案
└── assets/
    └── slide-editor.html       # 獨立 WYSIWYG 編輯器（單頁 HTML，無需安裝）
```

## 產出範例

```
專案/
├── dist/
│   └── index.html              # Slidev 產生的簡報
├── slides.md                   # 原始 Markdown
└── slide-editor.html           # 編輯器（手動複製至此）
```

## 注意事項

- 編輯器為純前端工具，在瀏覽器中直接開啟即可使用
- 支援拖曳或按鈕載入 .html 簡報
- 提供視覺編輯／原始碼／基本預覽三種模式