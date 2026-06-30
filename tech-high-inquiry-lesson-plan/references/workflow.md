# 九步驟分析方法論（依據課程設計者原始 GEM.txt 流程整理）

This is the analytical methodology behind each of the 9 steps referenced in SKILL.md. Apply it to whatever subject (物理/化學/生物/地球科學) and topic the user gives you — the chemistry/3D-printing example below is a calibration reference for depth and tone, not a template to copy verbatim.

## Searching the bundled curriculum PDF

The PDF is ~140 pages covering all four subjects, A版 and B版. Don't read it whole. Proven approach:

```bash
pdftotext -layout assets/All-技術型高中自然科學領域課程綱要.pdf curriculum.txt
grep -n "<search term>" curriculum.txt
sed -n '<start>,<end>p' curriculum.txt   # view the matched region with context
```

Useful anchor searches by step:
- **Step 1/2** (總綱核心素養): grep `^U-A\|^U-B\|^U-C` and `自V.2-U-A\|自V.2-U-B\|自V.2-U-C` — these full descriptions appear once, near the front of the relevant subject section. The exact phrasing differs slightly by subject (物理/化學/生物/地科 each have their own 自V.2-U-Ax wording) — always pull the wording for the SPECIFIC subject of the lesson, don't reuse another subject's wording.
- **Step 3** (學習內容): grep the subject's code prefix pattern, e.g. `CAa\|CCb\|CMa-\|CMc-` for chemistry, `PKc-\|PMc-\|PNa-` etc for physics (single-letter prefix = subject: P=Physics, C=Chemistry, B=Biology, E=Earth science; first capital letter after that = 主題 K/N/M/etc; lowercase = 次主題). View the surrounding table for 主題/次主題/學習內容/節數 hints.
- **Step 4** (議題融入): **Primary source: `assets/台灣_108_課綱19項議題融入.pdf`** (bundled). Use `pdftotext -layout` to extract and read this file; its 內涵 and 學習重點方向 text may be quoted verbatim for the 學習內涵 column of table 3. The curriculum PDF's appendix (grep `議題.*主題.*學習\|議題適切融入`) remains a secondary source for subject-specific 實質內涵 codes (e.g. 性U8, 人E6) where available.

  **All 19 issues — extracted verbatim from the bundled PDF** (use this table to select the best fit and draft the 學習內涵 column without re-reading the PDF every time):

  | # | 議題名稱 | 內涵摘要 | 學習重點方向摘要 | 自然科學課程常見連結情境 |
  |---|---|---|---|---|
  | 1 | **性別平等教育** | 消除性別歧視與刻板印象，促進性別地位實質平等，尊重多元性別認同 | 認識性別特質的多元性、破除性別刻板印象、培養性別平等意識、尊重與包容不同性別者、了解性別暴力防治 | STEM 參與機會平等；科技/實作課程全員動手操作不因性別設限 |
  | 2 | **人權教育** | 認識、尊重與保障個人及群體的基本權利，理解人權的普世性、不可分割性、相互依存性 | 認識基本人權、了解人權的歷史發展、關注弱勢群體的人權、培養尊重他人權利的態度、以和平方式解決人權爭議 | 科學倫理、研究對象的隱私與知情同意、實驗動物福利 |
  | 3 | **環境教育** | 培養愛護環境、珍惜資源的態度，理解人與自然的關係，具備環境保護的知識與行動能力 | 認識自然環境、了解環境問題及其成因、學習永續發展的觀念、培養環境倫理與責任、參與環境保護行動 | 生態系、物質循環、能源議題、垃圾減量、化學廢液處理 |
  | 4 | **海洋教育** | 認識海洋的重要性與豐富性，培養親海、愛海、知海的素養，具備海洋保育的知識與行動能力 | 認識海洋生態、了解海洋資源與產業、關注海洋環境問題、培養海洋文化意識、參與海洋保育行動 | 海洋生物、洋流、海洋汙染、微塑膠、地球科學海洋單元 |
  | 5 | **品德教育** | 培養學生良好品格與道德價值觀，涵養誠實、守信、負責、尊重、關懷等重要德行 | 認識重要品德價值、反思個人行為與道德選擇、培養道德判斷能力、實踐良好品德於日常生活中 | 實驗誠信、資料不造假、團隊合作中的負責任態度 |
  | 6 | **生命教育** | 探索生命的意義與價值，尊重生命的多樣性，培養積極樂觀的人生態度，學習面對生命中的各種挑戰 | 認識生命的奧秘與可貴、了解個體生命的獨特性、欣賞與尊重不同生命形式、探討生死議題、培養心理健康與調適能力 | 生物課（生命週期、演化）、醫學倫理 |
  | 7 | **家庭教育** | 認識家庭結構與功能，培養家庭成員間的良好互動與情感連結，學習經營幸福美滿的家庭生活 | 認識不同家庭型態、了解家庭成員的角色與責任、學習溝通與協調技巧、培養家庭關懷與支持能力 | 自然科學連結較弱；通常不用於理科教案 |
  | 8 | **原住民族教育** | 認識台灣原住民族的歷史、文化、語言與傳統，培養尊重多元文化、族群平等的意識 | 認識台灣各原住民族的特色、了解原住民族的權益與處境、培養多元文化視野、促進族群間的理解與尊重 | 傳統生態知識（TEK）、台灣地質與地理情境連結 |
  | 9 | **資訊教育** | 培養學生具備資訊素養，包括資訊倫理、資訊安全、資訊應用與問題解決能力 | 認識資訊科技的基本概念、培養安全合法使用資訊的習慣、學習有效搜尋、評估與應用資訊、發展程式設計與運算思維能力 | AI/大數據在科學探究的應用、資料分析、批判性評估數位資訊來源（**特別適合含 AI 工具的探究課程**） |
  | 10 | **科技教育** | 培養學生具備科技素養，包括科技認知、科技應用、科技創造與科技倫理 | 認識科技發展與影響、學習操作與應用科技工具、培養設計與製作能力、了解科技發展的倫理議題 | 3D列印、電子電路、機器人、感測器、創客實作（**適合工程設計/Maker 類探究課程**） |
  | 11 | **能源教育** | 認識能源的重要性與種類，了解能源使用對環境的影響，培養節約能源與發展綠色能源的意識與行動 | 認識不同能源的特性與應用、了解能源消耗造成的環境問題、學習節約能源的方法、認識再生能源的發展與利用 | 電磁感應、能量轉換效率、太陽能、核能、熱力學 |
  | 12 | **安全教育** | 培養學生具備保護自身與他人安全的能力，包括防災、交通安全、網路安全、性侵害防治等 | 認識各種潛在的危險情境、學習應對災害與緊急狀況的方法、培養安全防護意識與技能 | 化學實驗室安全、電器安全、輻射防護 |
  | 13 | **防災教育** | 認識各種災害的成因與影響，學習防災知識與技能，培養防災意識與應變能力 | 認識台灣常見的自然災害、了解災害預警與避難方式、學習急救與互助技能、培養防災準備的習慣 | 地震、颱風、土石流（地球科學）；核安、化學危害（化學） |
  | 14 | **閱讀素養教育** | 培養學生廣泛閱讀的興趣與習慣，提升理解、分析、評估與應用文本的能力，發展獨立思考與終身學習的基礎 | 培養多元閱讀的興趣、提升閱讀理解的策略、學習批判性閱讀與評估資訊、發展資訊素養與媒體識讀能力 | 科學新聞解讀、期刊摘要閱讀、AI 生成文本的批判性閱讀 |
  | 15 | **戶外教育** | 透過親身體驗自然環境與人文場域，培養學生探索、體驗、合作與解決問題的能力，增進對環境的關懷與尊重 | 參與戶外探索與體驗活動、學習團隊合作與溝通、培養環境觀察與感知能力、發展解決問題與應變能力 | 野外採集、地質踏查、生態調查、天文觀測 |
  | 16 | **國際教育** | 培養學生具備國際視野與跨文化理解能力，尊重多元文化，促進全球公民意識 | 認識不同國家與文化的特色、了解全球議題與趨勢、培養跨文化溝通與合作能力、發展全球責任感 | 全球氣候變遷、國際科學合作、跨國環境議題 |
  | 17 | **媒體素養教育** | 培養學生具備批判性思考媒體內容、辨識媒體訊息、安全合理使用媒體的能力 | 認識不同媒體的特性與影響、學習分析與評估媒體內容、培養媒體識讀與批判思考能力、了解網路倫理與安全 | 辨別科學謠言/偽科學、評估 AI 生成資訊的可靠性（**適合含 AI/網路資訊的探究課程**） |
  | 18 | **藝術與人文素養** | 培養學生審美感知、文化理解、藝術表達與創造力，涵養人文關懷與藝術欣賞能力 | 培養對藝術與人文的興趣、提升藝術欣賞與創作能力、了解不同文化脈絡下的藝術表現、發展人文關懷與反思能力 | 科學美學（結晶、星雲攝影）；自然科學連結較弱，通常不列為首選 |
  | 19 | **SDGs 永續發展目標** | 認識聯合國永續發展目標（SDGs），了解全球共同面臨的挑戰與機會，培養為永續發展貢獻的能力與意願 | 認識 SDGs 的各項目標、了解永續發展的重要性與挑戰、培養永續發展的意識與行動力、學習跨領域合作解決永續發展問題 | 任何涉及能源、環境、健康、永續材料、糧食的理科課程均可連結 |

  **Selection heuristics** (pick 1–2 per lesson; avoid over-selecting):
  - **Maker/engineering/3D-printing/robot topics** → 科技教育（#10）＋ 資訊教育（#9，if AI tools are used）
  - **AI tool–heavy inquiry** → 資訊教育（#9）＋ 媒體素養教育（#17，if critical evaluation of AI output is central）
  - **Energy conversion / electric circuits / electromagnetism** → 能源教育（#11）
  - **Ecology / field biology / environmental chemistry** → 環境教育（#3）, SDGs（#19）
  - **Lab safety / chemical handling** → 安全教育（#12）
  - **Geology / natural disaster topics** → 防災教育（#13）
  - **Cross-gender STEM participation** → 性別平等教育（#1）
  - **Data analysis / AI-assisted research** → 資訊教育（#9）

  **How to fill table 3 columns**:
  - **議題**: issue name (e.g. 科技教育)
  - **主題**: a 4–8 character label matching the lesson's specific connection to the issue (e.g. 科技的應用與創造、資訊的評估與應用) — derive from the 學習重點方向 bullet most relevant to the lesson
  - **學習內涵**: quote or closely paraphrase the issue's 內涵 + the 1–2 most relevant 學習重點方向 bullets from the PDF, adapted to the lesson context. This is the column where quoting the bundled PDF directly is appropriate.
  - **融入課程綱要學習重點**: cite the specific 學習表現 sub-item codes (e.g. 2-V.2-2 計畫與執行) and 學習內容 codes (e.g. PMc-V.2-1) that most naturally connect to this issue, with a 1–2 sentence explanation of how the issue is realized in the lesson's activities. If the curriculum PDF appendix has subject-specific 實質內涵 codes (e.g. 性U8 for 性別平等教育), cite those too.
