---
title: "LLM不只會問答，還會開始替你編譯知識，從Karpathy提出的LLM Wiki到Graphify竄起"
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

 Andrej Karpathy 在 X 上分享的 LLM Knowledge Bases，其實不是又一個「RAG 變體」的技巧。他描述的核心是：**讓 LLM 不再只是在 query 時從原始文件裡重新檢索，而是先把 raw sources 編譯成一個持續演化的知識庫**。這個觀點把個人知識庫的工作模式從「檢索-first」轉向「編譯-first」。

在傳統 RAG 流程裡，每次問問題都要把文件切成 chunk、轉成 embedding、存進向量資料庫，然後做最近鄰搜尋。這種做法在規模小時可以接受，但有幾個結構性缺點：chunk 邊界任意、重新檢索浪費 token、多文件推理每次都要重做，且難以累積「上次已經想清楚的東西」。

Karpathy 的做法恰恰相反：先把文章、論文、專案、圖片等資料放進 `raw/` 目錄，然後讓 LLM 增量「編譯」它們。產出的不是向量，而是結構化的 markdown wiki——包括摘要、概念頁、實體頁、比較頁、交叉連結與索引。這些 artefact 是可讀的、可審計的、且會隨著新資料與 lint 持續更新。更重要的是，當你對這個 wiki 提問時，LLM 不是從頭開始讀 raw files，而是直接在已編譯好的知識層上搜尋、合併與回饋，問答結果甚至可以再寫回 wiki，形成複利效應。在他的個人使用案例中，這種方法在約 100 篇文章、40 萬字的規模下，已經證明不需要複雜的 RAG 堆疊。

Graphify 的出現則把這個「編譯」概念往前推了一步。它不滿足於產出可讀的 markdown 頁面，而是直接把程式碼、文件、論文、圖表、影音等多模態內容編譯成 **queryable knowledge graph**。Graphify 的 pipeline 包含兩個關鍵階段：首先使用 tree-sitter 做決定性的 AST 抽取（不涉及 LLM，快速且私密），其次呼叫 Claude 子代理對非結構化來源做語意擷取。最後將節點與邊合併成 NetworkX 圖，套用 Leiden 社群偵測做 clustering，同時標記每條關係為 EXTRACTED、INFERRED 或 AMBIGUOUS，以保留來源與信心度資訊。輸出包括互動式的 graph.html、可持續查詢的 graph.json，以及人類可讀的 GRAPH_REPORT.md。根據實測，這樣的結構讓每次查詢所需的 token 數可以減少達 71.5 倍，因為助理不再需要重讀原始檔，而是直接在已編譯好的圖上遍歷。

從工程角度來看，這兩件事一起告訴我們：AI 助手的真正瓶頸不再只是模型能力，而是 **知識如何被組織與重複使用**。LLM Wiki 讓我們看到知識可以被編譯成穩定的 markdown 層；Graphify 則證明這層可以進一步結構化為圖，讓推理不只停留在文字摘要，而能走訪節點與關係。對於長期研究、大型專案或個人知識庫來說，這種「先編譯、後查詢」的工作流不只是較省 token，更重要的是它把知識變成一種可持續累積的工程 artefact，而不是每次都要從零開始的重複勞作。

事實上，這不意味著 RAG 已過時。在需要即時更新、海量資料、細緻權限控制的企業場景裡，向量資料庫與混合搜尋仍有其位置。但對於個人筆記、專案內部理解或 agentic coding workflow 而言，先投資編譯出一個乾淨、結構化的知識層，往往比每次臨時檢索更實用、也更持久。真正值得追的，可能不只是更大的模型，而是如何讓我們的知識在每次互動後都變得更好用。