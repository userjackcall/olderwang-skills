# olderwang-skills

老王自製的 OpenCode 技能集合，用 AI 加速教學備課工作流程。

## 技能列表

### [slidev-editor](./slidev-editor/)

**HTML 簡報產生及編輯器** — 從 Markdown 產生 Reveal.js 簡報，附帶獨立的所見即所得（WYSIWYG）編輯器，可在瀏覽器中直接載入、編輯、預覽簡報。

- 三種編輯模式：視覺編輯 ↔ 原始碼 ↔ 基本預覽
- 格式化工具列、復原／重作、插入圖片（可拖曳縮放）
- 側欄投影片列表 + 拖曳排序
- 全螢幕預覽（從當前投影片開始播放）
- 支援載入任意 .html 簡報檔

### [tech-high-inquiry-lesson-plan](./tech-high-inquiry-lesson-plan/)

**技術型高中自然科學「探究與實作」教案產生器** — 將教學主題或既有教案，自動轉換為符合台灣 108 課綱的正式教案，並填入教育部技高自然科推動中心的官方 Word 範本。

- 支援物理、化學、生物（A 版 / B 版）
- 9 步驟分析流程：三面九項 → 核心素養 → 學習內容 → 議題融入 → 學習表現 → 學習目標 → 學習過程 → 教學歷程 → 產出 docx
- 可選補充教材：WSQ 學習單、學習表現檢核表、學習評量＋評分規準
- 逐步確認／一次產出兩種模式

---

## 安裝方式

將此 repo clone 到專案的 `.opencode/skills/` 目錄下即可啟用所有技能：

```bash
cd 你的專案/.opencode/skills
git clone https://github.com/userjackcall/olderwang-skills.git
```

或者在 `opencode.json` 中引用：
```json
{
  "skills": ["olderwang-skills"]
}
```

## 貢獻

目前由老王一人維護。歡迎開 issue 或 PR 提出建議！
