---
name: tech-high-inquiry-lesson-plan
description: Converts a teaching topic, raw content, or an existing lesson plan into a 108課綱-compliant "探究與實作" (Inquiry and Practice) lesson plan for Taiwan technical/vocational senior high school (技術型高中) Natural Science (自然科學領域) courses, formatted to match the official 教育部技術型高級中等學校自然科學領域推動中心 blank template. Use whenever the user mentions 探究與實作教案, 108課綱, 技術型高中, 自然科學領域, 核心素養三面九項, 學習內容/學習表現, 議題融入, or asks to "改版"/"轉換"/"設計" a science lesson plan into this format — even from just a topic or informal draft. Also use for follow-ups on an already-generated plan of this kind, e.g. adding a WSQ學習單, 學習表現檢核表, or 學習評量/評分規準 into the same docx. Applies to 物理/化學/生物/地球科學 at the technical high school level; chemistry is the calibration example, but the workflow generalizes to all subjects.
---

# 技術型高中自然科學領域「探究與實作」教案產生器

將任意教學主題、教學內容、或既有教案，比對台灣108課綱技術型高中自然科學領域課程綱要，轉換成教育部技高自然科推動中心的正式「探究與實作」教案格式（填入官方空白教案 docx 範本）。

## 必讀的綁定資源

This skill bundles three reference files in `assets/` — **always use these bundled copies directly, do not ask the user to re-upload them** unless the user explicitly provides a newer/different version:

- `assets/All-技術型高中自然科學領域課程綱要.pdf` — the official curriculum guideline (covers 物理/化學/生物/地球科學 all subjects, both A版 and B版). ~140 pages. Use `pdftotext -layout` + `grep` to search it (see references/workflow.md for proven search patterns); do not try to read the whole thing into context at once.
- `assets/台灣_108_課綱19項議題融入.pdf` — the authoritative reference for all 19 議題融入 topics under 108 課綱, with 內涵 and 學習重點方向 for each. **Always consult this file for Step 4 (議題融入)** — it supersedes the curriculum PDF's appendix, which only details 4 of the 19 issues. See references/workflow.md Step 4 for the complete extracted content and selection guidance.
- `assets/探究與實作空白教案.docx` — the official blank lesson plan template to fill in.

Also bundled: `scripts/docx_helpers.py` — battle-tested low-level XML helpers for safely editing this specific template's tables (handles a python-docx merged-cell gotcha described below). **Use this script rather than reinventing table editing from scratch.**

Before touching the docx, read `references/template_structure.md` — it documents the exact table index, row count, and merge structure of every table in the blank template, reverse-engineered from the real file. Skipping this and guessing the structure wastes enormous effort and risks silent corruption (see the gotcha below).

## Critical gotchas

### 1. python-docx + vertically-merged cells

The blank template has several tables with vertically-merged cells (e.g. the 面向 column in the 核心素養具體內涵 table, and the 學習重點/核心素養 cells in the 學習過程 table). **`table.cell(row, col)` in python-docx resolves merged cells to their master cell** — calling it on any row within a merge silently returns/writes to the SAME underlying XML element as every other row in that merge, which causes content from later edits to silently overwrite earlier ones in seemingly unrelated table rows (including the header!). Always use the raw-XML helpers in `scripts/docx_helpers.py` (`get_tr_list`, `get_tc`, `set_tc_text`, `set_tc_vmerge`, `duplicate_tr`, `remove_tr`) which operate on `<w:tr>`/`<w:tc>` elements directly by row index, bypassing this resolution entirely.

### 2. Header paragraph text is BIG5-encoded → cannot match Chinese text by string content

The bundled blank template was authored in a BIG5 environment. When `python-docx` reads the file, all CJK characters appear as garbled Unicode (e.g. `'1.�椸�D�D�G'` instead of `'1.單元主題：'`). **Attempting to find header runs by their Chinese text (e.g. `if '單元主題' in r.text`) will silently fail** — the string will never match.

**Workaround**: identify header paragraphs and their runs by **paragraph index** (0-based in `doc.paragraphs[]`) and **run index** (0-based in `p.runs[]`), then write directly to those runs by index. Do NOT try to match text content.

The mapping for the bundled template is documented in `references/template_structure.md` (Header + Paragraph Index Map section). If the template is ever replaced, re-verify by printing `repr(r.text)` for each paragraph's runs.

### 3. Table6 (學習表現三項目十一子項) — must shade selected rows, NOT just leave it alone

