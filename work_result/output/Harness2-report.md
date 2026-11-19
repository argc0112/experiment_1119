# Harness² 論文研究報告

論文：*Harness²: Recursive Agent Harnessing for an Open World*  
作者：Ruiyao Xu 等；Northwestern University / Google Cloud AI Research  
版本：2610.14224v1；2026-10-09 提交，PDF 首頁日期 2026-10-12  
閱讀與外部資料檢索：2026-11-19  
原文：https://arxiv.org/abs/2610.14224  
程式碼：https://github.com/google-research/harness2

## 一、研究要解決什麼

Agent 的表現不只取決於模型，也受 system prompt、skills、scripts、memory、工具介面與執行控制影響。本文把這些可修改的外圍元件稱為 harness。研究問題是：面對設計時未見的新任務，而且線上沒有正確答案或可靠 verifier 時，能否改善當前 harness，同時累積有助於未來改善的經驗？

作者的兩項主張須分開看：第一是當前任務透過執行對照改善；第二是將修改過程保存為跨任務改善資源。兩者都不更新模型權重。改善模型的階段提示與演算法也保持固定；跨任務持續更新的是 library，而非整套 optimizer 自行重寫（PDF pp.4–7）。

## 二、核心做法

### 2.1 可修改的八個元件

| 運作位置 | 元件 | 功能 |
| --- | --- | --- |
| Context construction | System prompt、guardrails、memory、skills | 決定哪些指示或程序進入 context，以及何時載入 |
| Tool interface | Tool guidance、scripts | 指導工具使用，或提供可執行程序 |
| Orchestration | Subagents | 決定委派方式與子代理角色 |
| Middleware | Plugin | 在模型自主決策之外介入工具呼叫 |

證據：PDF p.3 Figure 2；pp.21–22 Appendix D.3 / Table 5。元件以檔案掛載；不是每個任務都修改全部。文字 guardrails 不能直接視為 OS 層 sandbox。

### 2.2 任務內：Propose–Probe–Compose

1. 用當前 harness 執行任務，取得 reference trajectory 與 deliverable。
2. Propose：提出兩類可執行修改。INV 處理規劃、工具使用、context 或委派等跨領域流程問題；DOM 處理領域知識、作業程序與交付限制。
3. Probe：在相同模型與環境配置下執行候選，比較修改前後的行為差異。這能提供修改效果的線索，不能證明答案正確。
4. Compose：檢視候選、diff 與執行紀錄，合併有支持的修改、修正衝突、選擇一個候選，或保留原始 harness。合成後仍需執行。

證據：PDF p.5 Figure 3、§2.2.1。作者要求具體執行證據，並允許 KEEP_BASE。

### 2.3 探索預算 k 與兩種遞迴

平行遞迴：蒐集 k 個 reference、每種 scope 各 k 個候選，共 2k 個 probes，再由 k 個獨立 composition sessions 產生並執行 successor。實作為一次 population update；k 控制寬度，不是平行方法的遞迴深度。

序列遞迴：串接 k 次改善步驟，後續步驟可讀先前修改與執行紀錄。k>1 時，selector 依任務與來源比較匿名 deliverables，選擇既有產出，不生成新產出；k=1 略過此選擇階段。

由上述流程可推導 executor rollout 數為 4k（k reference＋2k candidate probes＋k successors），另有 3k 個提案／合成 sessions；選擇與反思另計。這是流程數量，不是 token、美元或延遲的固定倍數。證據：PDF pp.5–6 Figure 4 / §2.2.2。

### 2.4 任務間：累積改善庫

任務完成後，reflector 檢視完整修改過程，把採用的 harness 與有執行對照支持的經驗存入 library；經驗以 EXPERIENCE.md 記錄情境、修改、證據與適用條件。證據不足可不新增 lesson。下一題從共同 H0 開始，由 library 協助提案與合成，不自動繼承上一題所有客製設定（PDF pp.6–7 §2.3）。

## 三、實驗如何設計

