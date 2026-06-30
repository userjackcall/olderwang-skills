# 補充教材生成：WSQ學習單／學習表現檢核表／學習評量＋評分規準

These three supplementary materials are **optional, on-demand additions** to an already-completed lesson plan docx (the output of the core 9-step workflow in workflow.md). They are not part of the original GEM.txt 9-step process — they were added in response to follow-up user requests, each one separately, after the main lesson plan was finished and confirmed. Treat them the same way: only generate one when the user explicitly asks for it, working off the **already-completed** docx rather than regenerating the whole lesson plan.

All three share a common implementation pattern: open the existing lesson-plan docx with python-docx, use `scripts/append_helpers.py` to add a page break and new paragraphs/tables at the **end** of the document (safe — brand new content has no merge-resolution risk, unlike editing the original template's tables), save, then **always validate visually** (convert to PDF, render the new pages as images, inspect) before delivering. Each addition should pull its substance from content the lesson plan **already established** (教學歷程, 教學評量, 學習表現檢核點) rather than inventing new pedagogical content from scratch — these are derivative materials, not a fresh design pass.

## 1. WSQ 學習單 (Watch–Summarize–Question worksheet)

**What it is**: a single-page (or two-page) fill-in-the-blank worksheet for students, structured in three labeled sections — 👁 W (Watch) 觀察與擷取, 📝 S (Summarize) 歸納與計算, Q (Question) 提問與反思 — built as plain formatted paragraphs (bold section headers + 【任務說明】 + numbered fill-in items with blanks made of full-width underscores ＿＿＿), not a table grid. This mirrors the official exemplar plan's own WSQ worksheet structure exactly.

**How to choose the content**: do NOT try to cover the whole lesson. Pick the **single 教學歷程 stage** with the clearest "observe data → apply a rule/formula → reach a conclusion" structure (in the calibration example, this was the 電子排列規則 + VSEPR分子形狀推論 stage — analogous to how the official circuit-lesson exemplar built its WSQ entirely around the Ohm's-law resistor calculation stage, not the whole 4-stage lesson). Structure:
- **W**: 2-3 fill-in items asking students to record/observe specific facts or data presented by the teacher (e.g. a rule's parameters, given quantities) — these are inputs to the S section's reasoning, not yet any computation.
- **S**: (1) a conceptual "why" fill-in question, then (2) a step-by-step application of the rule/theory to a specific worked case using the W-section data, ending in a concrete decision/result (mirrors the official example's "選用接近的＿＿＿Ω標準電阻" engineering-decision closer).
- **Q**: an open reflection question with 2-3 checkbox-style direction prompts (☐) plus a blank-line area for the student's own question.

## 2. 學習表現檢核表 (teacher-facing observation rubric)

**What it is**: a 4-column table (檢核項目「學習表現檢核點」/ 精熟「表現優異」/ 良好「達成預期」/ 待加強「需要加強」) for the teacher to fill in during class observation, with a 班級/姓名/組別 header line.

**How to generate it**: take the lesson plan's **學習表現檢核點 table verbatim, one rubric row per existing 檢核點** (do not subset or merge them — if the core plan has 10 檢核點, the rubric has 10 rows). For each, expand the single 檢核點 sentence into three concrete, behaviorally-distinct tiers:
- 精熟: independent + correct + goes beyond the minimum (extra reasoning, helping peers, connecting to broader concepts)
- 良好: meets the bar with minor gaps or needs occasional prompting
- 待加強: cannot perform the task independently or shows a clear misconception

Keep each tier description concrete and observable (what the teacher would actually see/hear), not just a vaguer/stronger restatement of the same sentence. Use `make_bordered_table` with roughly `[3200, 2200, 2200, 2147]` dxa column widths (wider item column, three even-ish rating columns) for a 4-column table fitting the page; smaller font (20pt half-points) in the body cells keeps the table from overflowing.

## 3. 學習評量＋評分規準 (assessment + scoring rubric)

**What it is**: two parts, each starting on its own page —
1. A clean student-facing quiz page: 一、單選題 (N items, each with inline A/B/C/D options) then 二、簡答題 (M items).
2. A teacher-facing 評量參考答案與評分規準 page repeating each question and adding: for MC, 解答 / 詳解 / 評分尺度 / 設計理由; for short-answer, 解答參考方向 / 評分尺度與給分標準 (a bulleted multi-tier point scale, e.g. 4 tiers summing to the question's point value) / 設計理由.

**How to choose content** (the default ask is "N單選題＋M簡答題", adapt N/M to what's requested):
- **Spread the MC questions across different 教學歷程 stages/content areas** — breadth, not depth in one spot. Pick one fact/concept/skill per stage so the quiz samples the whole lesson rather than clustering on whichever stage has the most quantitative content (that's what the WSQ is for).
- **Short-answer questions should test transfer, not recall.** A strong pattern (used in the calibration example): one question applies a taught principle to a **new but structurally related scenario** the lesson didn't directly work through (e.g. lesson taught VSEPR via methane, quiz asks about water — this catches students who memorized the specific answer vs. those who understood the principle). A second question ties back to the lesson's special pedagogical framing (e.g. an issue-integration theme like 科技教育, or a cross-domain synthesis prompt) rather than re-testing plain content recall.
- **設計理由 should name what's actually being checked** (e.g. "確認學生掌握化學活性的根本來源而非僅記憶週期表位置" / "檢核學生能否將理論從課堂示範案例遷移應用到未曾直接操作過的情境") — this is what makes the assessment feel like real exemplar-quality work rather than generic quiz boilerplate.
- Point totals: a clean split such as 10分×5MC=50 + 25分×2短答=50 (total 100) is a reasonable default if the user doesn't specify weighting; adjust to match N/M actually requested.

## Implementation checklist

1. Copy the current lesson-plan docx to a scratch path — never edit in place.
2. `from append_helpers import add_p, page_break, make_bordered_table, style_header_row, fill_table_rows` (also needs `docx_helpers.py` alongside it on the path, since `style_header_row`/`fill_table_rows` import from it).
3. Build the section(s) with `page_break(doc)` before each new page-level section.
4. Save, convert to PDF (`soffice --headless --convert-to pdf`), render just the new pages (`pdftoppm -png -r 100 -f <start> -l <end>`), and visually inspect before overwriting the delivered docx.
5. Overwrite the same output filename (these are additive revisions to one ongoing lesson-plan document, not new files) and re-`present_files`.
