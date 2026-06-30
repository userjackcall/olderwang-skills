# 空白教案範本結構地圖（探究與實作空白教案.docx）

Reverse-engineered from the actual bundled file via `python-docx`'s `doc.tables` list (0-indexed) plus raw XML inspection. Re-verify with the snippet at the bottom if the bundled template is ever replaced — table indices/row counts are NOT guaranteed to stay identical across template versions.

## Header (plain paragraphs, not a table)

**⚠️ BIG5 encoding limitation**: The blank template was authored in a BIG5 environment. When python-docx reads it, all CJK characters appear as garbled Unicode (e.g. `'1.�椸�D�D�G'` instead of `'1.單元主題：'`). **You CANNOT find runs by matching Chinese text** — string comparison will silently fail.

**Workaround**: identify header paragraphs by their **0-based paragraph index** (`doc.paragraphs[pi]`) and runs by their **0-based run index** (`p.runs[ri]`), then write directly. Use `repr(r.text)` to debug. The index mapping below is stable for the current bundled template — re-verify if the template is ever replaced.

### Paragraph Index Map

| Pidx | Content | Key runs to edit |
|------|---------|-----------------|
| 0 | 教育部 header line | (do not touch) |
| 2 | `編寫人： 學校：          ` | run0=label, run2=label, runs3-6=fill spaces |
| 3 | Checkboxes line | run0=□素養, run3=□議題, run6=□跨科, run11=■探究 |
| 4 | `1.單元主題：` | run0=`1.`, run1=fill topic after `單元主題：` |
| 5 | ` 2.教學對象：   ` | run0=` `, run1=`2.`, run2=fill after `教學對象：`, run3=clear |
| 6 | `3.科目/學分：    科目     學分數` | run0=`3.`, run1=`科目`, run3=`學分：`, run5=fill subject name, run9=`學分數 N` |
| 7 | `4.課程類別：■部定必修：__ __□校必□校選□彈性學習時間` | run2=`■` checkbox, run3=label, run4=fill version (e.g. `物理`), run5=clear, run6=fill sub-version (`B`), run7=`版`, run9/12/15=□/■ checkboxes |
| 8 | `5.單元節數： 年級  學期節數` | run1=label, run3=fill grade (e.g. `二年級`), run6=fill `上學期  N節` |
| 9 | `6.單元目標` | label only (pre-printed sub-items at P10/P13/P15/P17) |
| 18 | `科技教育議題之主題及學習內容融入課綱_ _版學習重點` | run0/1=issue name, run6/8=fill version `物理`/`B`, run7=clear |
| 23 | `1.課堂主題：` | run1=fill after `課堂主題：` |
| 24 | `2.教學節數：小時` | run2=fill `N小時` (replace `小時`) |
| 25 | `3.對應之      科綱要學習內容：` | run1=fill `對應之 物理 `, run3=fill `科綱要學習內容：...` |

### Editing recipe (python)

```python
def set_run(p_idx, r_idx, text):
    p = doc.paragraphs[p_idx]
    if r_idx < len(p.runs):
        p.runs[r_idx].text = text

# Example: fill topic
set_run(4, 1, '單元主題：我的主題名稱')
set_run(5, 2, '教學對象：技術型高中二年級')
set_run(6, 5, '物理')           # subject name
set_run(6, 9, '學分數 2')        # credits
set_run(7, 4, '物理')           # version part 1
set_run(7, 6, 'B')              # version part 2
set_run(8, 3, '二年級')
set_run(8, 6, '上學期  5節')
set_run(18, 6, '物理')          # issue-integration version
set_run(18, 8, 'B')
set_run(23, 1, '課堂主題：我的主題')
set_run(24, 2, '5小時')
set_run(25, 1, '對應之 物理 ')
set_run(25, 3, '科綱要學習內容：PKa-V.2-8 ...')
```

## Tables (doc.tables index)

