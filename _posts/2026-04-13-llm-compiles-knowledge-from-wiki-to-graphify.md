---
title: "LLM 不只會問答，還開始替你編譯知識：從 Karpathy 的 LLM Wiki 到 Graphify"
date: 2026-04-13 09:00:00 +0800
categories:
  - AI
  - tech
tags:
  - LLM
  - Knowledge-Base
  - RAG
  - Graphify
  - Context-Engineering
  - Agentic-Development
---

![LLM 從 wiki 層走向 knowledge graph 的知識編譯示意圖](/assets/images/posts/2026-04-13-llm-wiki-graphify-hero.png)

最近 Andrej Karpathy 在 X 上提到的 **LLM Knowledge Bases**，我覺得有意思的地方，不是它又提出一個新的 RAG 變形，而是它重新定義了一件事：**LLM 處理知識，到底應該是查資料，還是先把資料編譯成更好用的形式。**

這兩種做法看起來很像，但工作流其實差很多。

傳統 RAG 的思路大家都很熟。文件先切 chunk、做 embedding、放進向量資料庫，等有人發問時，再把相關內容撈回來交給模型回答。這套方法不是不能用，但它一直有幾個結構性的問題：

- chunk 邊界本來就很任意  
- 同樣的推理過程，常常每次 query 都要重做一次  
- 多份文件之間的關聯，未必真的有被整理出來  
- 模型上次已經整理過的脈絡，下一次往往還是得重新推一次  

換句話說，這種做法比較像是把知識維持在原料狀態，需要時才即時加工。

Karpathy 提的方向剛好相反。  
他不是讓 LLM 每次都回到 raw sources 重新翻資料，而是先把文章、論文、專案、圖片等內容放進 `raw/`，再讓 LLM 持續做增量式的「編譯」。

編譯出來的結果不是向量，而是一層**人可以直接閱讀，也能持續維護的 markdown wiki**。裡面會有摘要、概念頁、實體頁、比較頁、交叉連結和索引，而且會隨著新資料加入持續更新。

這件事真正有價值的地方在於，之後再提問時，LLM 已經不需要每次都從原始資料重新讀起。它可以直接站在這層已經整理過的知識上做搜尋、合併和回答。甚至回答本身，還可以再寫回 wiki，讓整個知識庫越用越完整。

這就不是單純的檢索，而比較像是**把知識當成一個可持續演進的工程產物來維護**。

而且 Karpathy 的案例不是停留在概念層。  
在大約 100 篇文章、40 萬字的規模下，這套方法已經足以運作，而且不需要額外疊很重的 RAG 基礎設施。這點其實很關鍵，因為它代表很多個人知識庫或中小型研究工作，未必真的需要那麼複雜的系統。

如果說 LLM Wiki 是把知識編譯成 markdown，**Graphify** 則是把這個概念再往前推一步。

它不只想把資料整理成可讀頁面，而是直接把程式碼、文件、論文、圖表、影音等來源，編譯成一個**可以查詢的 knowledge graph**。

這就不是單純在做筆記，而是把知識結構化到節點與關係層級。

Graphify 的 pipeline 很工程導向。大致分成兩段：

第一段是用 tree-sitter 做 AST 抽取。這部分是 deterministic 的，不依賴 LLM，所以速度快，也比較保密。  
第二段才交給 Claude 子代理去處理那些比較非結構化的來源，做語意層的擷取與連結。

最後它把節點和邊整合成 NetworkX 圖，再用 Leiden 社群偵測做 clustering。更重要的是，它不會把每條關係都包裝成同樣可信，而是明確標示為：

- `EXTRACTED`
- `INFERRED`
- `AMBIGUOUS`

也就是說，它會保留每條關係的來源性質與信心程度。這點很實際，因為很多知識圖譜看起來很完整，但實際上哪些是明確抽取、哪些只是模型推論，往往不夠透明。

Graphify 的輸出也不只一種，包括互動式的 `graph.html`、可持續查詢的 `graph.json`，以及人類可讀的 `GRAPH_REPORT.md`。  
根據它的實測，這種先編譯再查詢的方式，能把每次查詢所需的 token 大幅壓低，原因不難理解：助理不需要每次都重讀原始檔，而是直接在已經整理好的圖結構上遍歷。

把這兩件事放在一起看，我覺得最值得注意的，不只是某個工具本身，而是一個很明顯的方向：

**AI 助手的瓶頸，正在從模型能力，逐漸轉向知識如何被整理、保存與重複使用。**

以前大家最常問的是模型夠不夠強。  
但現在更值得問的問題也許是：你讓模型接觸知識的方式，是不是還停留在每次都從零開始。

LLM Wiki 讓我們看到，知識可以先被整理成一層穩定的 markdown 層。  
Graphify 則更進一步證明，這層知識還可以再被結構化成圖，讓推理不只停留在文字摘要，而是能沿著節點與關係走。

對長期研究、複雜專案，或個人知識庫來說，這種「先編譯、後查詢」的模式，價值不只是省 token。更大的差別在於：**知識不再只是一次次被消耗的上下文，而是會累積、會演進、會越用越有價值的資產。**

當然，這不代表 RAG 已經過時。  
如果場景是即時更新資料、海量文件、細緻權限控制，或者企業級搜尋系統，向量資料庫和混合搜尋仍然很有位置。

但如果你的工作更接近下面這些：

- 個人研究筆記  
- 專案知識整理  
- agentic coding workflow  
- 小團隊的文件理解與脈絡延續  

那麼先花力氣把知識編譯成乾淨、穩定、可演進的結構層，通常會比每次 query 才臨時檢索更實用，也更持久。

我現在越來越覺得，接下來真正值得追的，也許不只是更大的模型，而是**如何讓知識在每一次互動之後，都變得更容易被下一次使用。**  
這才比較像是 AI 真正進入工作流，而不只是多了一個會聊天的介面。

#### 參考來源
1. Andrej Karpathy: LLM Knowledge Bases (X post) – https://x.com/karpathy/status/2039805659525644595  
2. Karpathy's LLM Wiki (gist) – https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f  
3. VentureBeat: Karpathy shares 'LLM Knowledge Base' architecture – https://venturebeat.com/data/karpathy-shares-llm-knowledge-base-architecture-that-bypasses-rag-with-an  
4. Graphify GitHub – https://github.com/safishamsi/graphify  
5. Graphify official site – https://graphify.net/