Table6 is pre-printed static text and must NOT have its text edited. However, **you must actively shade selected sub-item rows in yellow (FFE699)** to indicate which 學習表現 apply to this lesson — identical in spirit to how table0 (三面九項) requires shading selected competency cells. This step was initially missed because table6 was incorrectly documented as "STATIC, do not edit" (implying no action needed), but the correct behaviour is: **text = do not edit; shading = always apply**. Leaving it unshaded makes ALL sub-items appear unselected, which is wrong.

The table6 col0 (項目) column also has vMerge chains across each group, so you must use `get_tr_list`/`get_tc` here too — not `table.cell()`. Use the `highlight_table6` function documented in `references/template_structure.md` (Table 6 section), which handles both the col1 (子項) shading and the col0 (項目) label shading for any group that has at least one selected member. Call this **after** filling tables 7-9 (which establish which sub-items are selected) and **before** saving the docx.

## Workflow

Ask the user (if not already specified) which mode they want, using `ask_user_input_v0`:
- **逐步確認模式 (interactive, default)**: walk through the 9 steps one at a time, presenting each step's output and waiting for explicit confirmation/edits before continuing. This is the original, tested flow — prefer it for first-time users or anything that benefits from collaborative refinement (core competency selection, issue selection, time allocation are all judgment calls worth checking).
- **一次產出模式 (fast)**: silently run all 9 analytical steps internally, then present the complete draft (all step outputs) in one message for a single round of feedback before generating the docx.

Both modes follow the same 9 analytical steps — see `references/workflow.md` for the full step-by-step methodology (what each step must produce, how to search the curriculum PDF for it, and worked chemistry examples to calibrate depth/format against). Summary of the 9 steps:

1. Identify which of the 總綱核心素養三面九項 (e.g. A2, B1, C2...) apply to the topic.
2. Pull the official 總綱核心素養項目說明 + 自然科學領域核心素養具體內涵(V.2-U) full text for each selected item, from the curriculum PDF.
3. Identify matching 學習內容 codes (主題/次主題/代碼) from the curriculum, write a content-specific 學習內容說明, a 主要概念 (title + brief), and a 參考節數 for each.
4. Find a matching 議題融入 (e.g. 科技教育, 性別平等教育, 環境教育...) with 主題/實質內涵/融入課程綱要學習重點 — search the curriculum PDF's appendix first; if the topic's natural issue isn't covered there with full detail, it's fine to use a generally-known issue-integration code (like 生活科技領域 codes such as 生P-V-1) the way the official chemistry-teacher example does, but flag this transparently to the user rather than fabricating codes.
5. Map each Step-3 學習內容 to the 學習表現三項目十一子項 (from the curriculum's V.2 performance table) with a custom 檢核點.
6. Write 4-5 學習目標 and at least 4 學習脈絡 (typically: 發現問題/規劃研究/論證建模/表達分享).
7. Synthesize into 學習過程 fields: 主題名稱/教學時間/教材來源/教學資源/教學準備/教學說明/學習重點(學習內容+學習表現+核心素養)/情境規劃.
8. Design 教學歷程 stages covering the full class-period count, each with 教學時間/教學資源/教學評量 (evaluations should cite the Step-5 檢核點 codes where possible).
9. Fill all of the above into the bundled blank docx (see references/template_structure.md for exactly which table/row maps to which step). After filling tables 7-9, call `highlight_table6` (documented in template_structure.md Table 6 section) to shade the selected 學習表現子項 rows — this is NOT optional. Then validate visually, save with a descriptive filename, and present it.

## Optional supplementary materials

After the core 9-step lesson plan docx is done and confirmed, the user may separately ask for any of these — generate only what's asked for, working off the **already-completed** docx (don't regenerate the lesson plan itself):

- **WSQ學習單** (Watch–Summarize–Question worksheet) — a fill-in-the-blank student worksheet built around the single most calculation/reasoning-heavy 教學歷程 stage.
- **學習表現檢核表** — a 3-tier (精熟/良好/待加強) teacher observation rubric, one row per existing 學習表現檢核點.
- **學習評量＋評分規準** — a student quiz (single-choice + short-answer) plus a teacher answer key with per-question 詳解/評分尺度/設計理由.