| 項目 | 設定 |
| --- | --- |
| Executors | Gemini 3.5 Flash、GLM-5；權重凍結 |
| 初始 harnesses | OpenCode、Codex，構成四種配置 |
| 預設 editor | Claude Opus 4.8，執行於 OpenCode |
| 主表 | k=1；+ Recursion 為平行 k=5 相對 k=1 的平均增量 |
| 重複方式 | 三個 runs，各自洗牌任務順序；報告平均與標準差 |
| 改善庫 | 每個 benchmark stream 從空庫開始，逐任務更新 |
| 比較方法 | Base、ReasoningBank、Meta-Harness、Harness Scaling |

| Benchmark | 實際測試量 | Metric 與評分 |
| --- | --- | --- |
| WorkBuddy-Bench | Code 40、Web 35、Office 25 | 每題 checks 通過比例；包含確定性檢查與 LLM/VLM judge；跨領域 Avg. 為 macro-average |
| JobBench | 34 | 專業工作交付物的 weighted rubric score；Claude Opus 4.8 judge |
| LAB | 100 | 法律任務逐準則通過率 Partial；Claude Sonnet 4.6 judge |

證據：PDF p.7 §3.1、pp.18–20 Appendix B。測試並非使用這些 benchmark 的全部題目。評分 rubrics、答案與 verifier outcomes 保留在評估端，不作為改善流程的回饋。無 ground-truth feedback 不表示研究沒有事後評分。

Meta-Harness 使用各自獨立的訓練／開發題離線搜尋；Harness² 在線處理新任務。這個差異是方法設定的一部分，不是所有方法都使用相同訓練訊號。

## 四、三組最重要的證據

### 證據 A：四種配置的 WorkBuddy 平均皆提高

| 配置 | Base，mean (SD) | Harness² k=1，mean (SD) | k=1 增益 | k=5 平均¹ |
| --- | ---: | ---: | ---: | ---: |
| Gemini / OpenCode | 72.4 (0.9) | 77.5 (2.5) | +5.1 | 80.8 |
| Gemini / Codex | 74.5 (1.6) | 77.5 (2.0) | +3.0 | 80.4 |
| GLM-5 / OpenCode | 64.2 (1.4) | 75.7 (2.2) | +11.5 | 79.6 |
| GLM-5 / Codex | 64.4 (1.2) | 75.3 (1.3) | +10.9 | 78.3 |

¹ k=5 由原表 k=1 平均加上 + Recursion 增量計算；原文未在該列提供 k=5 SD，不自行推估。

原文證據：PDF p.8 Table 1。GLM-5 / OpenCode 的最強比較基準 Meta-Harness 為 69.7 (0.6)，Harness² k=1 高 6.0 分；Gemini / OpenCode 的 Meta-Harness 為 76.4 (0.7)，Harness² 高 1.1 分。摘要的 +11.5 是對 base 的絕對分數差，不是相對提升 11.5%。

作者結論：改善 harness 能讓較弱 executor 的某些配置超過較強模型的原始配置，例如 GLM-5 / OpenCode 75.7 高於 base Gemini / OpenCode 72.4。

工程推論：在決定更換模型前，值得量測既有 harness 的流程缺陷。但主實驗有較強 editor 與額外計算，不能推論其成本一定低於更換模型。

### 證據 B：效果延伸到其他專業領域，亦有例外

以下均為 OpenCode；括號為 SD。

| Executor | Benchmark | Base | k=1 | 單步增益 |
| --- | --- | ---: | ---: | ---: |
| Gemini | JobBench Score | 31.5 (1.1) | 36.0 (2.3) | +4.5 分 |
| GLM-5 | JobBench Score | 20.6 (2.0) | 24.2 (0.9) | +3.6 分 |
| Gemini | LAB Partial | 79.9 (0.6) | 81.9 (0.6) | +2.0 百分點 |
| GLM-5 | LAB Partial | 59.6 (2.7) | 71.4 (2.9) | +11.8 百分點 |

