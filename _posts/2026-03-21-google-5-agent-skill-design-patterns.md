---
title: "Google 推薦 5 大 Agent Skill 設計模式"
date: 2026-03-21 17:57:00 +0800
categories:
  - tech
tags:
  - ai
  - agent
  - skills
  - prompt-engineering
  - google
---

最近看到 Google Cloud Tech 整理的「5 大 Agent Skill 設計模式」，我覺得這篇有價值的地方，不是又多了幾個新名詞，而是它把很多人做 AI Agent 時真正卡住的點講清楚了：**問題通常不在 SKILL.md 格式，而在 Skill 內容到底怎麼設計。**

很多開發者一開始做 Skill，很容易把注意力放在 YAML、目錄結構、命名方式，甚至是 references/、assets/ 要怎麼擺。但如果從實務角度看，這些都只是包裝。真正決定 Agent 能不能穩定做事的，還是 Skill 裡面的規則、流程、模板，以及它在什麼時機被載入。

![Google Agent Skill 設計模式主視覺](https://image-cdn.learnin.tw/bnextmedia/image/album/2026-03/kk86-1773977456.jpg?w=900&output=webp)

我自己的理解是，Skill 的本質不是一段比較漂亮的 prompt，而是把 Agent 的工作方式模組化。你要先定義好：什麼時候啟用、要讀哪些參考資料、該怎麼執行、輸出要長什麼樣、在什麼條件下不能往下一步。Google 這次整理的 5 種模式，剛好就是在回答這些問題。

## 1. Tool Wrapper：把工具知識從大 Prompt 裡拆出來

第一種模式是 **Tool Wrapper**。它很適合拿來封裝某個特定工具、API、SDK，或框架的使用規則。

例如你在做 FastAPI、Slack SDK、Stripe API，或公司內部服務整合時，通常會有一堆慣例、最佳實務、錯誤處理方式。如果把這些東西全部塞進系統提示詞，久了只會越來越肥，也越來越難維護。Tool Wrapper 的做法就是把這些規則拆成獨立 Skill，需要時再載入。

它的重點不複雜：

- 明確定義什麼情境該啟用這個 Skill
- 指定要載入哪些 references 文件
- 要求 Agent 在寫程式或 review 時把這些規則視為高優先級

這種設計很像把「工具說明書」變成一個可重複使用的模組。我覺得對多 API、多框架的團隊特別有用，因為你不用讓每個 Agent 都背完整本百科全書，而是按需載入就好。

## 2. Generator：先固定骨架，再讓 AI 填內容

第二種模式是 **Generator**。它解決的不是 Agent 不懂，而是 **每次輸出的結構都不太一樣**。

這件事其實很常見。明明都是要寫技術報告、API 文件、SQL 查詢，結果模型每次生出來的章節都不同，有時漏段落，有時亂加內容。這時候，與其一直補 prompt，不如直接給它模板。

Generator 的典型做法是：

- `assets/` 放輸出模板
- `references/` 放風格指南
- `SKILL.md` 規定 Agent 要先讀風格、再讀模板、補齊必要變數後，照模板完整輸出

它的價值很直接：把自由生成改成受約束生成。這樣產出會更穩，也比較適合團隊協作。

如果你的 Agent 常常要產出固定格式內容，例如報告、文件、企劃、程式碼骨架，我會覺得 Generator 幾乎是必備模式。

## 3. Reviewer：把檢查標準獨立管理

第三種模式是 **Reviewer**。這個模式我很喜歡，因為它把兩件事情拆開來：

- 要檢查什麼
- 怎麼檢查

也就是說，真正的審查標準放在 checklist 或 references 裡，`SKILL.md` 則只負責定義流程，例如先讀 checklist、逐條套用、分級問題嚴重程度、解釋原因、給修正建議。

這樣的好處是，當你想更新規則時，不需要整個流程全部重寫。你可以很容易把同一套 Reviewer 架構，換去做不同任務：

- Python code review
- 安全性檢查
- 文件品質審查
- SEO 結構檢查
- 引文或事實核實

從工程角度看，這其實是把 prompt engineering 往「規則系統化」推進一步。

## 4. Inversion：先把需求問完整，再開始動手

第四種模式是 **Inversion**。它最關鍵的地方在於：**不讓 Agent 太早開始回答。**

一般情況下，使用者丟一段需求，Agent 就急著開始設計、規劃、甚至直接產出。但很多失敗案例，問題正是出在前面的資訊收集根本不完整。Inversion 的做法是反過來，由 Agent 主導提問，而且會設「閘門」：問題沒問完，就不能開始設計。

典型流程會像這樣：

- 先問問題到底是什麼
- 再問使用者是誰
- 再問規模與限制
- 最後才進入規劃或生成階段

這很適合用在：

- 專案規劃
- 系統設計
- 需求訪談
- 表單／訂單流程
- 任何需要多個前置條件才能開始的任務

如果你常覺得 AI 回得很快，但方向常常歪，Inversion 很值得導入。它本質上是在降低「資訊不完整時亂猜」的機率。

## 5. Pipeline：把複雜工作拆成不能跳步的流程

第五種模式是 **Pipeline**。這是把複雜任務切成多個步驟，每一步都設放行條件，前一步沒完成就不能往下走。

這種模式很適合多階段工作，例如：

1. 先解析原始資料
2. 再產生中間結果
3. 再組裝成最終文件
4. 最後再做品質檢查

它的強項在於把 Agent 從「一次吐出完整答案」改成「依序執行工作流」。這樣不只比較容易 debug，也能在中途插入人工確認，或加 Reviewer 做最後把關。

如果任務本身有順序依賴，像是文件生成、PDF 結構化、API 文件整理、資料摘要流程，Pipeline 幾乎一定比單一 prompt 更可靠。

## 真正實用的地方：這 5 種模式可以混搭

這 5 種模式不是互斥的，我覺得真正好用的地方反而是可以組合。

例如：

- **Inversion → Generator**：先把需求問清楚，再照模板輸出
- **Pipeline + Reviewer**：先跑完整流程，最後再做結構化審查
- **Multiple Tool Wrappers**：把不同 API 或框架拆成不同 Skill，按需載入

這比單純研究 prompt wording 有用得多，因為它開始碰到 Agent 系統設計的核心：上下文怎麼切、流程怎麼控、規則怎麼維護、輸出怎麼驗證。

## 我的看法：Skill 的真正價值，是讓 Agent 從「偶爾很強」變成「穩定可用」

我看完這篇後最大的感想是，Skill 真正重要的，不是讓模型看起來更聰明，而是讓它的行為更可控。

你可以把它理解成：

- Tool Wrapper 管工具知識
- Generator 管輸出形狀
- Reviewer 管品質標準
- Inversion 管資訊收集
- Pipeline 管執行順序

當這些東西被拆開之後，Agent 才比較有機會從 demo 走向真的能落地的工作流。

對個人開發者來說，這也很實際。你不一定要先把整套 Skill 手工寫完，而是可以先把任務類型對應到合適模式，再讓 AI 協助生成 `SKILL.md`、`references/`、`assets/` 初稿。重點不是追求完美格式，而是先把流程和標準建立起來。

## 小結

Google 整理的這 5 大 Agent Skill 設計模式，我認為最值得記住的不是名字，而是背後的設計思路：

- 不要只關心格式
- 要重視 Skill 的觸發條件與上下文管理
- 把規則、模板、流程、審查標準拆開管理
- 讓 Agent 在正確時機讀取正確資訊
- 用流程控制提高產出穩定性

如果未來 AI Agent 要真的成為工作流的一部分，而不只是聊天介面裡偶爾靈光一現的功能，這種 Skill 設計方法只會越來越重要。

---

**參考來源**
- 原文：[Skill 檔案應該怎麼寫？Google 提出「5大 Agent Skill 設計模式」](https://www.bnext.com.tw/article/90356/ai-agent-skill-is-cool-due)
- Google Cloud Tech X 貼文：https://x.com/GoogleCloudTech/status/2033953579824758855
- 資料整理：Google Cloud Tech（經數位時代報導）