Full methodology, content-selection heuristics, and implementation steps are in `references/supplementary_materials.md` — read it before generating any of these. Use `scripts/append_helpers.py` (page breaks, formatted paragraphs, bordered tables matching the template's visual style) to append each as new pages at the end of the docx; this is safe to mix with `scripts/docx_helpers.py`'s raw tr/tc functions for filling table cells, since brand-new tables have no pre-existing merges. As always, validate the new pages visually (PDF render) before delivering.

## Generating the docx (Step 9 mechanics)

1. Copy `assets/探究與實作空白教案.docx` to a scratch working directory — never edit the bundled copy in place.
2. **Header paragraphs**: due to a BIG5 encoding issue in the template (see Critical Gotcha #2), **do NOT attempt to find header runs by matching Chinese text** — it will silently fail. Instead, use the **paragraph-index + run-index** approach documented in `references/template_structure.md` (Header + Paragraph Index Map). For all table editing, open the file with `python-docx` and use `scripts/docx_helpers.py`'s raw tr/tc functions — never `table.cell()` for tables that have merges (table indices 1, 6, and 9; see template_structure.md).

   Example header filling snippet (see template_structure.md Header section for the full index map):
   ```python
   def set_run(p_idx, r_idx, text):
       p = doc.paragraphs[p_idx]
       if r_idx < len(p.runs):
           p.runs[r_idx].text = text

   set_run(4, 1, '單元主題：光的干涉探究與實作——從雙狹縫到精密量測')
   set_run(5, 2, '教學對象：技術型高中二年級')
   set_run(6, 5, '物理')
   set_run(6, 9, '學分數 2')
   set_run(7, 4, '物理')
   set_run(7, 6, 'B')
   set_run(8, 3, '二年級')
   set_run(8, 6, '上學期  5節')
   set_run(18, 6, '物理')
   set_run(18, 8, 'B')
   set_run(23, 1, '課堂主題：光的干涉探究')
   set_run(24, 2, '5小時')
   set_run(25, 1, '對應之 物理 ')
   set_run(25, 3, '科綱要學習內容：PKa-V.2-8 光的干涉...')
   ```
3. Adjust each table's row count to match the number of items you actually have (duplicate or remove `<w:tr>` elements with `duplicate_tr`/`remove_tr`) — do not leave unused placeholder rows, and do not silently drop content because there are "only" N template rows. The instructions explicitly allow adjusting row counts.
4. After filling, sanity-check the row/column totals add up (e.g. 參考節數 should sum to the total 單元節數).
4a. **Call `highlight_table6_in_doc`** (from `scripts/highlight_table6.py`) to shade the selected 學習表現子項 rows in table6. This is a required step that is easy to forget because the table text is pre-printed and looks "done" without any shading — but leaving it unshaded makes all sub-items appear unselected. The easiest approach is to use `collect_selected_from_table7(doc)` (same module) which auto-reads the filled table7's 子項 column and returns the correct set, then passes it straight to `highlight_table6_in_doc(doc, selected)`:

```python
from highlight_table6 import highlight_table6_in_doc, collect_selected_from_table7
selected = collect_selected_from_table7(doc)   # auto-detect from table7
highlight_table6_in_doc(doc, selected)          # shade table6 accordingly
doc.save(out_path)
```
5. **Always validate visually before delivering**: convert to PDF (`soffice --headless --convert-to pdf`) and view at least the first 2-3 pages and the last page as images (`pdftoppm -png -r 100`) to confirm merged cells, shading, and table boundaries rendered correctly and nothing overflowed or got corrupted. This step caught a real merged-cell corruption bug during development — do not skip it.
6. Save to `/mnt/user-data/outputs/` with a descriptive filename based on the lesson topic (e.g. `<主題>－探究與實作教案.docx`), then `present_files`.

## Notes

- If the user pastes raw/messy source content (e.g. an existing informal lesson plan PDF) instead of a clean topic, treat that content as the GEM.txt-style "教學主題或內容或教案" input — extract the substantive teaching content from it before running Step 1.
- If 節數 arithmetic doesn't cleanly sum (a common error risk when content spans many class sessions), recompute carefully and explain any correction transparently rather than silently leaving a mismatch — this happened during development (a verbal step-3 summary said 8節 when the table only summed to 7.5節) and was caught and fixed at the docx-generation stage; catch this kind of error as early as possible instead.
- Stay within the bundled curriculum PDF's actual content for codes and quoted text — never invent 課綱 code numbers or descriptions. For **議題融入 (Step 4)**, the 19-issue reference (`assets/台灣_108_課綱19項議題融入.pdf`) is now the primary source — its 內涵 and 學習重點方向 text may be quoted directly for the 學習內涵 column. The curriculum PDF's appendix (which only details 4 of the 19 issues at code level) is a secondary source for subject-specific 實質內涵 codes (e.g. 性U8, 人E6) when available; if such subject-specific codes are not found in the appendix, use the 19-issue PDF's general 學習重點方向 text and note this transparently.