| # | Content | Header rows | Default data rows | Merges? |
|---|---|---|---|---|
| 0 | 總綱素養三面九項對應 (3x3 grid, A1-C3) | 1 (面向 labels) | 3 (fixed, all 9 items pre-printed) | No |
| 1 | 核心素養具體內涵 (面向/項目/項目說明/具體內涵) | 2 | 6 | **Yes** — col0 (面向) vMerge chains |
| 2 | 自然科學領綷學習內容對應 (主題/次主題/學習內容/學習內容說明) | 1 | 9 | No |
| 3 | 議題適切融入 (議題/主題/學習內涵/融入課程綱要學習重點) | 1 | 1 | No |
| 4 | 單元教學架構 (主題/次主題/學習內容/參考節數) | 1 | 3 | No |
| 5 | 課堂教學計畫對應學習內容 (學習內容/學習內容說明/主要概念) | 1 | 9 | No |
| 6 | 學習表現三項目十一子項 (pre-printed reference — shade selected rows, do NOT add/remove/edit text) | 0 | 12 | **Yes** — col0 (項目) vMerge chains across each 項目 group; see Table 6 section below |
| 7 | 學習表現檢核點 (學習內容/項目/子項/學習表現/檢核點) | 1 | 18 | No |
| 8 | 學習目標 + 學習脈絡 | 0 | 5 (row0=學習目標, rows1-4=學習脈絡 1-4) | No (looked merged but isn't — verify with raw tr before assuming) |
| 9 | 學習過程 + 教學歷程 (the big one) | — | 21 total rows | **Yes**, complex — see below |

### Table 0 — 三面九項 selection

All 9 items are pre-printed as static text (e.g. `A2系統思考與解決問題`). To "select" an item: shade its cell (`set_tc_shade` / `w:shd fill="FFE699"` or similar) and bold the run, optionally prefix the text with a marker like `★`. Don't add/remove rows.

**⚠️ CRITICAL — Counterintuitive layout (rows=項目編號, columns=面向):**  
This table is arranged by **項目編號 (1/2/3) as rows** and **面向 (A/B/C) as columns** — the opposite of what most people intuitively expect (面向 as rows, 項目 as columns). Double-check every (row, col) cell against the diagram below before writing selection code.

```
Col:    0             1             2
Row0: [ A自主行動 ] [ B溝通互動 ] [ C社會參與 ]   ← header
Row1: [   A1     ] [   B1     ] [   C1     ]   ← 項目1
Row2: [   A2     ] [   B2     ] [   C2     ]   ← 項目2
Row3: [   A3     ] [   B3     ] [   C3     ]   ← 項目3
```

**Typical mistake:** writing `(1,1)` expecting `A2` but getting `B1` instead. If your lesson selects e.g. `A2, A3, B1, B2, C2`, the correct indices are `{(2,0), (3,0), (1,1), (2,1), (2,2)}`.

```python
# CORRECT example for selecting A2, A3, B1, B2, C2:
for (ri, ci) in [(2,0), (3,0), (1,1), (2,1), (2,2)]:
    set_tc_shade(get_tc(trs[ri], ci), 'FFE699')
    # + bold the text runs
```

### Table 1 — 核心素養具體內涵 (vMerge gotcha lives here)

Header = rows 0-1 (own separate vMerge chain for col0, don't touch). Data rows start at tr index 2. **Default factory state has col0 vMerge grouped as 3+2+1 across the 6 data rows** (a generic placeholder grouping, not tied to any real selection) — this essentially never matches what you need, so always rebuild the col0 vMerge chain explicitly:

```python
from docx_helpers import get_tr_list, get_tc, set_tc_text, remove_tr
from docx.oxml.ns import qn

def set_tc_vmerge(tc, val):  # val: 'restart' | 'continue' | None
    ...  # see scripts/docx_helpers.py
```

Recipe: first `remove_tr`/`duplicate_tr` to get exactly N data rows (N = number of selected core competencies, typically 5). Then, walking raw `tr` list indices 2..2+N-1, set col0 `vMerge='restart'` + text (面向 label, e.g. `A 自主行動`) on the first row of each 面向 group, `vMerge='continue'` + empty text on subsequent rows in that group, and **remove vMerge entirely** (`set_tc_vmerge(tc, None)`) on any group that's a single standalone row. Columns 1-3 (項目/項目說明/具體內涵) are never merged — fill every row directly.

**Do not use `table.cell(r, 0)` here** — for any row inside a vMerge span it resolves to the chain's master cell, so sequential edits across rows 2-6 can all silently land on the same physical XML element (this exact bug was hit and caused the table header to get overwritten with row-6 content during development). Always use `get_tr_list(table)[row_idx]` + `get_tc(tr, col_idx)` instead.


### Table 2 — 學習內容對應 (code+content format requirement)

**⚠️ 學習內容欄位必須填寫「編號+完整名稱」** — 不可只寫代碼（如 `PKc-V.2-11`），必須同時包含完整學習內容名稱（如 `PKc-V.2-11 電流的磁效應`）。使用者在校驗步驟 7（學習過程整合）時，也要求學習重點欄位比照此格式。

**Typical mistake:** 只填 `PKc-V.2-11` 而被要求補上完整名稱，回頭修改費時。應在第一次填入時就使用完整格式。

```python
# WRONG — 只填代碼：
set_tc_text(get_tc(tr, 2), 'PKc-V.2-11', font_size=18)

# CORRECT — 代碼+名稱：
set_tc_text(get_tc(tr, 2), 'PKc-V.2-11 電流的磁效應', font_size=18)
```

此規則同樣適用於 Table 5（col0 學習內容）和 Table 7（col0 學習內容）。Table 9 的學習內容清單欄位也比照辦理，每個代碼後面接完整名稱。

### Table 3 — 議題適切融入 (code+content format requirement)

**⚠️ 融入課程綱要學習重點欄位（col 3）中的學習表現與學習內容代碼必須使用「編號+完整官方描述」**，不可只用簡稱如 `2-V.2-2（計畫與執行）` 或 `PMc-V.2-1（物理在生活中的應用）`。

**Typical mistake:** 填入 `1-V.2-3（批判思辨）` 而被要求補上完整內容。應在第一次填入時就使用完整格式。

```python
# WRONG — 只用簡稱：
set_tc_text(get_tc(tr, 3),
    '2-V.2-2（計畫與執行）：學生規劃3D列印支架的組裝步驟。')

# CORRECT — 代碼+完整官方描述：
set_tc_text(get_tc(tr, 3),
    '2-V.2-2 學生訂定後能進行測試、預測、估算、判斷、計畫、觀察與實驗，以檢核科學論證等能力。\n'
    '學生規劃3D列印支架的組裝步驟，並依計畫逐步執行。\n'
    'PMc-V.2-1 物理在生活中的應用\n'
    '認識3D列印技術於工程製造的實務應用。')
```

每項代碼應獨立一行，承接的應用說明再另起一行，讓欄位內容層次分明。

### Table 6 — 學習表現三項目十一子項 (highlighting selected sub-items)

**DO NOT add, remove, or edit the text** in this table — all 11 sub-items are pre-printed as official curriculum text and must stay intact. The only permitted edit is **shading selected cells yellow** (FFE699) to indicate which 學習表現 items apply to this lesson.

**Row index → 學習表現子項 mapping** (0-indexed raw `<w:tr>` list, using `get_tr_list(table)`):

| Row idx | col0 text (學習表現項目) | col1 text (學習表現子項) | Notes |
|---|---|---|---|
| 0 | 學習表現項目 (header) | 學習表現子項 (header) | do not touch |
| 1 | 探究能力-思考智能 | 想像創造 | col0 vMerge **restart** spanning rows 1-4 |
| 2 | (vMerge continue) | 推理論證 | |
| 3 | (vMerge continue) | 批判思辨 | |
| 4 | (vMerge continue) | 建立模型 | |
| 5 | 探究能力-問題解決 | 觀察與定題 | col0 vMerge **restart** spanning rows 5-8 |
| 6 | (vMerge continue) | 計畫與執行 | |
| 7 | (vMerge continue) | 分析與發現 | |
| 8 | (vMerge continue) | 討論與傳達 | |
| 9 | 科學的態度與本質 | 培養科學探究的興趣 | col0 vMerge **restart** spanning rows 9-11 |
| 10 | (vMerge continue) | 養成應用科學思考與探究的習慣 | |
| 11 | (vMerge continue) | 認識科學的本質 | |

**Highlighting recipe** — must use raw `get_tr_list` / `get_tc` (NOT `table.cell()`) because col0 has vMerge chains:

```python
from docx_helpers import get_tr_list, get_tc, set_tc_shade

def highlight_table6(table, selected_subitems: set):
    """
    selected_subitems: set of 學習表現子項 strings matching the col1 text above
    (e.g. {"推理論證", "建立模型", "計畫與執行", "培養科學探究的興趣"}).
    Shades col1 for every selected subitem, and also shades the col0 項目 label
    cell of any group that contains at least one selected row.
    """
    HIGHLIGHT = 'FFE699'
    trs = get_tr_list(table)

    # col1 subitem text → row index
    subitem_rows = {
        '想像創造': 1, '推理論證': 2, '批判思辨': 3, '建立模型': 4,
        '觀察與定題': 5, '計畫與執行': 6, '分析與發現': 7, '討論與傳達': 8,
        '培養科學探究的興趣': 9, '養成應用科學思考與探究的習慣': 10, '認識科學的本質': 11,
    }
    # col0 label appears at these row indices (vMerge restart rows)
    label_row_for_group = {
        frozenset([1,2,3,4]): 1,   # 探究能力-思考智能
        frozenset([5,6,7,8]): 5,   # 探究能力-問題解決
        frozenset([9,10,11]): 9,   # 科學的態度與本質
    }

    selected_row_indices = {subitem_rows[s] for s in selected_subitems if s in subitem_rows}

    # shade col1 for each selected row
    for ridx in selected_row_indices:
        set_tc_shade(get_tc(trs[ridx], 1), HIGHLIGHT)

    # shade col0 label if ANY row in its group is selected
    for group, label_ridx in label_row_for_group.items():
        if group & selected_row_indices:
            set_tc_shade(get_tc(trs[label_ridx], 0), HIGHLIGHT)
```

**Important**: call this function AFTER filling all other tables (especially table7 whose content drives the selection), and BEFORE saving. This was discovered as a missing step during development — the 三面九項 table (table0) was highlighted correctly from the start, but table6 was initially documented as "STATIC, do not edit" and left unshaded, causing all sub-items to appear unselected in both lesson plans produced. The fix is this `highlight_table6` call, which must be added to every lesson plan generation script.

**Common selections by lesson type** (for reference, not as defaults — always derive from the actual lesson's Step-5 analysis):
- Electrolysis lesson: 推理論證, 建立模型, 觀察與定題, 計畫與執行, 分析與發現, 討論與傳達, 培養科學探究的興趣
- Chemistry/3D-printing lesson: 推理論證, 建立模型, 觀察與定題, 計畫與執行, 培養科學探究的興趣

### Table 7 — 學習表現檢核點 (filling the 學習表現 column)

**Important: the 學習表現 column (col 3) must contain BOTH the sub-item code AND its full description from the curriculum**, not just the code number alone. For example, write `2-V.2-1 學生常基於好奇、求知需求，對現象進行觀察且發掘與問題相關的訊息，並確定待解決或待探究的問題。` rather than just `2-V.2-1`. This ensures the teacher can read the assessment criteria directly from the lesson plan without cross-referencing back to the curriculum PDF. The code+description pairs for V.2 are listed in the curriculum PDF's shared 學習表現 table (see workflow.md Step 5 for grep patterns). Data rows start at index 1 (row 0 = header), columns: 0=學習內容, 1=項目, 2=子項, 3=學習表現, 4=檢核點. Adjust row count with `duplicate_tr`/`remove_tr` to match your number of 檢核點 items (typically 5-8).

### Table 8 — 學習目標 + 學習脈絡 (column placement gotcha)

**⚠️ Row 0 = 學習目標（col0 標籤 + col1 填入目標內容）；Rows 1-4 = 學習脈絡（col0 標籤 + col1 填入步驟說明）。**

Blank template layout (verified from raw XML):

```
Row 0: col0="學習目標" (pre-printed label, keep as-is)  |  col1=empty (FILL 5 learning goals here)
Row 1: col0="學習脈絡" (pre-printed label, keep as-is)  |  col1="1.發現問題：" (pre-printed label — FILL full context after it, e.g. "1.發現問題：觀察到什麼現象？")
Row 2: col0=empty (leave empty)                         |  col1="2.規劃研究： " (pre-printed — FILL full context after it)
Row 3: col0=empty (leave empty)                         |  col1="3.論證建模： " (pre-printed — FILL full context after it)
Row 4: col0=empty (leave empty)                         |  col1="4.表達分享： " (pre-printed — FILL full context after it)
```

**Key rules:**
1. **Row 0 col1** = write the 5 learning goals (numbered 1.～5.), NOT empty.
2. **Rows 1-4 col0** = keep pre-printed labels (`學習脈絡` in row 1, empty in rows 2-4), do NOT write anything extra.
3. **Rows 1-4 col1** = write the full context description **including the pre-printed label prefix** (e.g. `1.發現問題：馬達為什麼會轉？`). Since `set_tc_text` overwrites the entire cell, you must include the label text as part of what you write — it will NOT be preserved from the blank template otherwise.

```python
# CORRECT:
# Row 0 — learning goals into col1:
set_tc_text(get_tc(trs[0], 1),
    '1. 能安全使用工具拆解馬達，辨識關鍵構造。\n'
    '2. 能運用右手開掌定則解釋直流電動機的旋轉原理。\n'
    '3. ...\n4. ...\n5. ...',
    font_size=18)

# Rows 1-4 — full context (label prefix + description) into col1:
set_tc_text(get_tc(trs[1], 1),
    '1.發現問題：馬達為什麼會轉？內部有哪些零件？', font_size=18)
set_tc_text(get_tc(trs[2], 1),
    '2.規劃研究：規劃安全拆解步驟與AI提問策略。', font_size=18)
set_tc_text(get_tc(trs[3], 1),
    '3.論證建模：實體零件比對驗證AI資訊；建立因果模型。', font_size=18)
set_tc_text(get_tc(trs[4], 1),
    '4.表達分享：各組展示成果，分享除錯經驗。', font_size=18)

# Rows 1-4 col0 — keep pre-printed labels as-is (row1) or leave empty (rows 2-4):
# (do NOT write anything to col0 of rows 1-4)
```

**Common mistakes:**
1. Leaving **row 0 col1 empty** — goals must be filled here, they do not go elsewhere.
2. Writing goals into **row 0 col0** — this overwrites the pre-printed "學習目標" label.
3. Writing context descriptions into **rows 1-4 col0** — col0 is for the "學習脈絡" label only; context goes into col1.
4. **Omitting the label prefix** (e.g. writing only `130馬達為什麼會轉？` instead of `1.發現問題：130馬達為什麼會轉？`) — because `set_tc_text` clears the cell, the pre-printed `1.發現問題：` is lost unless you include it in the text you write.

If the pre-printed label in col0 was accidentally overwritten, restore it: `set_tc_text(get_tc(trs[0], 0), '學習目標', bold=True)`.

### Table 9 — 學習過程 + 教學歷程

Raw tr layout (0-indexed):
- rows 0-5: simple `[label(gs1), content(gs7)]` — 主題名稱/教學時間/教材來源/教學資源/教學準備/教學說明. Fill `get_tc(tr, 1)`.
- row 6: 5 `<w:tc>` cells = `[學習重點(gs1,vMerge restart), 學習內容(gs1 label), content(gs1, FILL), 核心素養(gs2 label,vMerge restart), content(gs3,vMerge restart, FILL)]`. Fill index 2 (學習內容 list) and index 4 (核心素養 list — this cell vertically spans rows 6-7, set it only once here).
- row 7: 5 cells = `[continue(skip), 學習表現(gs1 label), content(gs1, FILL), continue(skip), continue(skip)]`. Fill index 2 (學習表現 list) only.
- row 8: `[情境規劃(gs1 label), content(gs7, FILL)]`.
- row 9: 教學歷程 sub-table header, 4 cells: `教學歷程(gs4) / 教學時間(gs2) / 教學資源(gs1) / 教學評量(gs1)` — labels, don't touch.
- rows 10-20 (11 rows by default): 教學歷程 data rows, each `[歷程內容(gs4), 教學時間(gs2), 教學資源(gs1), 教學評量(gs1)]`, no vMerge. Adjust row count with `duplicate_tr`/`remove_tr` to match your number of teaching stages (typically 4-6), then fill all 4 cells per row directly — these are NOT merged, `get_tc(tr, col)` on a plain row index is safe (the gridSpan-only merges here don't trigger the vMerge resolution bug, but using the raw helpers throughout is still recommended for consistency).

## Validation snippet (re-run if template changes)

```python
from docx import Document
from docx.oxml.ns import qn
doc = Document('blank.docx')
for i, t in enumerate(doc.tables):
    trs = t._tbl.findall(qn('w:tr'))
    print(i, len(trs), 'rows x', len(trs[0].findall(qn('w:tc'))), 'cols (row0)')
```
