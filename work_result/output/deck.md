# Harness² 逐頁大綱

13頁，約10分鐘；16:9、白底、深藍；繁體中文。頁碼皆為PDF實體頁碼（與頁腳一致）。下列其餘裁切座標以1.6x渲染尺寸表示；成品使用4x渲染，裁切座標乘2.5。

## 1. Harness²：讓 Agent 改善自己的執行方式

- 畫面文字：模型權重不變，透過執行對照修改 harness；研究問題、核心機制、三組證據與工程適用邊界
- 原文證據：Xu et al. · arXiv:2610.14224v1 · 2026-10 · Northwestern / Google Cloud AI Research
- 圖表裁切：無；以可編輯文字或原生表格呈現
- 講稿：25 秒。研究的對象是 frozen model 外圍的 harness。任務內比較候選修改，任務後保留可重用元件與經驗。不是微調權重，也不是沒有邊界的自主自我改寫。

## 2. 沒有正確答案時，仍需要判斷哪個修改值得保留

- 畫面文字：當前任務缺少可信 verifier；下題直接沿用客製規則可能帶入錯誤前提；比較行為後保留經驗
- 原文證據：PDF pp.2–4 · §1、§2.1
- 圖表裁切：無；以可編輯文字或原生表格呈現
- 講稿：45 秒。開放世界表示任務在設計 harness 時未見；線上改進拿不到答案、隱藏測試或 rubric。難點有二：如何判斷當前修改，以及如何讓修改經驗幫助下一題。本文允許同一題在預算內重跑，不能把它理解成單次執行就有改善。

## 3. Harness 的八種元件涵蓋提示、工具與執行控制

- 畫面文字：原圖八元件；可掛載檔案；模型參數凍結
- 原文證據：PDF p.3 · Figure 2；Appendix D.3 / Table 5
- 圖表裁切：p.3 Figure 2: rendered 953×1348, pixel (100,135,854,405)
- 講稿：45 秒。圖按運作角色區分 context construction、middleware、tool interface、orchestration。八元件為 system prompt、guardrails、memory、tool guidance、subagents、skills、scripts、plugin。並非每題都要修改所有元件。guardrails.md 是文字規則，不能直接等同 OS sandbox。

## 4. 先提出並實測修改，再把經驗帶到下一個任務

- 畫面文字：Propose → Probe → Compose；任務後保留 harness 與經驗；下題從 H0 開始
- 原文證據：PDF pp.4–7 · Figure 3、§2.2–2.3
- 圖表裁切：p.5 Figure 3: pixel (98,130,855,380)
- 講稿：70 秒。先用基礎 harness 得到 reference trajectory。提出 INV（通用流程缺陷）與 DOM（領域程序）兩種候選，各自執行，再由 Composer 依行為差異合成或保留 base，並執行合成結果。任務後保存採用的 harness 與 EXPERIENCE.md。下一題從共同 H0 開始，透過改善庫輔助新提案；不自動繼承上一題所有修改。任務內 improvement harness 固定；跨任務改的是 library，stage instructions 與模型權重固定。

## 5. 三個專業基準以隱藏評分衡量結果，評分不回饋給編輯器

- 畫面文字：WB 40/35/25；JB 34；LAB100；各指標與主配置
- 原文證據：PDF p.7、pp.18–20 · §3.1、Appendix B
- 圖表裁切：無；以可編輯文字或原生表格呈現
- 講稿：45 秒。主實驗兩個 executors（Gemini 3.5 Flash、GLM-5）乘上 OpenCode、Codex 共四配置，預設 editor 為 OpenCode 上的 Claude Opus 4.8。平均與標準差來自三次任務順序洗牌的 runs；不是三個樣本的顯著性證明。WB 使用各領域分數宏平均；不同 benchmark 指標不可直接平均。

## 6. WorkBuddy 四種配置皆改善，GLM-5 最大增加 11.5 分

- 畫面文字：Gemini/OpenCode72.4(0.9)→77.5(2.5)→80.8；Gemini/Codex74.5(1.6)→77.5(2.0)→80.4；GLM/OpenCode64.2(1.4)→75.7(2.2)→79.6；GLM/Codex64.4(1.2)→75.3(1.3)→78.3；k5依增量計算
- 原文證據：PDF p.8 · Table 1 · WorkBuddy 領域宏平均
- 圖表裁切：無；以可編輯文字或原生表格呈現
- 講稿：60 秒。表中 base 與 k1 為 mean（SD）；k5 為依原表增量相加的衍生平均，未重建 SD。GLM-5/OpenCode：64.2→75.7，增加11.5分；同配置最強比較方法 Meta-Harness69.7，Harness2 高6.0。Gemini/OpenCode 最強 baseline Meta-Harness76.4，Harness2 77.5，差1.1分。主張為四配置平均改善，不是每一領域單格都最佳。GLM-5/Codex Code 的 Meta-Harness76.8 高於 Harness2 k1 74.9。

## 7. 跨基準仍有增益，但不是所有對照都勝出

- 畫面文字：JB Gemini/OpenCode31.5→36.0，+4.5分；LAB GLM/OpenCode59.6→71.4，+11.8百分點；LAB Gemini/Codex75.6落後Meta76.5
- 原文證據：PDF p.9 · Table 2 · OpenCode；例外為 Gemini / Codex
- 圖表裁切：無；以可編輯文字或原生表格呈現
- 講稿：45 秒。挑兩個不同設定呈現摘要所報告最大單步值：JobBench +4.5屬於Gemini/OpenCode，LAB +11.8 percentage points屬於GLM-5/OpenCode。兩者不同量尺，不作平均。LAB Gemini/Codex：Harness2 75.6 (4.5)低於 Meta-Harness76.5 (2.1)。這是評分指標的改善，不能換算成整題全對率。完整主要數據與SD見書面報告。