原文證據：PDF p.9 Table 2。JobBench 的 +4.5 屬於 Gemini / OpenCode；LAB 的 +11.8 屬於 GLM-5 / OpenCode，不能合成單一平均效果。

例外：LAB 的 Gemini / Codex，Harness² k=1 為 75.6 (4.5)，低於 Meta-Harness 的 76.5 (2.1)。WorkBuddy GLM-5 / Codex 的 Code 單域，Meta-Harness 76.8 也高於 Harness² k=1 的 74.9。四配置的平均優勢不等於每一格皆最佳。

原文補充：同模型同時當 executor 與 editor 的 k=2 實驗中，五個模型都改善（PDF p.11 Figure 8）。這支持增益不必依賴較強 editor，但不能把該結果的幅度套回 k=1 主表。

### 證據 C：消融與移轉支持「改善過程能累積經驗」

Gemini 3.5 Flash / OpenCode，k=1：

| 方法 | WorkBuddy | JobBench |
| --- | ---: | ---: |
| 完整 Harness² | 77.5 | 36.0 |
| 移除 library | 74.4 | 34.2 |
| 移除 scoped editor，改為兩個 generic editors | 74.6 | 32.3 |
| 移除 composition，改為候選擇一 | 75.9 | 34.1 |

原文證據：PDF p.10 Table 3。Scope 消融維持候選數、輸入證據與 probe 預算，支持 INV/DOM 分工的價值；這不是單純多問一次的比較。每列為各自消融，損失不能相加，也沒有足夠資訊自行宣稱統計顯著性。Composition 消融改變流程，不自動等於完全相同總 token。

PDF p.11 Figure 7 另顯示：

- 不做新的 reference/probe、也無 library 的直接修改，Code 從 base 73.4 降至 63.2，Web 69.9 降至 65.0。
- 用 k=5 累積的 library 支援直接修改，Code / Web / Office 為 74.3 / 74.3 / 79.1；相對 base 增加 0.9 / 4.4 / 5.1 分。前期建庫仍付過探索成本。
- JobBench library 支援 WorkBuddy k=1，Code 76.5→77.1、Web 76.4→76.9、Office 79.5→82.4，分別增加 0.6 / 0.5 / 2.9 分。

作者結論：更豐富的任務內探索，能產生有助於後續改善的可重用經驗。

工程推論：應保存可執行元件、修改原因與適用條件，而不只保存成功答案。這些結果仍限於已測試的專業任務族，不保證任意企業資料或不同模型皆受益。Figure 11 的排除自身 entry 後重新評估屬回溯重用測試，使用較晚任務經驗，不能算成原始線上時點收益。

## 五、成本與適用邊界

PDF pp.9–10 Figures 5–6：Gemini / OpenCode 的平行 k=1→5，使 Code 76.5→80.8、Office 79.5→83.4。WorkBuddy 平均對 base 增益 +5.1→+8.4；JobBench +4.5→+7.5。兩個 Harness² 設定位於所測方法的經驗 Pareto frontier。

Figure 6 計算交付前的總 tokens，包含 execution 與 improvement；它不能直接證明美元成本或端到端延遲較低，也不代表已找到普遍最優預算。任務後反思與持續經驗庫管理還需納入部署成本。原表的 #Steps 不能代表整個搜尋與編輯流程的總成本。平行化可縮短等待時間，但取決於併發資源、模型服務限制與各 rollout 長度。

PDF pp.25–26 Figure 14 提供重要失敗案例：WB-Code 的候選有 95、85、85、80、55，selector 選到 55，reference 為 96。這些都是事後評分；線上 selector 看不到分數。另有錯誤 key 規則或 cache 假設傳到多個候選，及 judge 未返回有效結果的情況。成功選出產出不等於確立修改的因果效果；候選多也可能共享同一錯誤前提。