- **Step 5** (學習表現): grep `想像創造\|推理論證\|批判思辨\|建立模型\|觀察與定題\|計畫與執行\|分析與發現\|討論與傳達` to find the V.2 (technical high school) 三項目十一子項 table with full descriptions — this table is SHARED across all subjects (not subject-specific), located once near the front of the natural-science general section.

## Step-by-step output shape

1. **三面九項**: a short table (自主行動/溝通互動/社會參與 columns) bolding the selected items, with 1-2 sentence rationale per item tying it to concrete elements of the topic. Don't over-select — 4-6 items is typical, matching the granularity of official exemplar lesson plans.
2. **核心素養具體內涵**: full official text (U-Ax + 自V.2-U-Ax) for every item selected in step 1, grouped by 面向 (A自主行動/B溝通互動/C社會參與) — preserve every code prefix exactly as it appears in the curriculum.
3. **學習內容**: a table with 主題/次主題/學習內容/學習內容說明/主要概念/參考節數. 學習內容說明 must be specific to what's actually taught in this lesson (don't just copy the bare official one-line content code description — explain how this lesson's actual activities realize it, the way a real exemplar lesson plan does). 主要概念 is a short title + one-sentence gloss. 節數 must be a clean sum that matches the lesson's total class periods — double check the arithmetic before moving on (a mid-development bug here, caught and fixed at the final docx stage, shows this is an easy place to slip up when content spans many sessions).
4. **議題融入**: 議題/學習主題/實質內涵/融入課程綱要學習重點 with rationale. See the curriculum-search note above for handling issues not detailed in the bundled PDF.
5. **學習表現**: a table mapping each step-3 學習內容 to one or more 學習表現 (項目/子項/學習表現全文/自訂檢核點). For hands-on/operational content pair an operational performance code (often 2-V.2-2 計畫與執行) with an attitude/interest code (often 3-V.2-1 培養科學探究的興趣), mirroring how official exemplar plans give roughly 1.5-2 rows per 學習內容 item.
6. **學習目標+學習脈絡**: 4-5 學習目標 (concrete, each tied to a specific skill/content from steps 1-5), 4+ 學習脈絡 stages — the canonical 4-stage shape is 發現問題/規劃研究/論證建模/表達分享 (探究與實作 spirit: problem-finding → planning → modeling/argumentation → communicating), but adapt the labels if the topic doesn't fit that shape naturally.
7. **學習過程**: synthesize 主題名稱/教學時間/教材來源/教學資源/教學準備/教學說明/學習重點(學習內容+核心素養+學習表現, as short code lists)/情境規劃 (a short narrative arc of the whole lesson).
8. **教學歷程**: break the total class periods into 4-8 stages (don't force every individual content-session from a messy source plan into its own stage — group logically), each with full numbered sub-steps (what the teacher/students actually do), 教學時間 (sum must equal total), 教學資源, and 教學評量 that explicitly cites the step-5 檢核點/code where applicable.
9. **Generate the docx** — see SKILL.md's "Generating the docx" section and references/template_structure.md.

## Worked example (chemistry, for calibration — do not copy verbatim into a different topic)

Topic: 化學分子與3D列印的邂逅 (8 class periods: 基礎化學2節+彈性學習時間3D列印入門6節)

- Step 1 selected: A2系統思考與解決問題, A3規劃執行與創新應變, B1符號運用與溝通表達, B2科技資訊與媒體素養, C2人際關係與團隊合作.
- Step 3 produced 7 學習內容 items spanning CAa-V.2-3/4/6 (原子結構/電子排列/元素週期表), CCb-V.2-3/4 (物質的結構/分子模型實驗), CMa-V.2-1 (科學技術社會互動), CMc-V.2-4 (材料與化學：塑膠) — summing to 8 節 across 5 grouped 單元教學架構 rows (1 + 0.5 + 1 + 3.5 + 2 = 8).
- Step 4: the topic's natural fit is 科技教育（#10, 「設計與製作」）. Source from `assets/台灣_108_課綱19項議題融入.pdf`: 內涵「培養學生具備科技素養，包括科技認知、科技應用、科技創造與科技倫理」; 融入課程綱要學習重點 cites 2-V.2-2（計畫與執行）and PMc-V.2-1（物理在生活中的應用）.
- Step 8 produced 6 teaching stages (3節+1節+1節+1節+1節+1節 = 8節) each with 4-6 numbered sub-actions.

This shape — roughly 5-7 學習內容 items, 5 core competencies, 4-8 teaching stages for an 4-8 period lesson — is a reasonable density target, but always let the actual source topic/content drive the real numbers rather than forcing this exact count.