## 8. 增加探索預算能提高分數，也會增加總推論量

- 畫面文字：原圖總tokens比較；WB增益5.1→8.4；JB4.5→7.5；4k rollouts，另計編輯選擇反思
- 原文證據：PDF pp.9–10 · Figures 5–6 · Gemini 3.5 Flash / OpenCode
- 圖表裁切：p.10 Figure 6: pixel (365,328,852,520)
- 講稿：60 秒。k 是探索預算而非 token 數。平行是每個scope各k個候選、k個composition；序列是k個串接改善步。從流程推導 executor rollouts共4k（k reference +2k probes +k successors），另有3k編輯/合成 sessions；selector在k>1啟用，reflection另計。k1不是只執行一次。Fig6是所有交付前tokens的empirical Pareto frontier，不代表美元成本或端到端延遲。跨任務reflection的後續維護成本還須另計。#Steps是執行步數，不能拿來代表整個搜尋流程總成本。

## 9. 經驗庫、角色分工與合成都有貢獻

- 畫面文字：完整77.5/36.0；w/o library74.4/34.2；w/o scoping74.6/32.3；w/o composition75.9/34.1
- 原文證據：PDF p.10 · Table 3 · Gemini 3.5 Flash / OpenCode、k=1
- 圖表裁切：無；以可編輯文字或原生表格呈現
- 講稿：45 秒。w/o scoping仍保留兩候選與相同probe budget，用generic editors替換INV/DOM；因此不能僅用多問一次解釋完整改善。去composition改成候選擇一，仍不代表完全等token控制。每列是各自消融，不可把損失相加；消融結果未報SD，不能自行主張顯著性。

## 10. 經驗可跨任務重用，但缺乏證據的直接修改可能倒退

- 畫面文字：原圖直接修改與JB→WB移轉；跨基準+0.6/+0.5/+2.9
- 原文證據：PDF pp.10–11 · Figure 7 · Gemini 3.5 Flash / OpenCode
- 圖表裁切：p.11 Figure 7: pixel (365,192,851,352)
- 講稿：50 秒。左圖直接編輯沒有新的reference/probe；無library時Code73.4→63.2，較豐富k5 library可到74.3，但不等於完整探測系統76.5或80.8。右圖JobBench經驗移轉到WB一階改善，Code76.5→77.1、Web76.4→76.9、Office79.5→82.4。這是特定任務族的轉移。Figure11排除自身entry再重評使用較晚任務經驗，是回溯測試，不是當時在線收益。

## 11. 好候選已經存在，選擇器仍可能交出較差結果

- 畫面文字：原圖選擇失敗；最高95卻選55，base96；共同偏誤
- 原文證據：PDF pp.25–26 · Figure 14；p.29 · Appendix F–G
- 圖表裁切：p.26 Figure 14 WB-Code 案例: pixel (100,200,370,425)；原圖以4x重新渲染，裁切座標乘2.5
- 講稿：45 秒。WB-Code失敗案例：候選有95、85、85、80、55，選擇器選55，reference96。數字是事後評分，線上selector看不到。另一類是錯誤假設傳到所有候選，或judge本身失敗。作者限制：額外推論成本、執行變異與共同偏誤，無正確性或轉移保證。案例不是失敗率估計。

## 12. 工程試驗應先驗證收益、成本與回歸是否可接受

- 畫面文字：以內部可重放任務比較Base/Best-of-k/Harness2；品質/token/延遲/回歸/選擇錯誤；保留版本與適用條件；副作用隔離
- 原文證據：工程應用推論；依據 PDF pp.5–7、10、25–29
- 圖表裁切：無；以可編輯文字或原生表格呈現
- 講稿：40 秒。以下是編輯者建議而非作者已完成的企業驗證。適合可重放、可隔離副作用、有可比較交付物的離線或非即時任務。對外寫入不可直接在每個probe重複發生，應使用測試副本或dry run。採共同H0避免客製harness累積漂移，library保留來源與條件。企業權限與sandbox不能由可自我修改的guardrails.md取代。

## 13. 外部評論看好可執行經驗，也提醒別把自評當成真值

- 畫面文字：Zhou：看好從差異累積經驗，提醒非正確性保證；Torres：可執行harness與理由，demo不等於自主改進；原論文圖5；非獨立復現或共識
- 原文證據：外部個人技術評論 · 檢索 2026-11-19；完整連結與限定見講稿
- 圖表裁切：p.9 Figure 5: pixel (360,852,852,1043)
- 講稿：40 秒。Zhongzhu Zhou（2026-10-24）認為這是execution-informed workflow search with accumulated experience，不是correctness oracle或無限自我改進。Nicolás Torres（2026-10-19）推薦保留完整harness與EXPERIENCE.md，並強調credential-free demo的proposer是scripted，僅證明流程可運作。這兩篇是個人解讀，不是第三方基準復現或社群共識。Torres原文把JobBench+4.5歸到GLM-5，與Table2不符，本報告依論文更正為Gemini/OpenCode。附圖是論文Figure5，供討論預算與收益；圖中座標及比較線保留。
https://www.zhongzhuzhou.org/blog/2026-10-24-harness2-technical-review-en/
https://ntorres.dev/blog/harness2-google-recursive-agent-harness
https://github.com/google-research/harness2