作者明列限制（PDF p.29 Appendix F–G）：額外推論成本；執行波動與共同模型偏誤；無正確性與跨任務轉移保證。未來需經驗整理、過時規則更新、更佳檢索，以及依任務難度自適應探索預算。

## 六、對企業 Agent 平台的建議（本報告推論）

1. 先用可重放、可隔離副作用的代表性內部任務比較 Base、Best-of-k 與 Harness²。相同模型、輸入、環境、預算口徑要保留。
2. 同時量測交付品質、總 tokens／成本、端到端延遲、回歸率，以及候選最佳表現與最終選擇的落差。
3. 共享經驗保留版本、任務來源、執行差異、適用條件與不確定性。先釐清敏感資料如何隔離，再評估跨使用者重用。
4. 將固定企業權限與可修改的作業指引分開實作；文字 guardrails 不能取代 sandbox、網路限制或工具權限。
5. 寫入外部系統、發送通知或不可逆操作不能在每個 probe 重複發生；先建立測試副本或 dry run 路徑。

這些是建議的驗證設計，並非本論文已證明的企業部署能力。本文沒有量測使用者關注的服務啟動時間，也不能據此替 OpenCode / Codex 的 sandbox 或 hook 支援度排名。

## 七、外部評論與推薦閱讀

### Zhongzhu Zhou，2026-10-24

文章：https://www.zhongzhuzhou.org/blog/2026-10-24-harness2-technical-review-en/

其解讀肯定「以執行證據進行工作流程搜尋並累積經驗」，提醒 library 更新不等於無限自主改進，行為對照也不是 correctness oracle。這篇適合閱讀方法定義、消融與因果歸因限制；文中的額外診斷建議是評論者提案，不是作者新增實驗。

### Nicolás Torres，2026-10-19

文章：https://ntorres.dev/blog/harness2-google-recursive-agent-harness

其工程建議是保存完整 harness 與 EXPERIENCE.md，並明確區分腳本 demo 展示的流程能力與模型自主發現修改的能力。推薦閱讀其元件保存與預算觀點；版本相關的實作問題需回到當前官方程式碼核對。文中將 JobBench +4.5 歸入 GLM-5 的敘述與原論文 Table 2 不一致，本報告依 Table 2 使用 Gemini / OpenCode。

這兩篇是個人技術解讀，並非獨立完整 benchmark 復現，也不能代表社群共識。簡報最後一頁附原論文 Figure 5 作討論對照圖，不將其冒充為外部實測。

## 八、證據清單

| 主張 | PDF 頁碼 | Figure／Table | Metric、baseline、條件 |
| --- | --- | --- | --- |
| 八個可編輯元件 | 3、21–22 | Fig.2、Table 5 | 架構定義，非效果比較 |
| 雙層改善與 PPC | 4–7 | Fig.3–4 | Frozen model；相同模型／環境候選對照；線上無正解回饋 |
| WB 四配置改善 | 8 | Table 1 | 領域 macro-average；Base / RB / Meta / Scaling；3 shuffled runs |
| JB 與 LAB 改善及例外 | 9 | Table 2 | JB weighted score；LAB Partial；相同四配置 |
| k 與品質／總tokens | 9–10 | Fig.5–6 | Gemini / OpenCode；k1–5；Pareto 為觀測方法內比較 |
| 三項機制消融 | 10 | Table 3 | Gemini / OpenCode、k1；scope 保留候選數與probe budget |
| 經驗庫與跨benchmark移轉 | 10–11 | Fig.7 | 直接編輯無新probe；JB→WB k1；條件不可混用 |
| 同模型 executor/editor | 11 | Fig.8 | OpenCode、k2；五個模型 |
| 選擇與假設傳播失敗 | 25–26 | Fig.14 | 選定案例，事後評分，非總體失敗率 |
| 作者限制與未來方向 | 29 | Appendix F–G | 成本、變異、共享偏誤、經驗庫管理、adaptive recursion |

頁碼為 PDF 實體頁碼，與印刷頁腳一致；重要主結果已直接核對原圖表及圖說。
