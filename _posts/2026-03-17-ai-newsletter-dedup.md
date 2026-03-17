---
title: "AI Newsletter 去重機制優化：如何避免每天都報導相同的 AI 新聞"
date: 2026-03-17 21:00:00 +0800
categories:
  - tech
tags:
  - ai-newsletter
  - automation
  - workflow
---

## 前言：被「重複新聞」疲勞轟炸的困擾

每天早上生成 AI Daily Newsletter 本來是一件愉快的事，但一段時間後，我發現一個問題：**明明上週才報導過的主題，這週又出現了**。

例如：
- 「GPT-5.4 發布」連續出現 3 天，每次只是不同媒體的報導
- 「Anthropic 國防合約爭議」炒了整週，來源文章換湯不換藥
- 「NVIDIA GTC 預熱」相關新聞反复出現

讀者看到相同的議題天天轟炸，不僅疲乏，也會質疑 Newsletter 的專業度。更糟的是，這浪費了寶貴的 5-6 個名額，讓真正「今天發生的新鮮事」被排擠掉。

**這就是為什麼我決定建立 news-ledger.json 去重機制**——不是為了複雜，而是為了讓每天的 Newsletter 真的值得閱讀。

---

## 核心設計：指紋系統 + 7 天回溯

我借用了「指紋」（fingerprint）的概念，每則新聞被賦予一個唯一識別碼：

```
fingerprint: "Broadcom|broadcom-ai-revenue|2026-03-05"
```

格式是 `公司|標準化主題|日期`。這樣只要比對指紋，就能快速判斷：

1. **完全相同** → 直接排除
2. **相同公司 + 相同主題，但不同時間** → 視為「延續報導」，需要額外理由才能入選
3. **相同公司 + 類似主題（7 天內）** → 需要說明「這次有什麼新意」

```json
"duplicate_policy": {
  "lookbackDays": 7,
  "sameFingerprint": "reject",
  "sameTopic": "reject_unless_material_update",
  "sameCompanyTopicWindowDays": 3,
  "allowIf": [
    "official_launch",
    "major_feature_or_benchmark_update",
    "pricing_or_spec_disclosure",
    "new_partnership_or_distribution",
    "regulatory_or_governance_change",
    "senior_leadership_change",
    "clear_new_productization_or_real_world_deployment"
  ]
}
```

---

## reason_picked 強制格式：把「為什麼選它」寫清楚

這是最關鍵的改變。之前、新聞入選的理由是自由的文字描述，容易寫得模糊。這次我強制規定格式：

```json
"reason_picked": {
  "code": "official_launch",
  "secondary": ["official_source", "benchmark_detail"],
  "note": "首次正式發布，含 benchmark 細節"
}
```

- **code**：只能從 10 個允許的值中選擇（如 `official_launch`、`material_followup`）
- **secondary**：輔助標籤，最多 3 個
- **note**：一句話說明「今天為什麼值得選」

如果今天選的新聞和近 7 天主題相近，`note` 必須寫出明確的新意，否則無法入選。這逼迫我在篩選時多思考一步：**這則新聞真的值得佔用今天的名額嗎？**

---

## 實際效果

機制上線後（2026-03-09），成效顯著：

- **不再有連續重複**：GPT-5.4、Anthropic 國防爭議、infra 軍備競賽等主題在重複出現時被正確攔截
- **選題更多元**：因為名額不被舊聞佔據，能納入更多新公司、新應用的新聞
- **質量提升**：每次入選都必須有明確理由，讀者能感受到「今天真的有事發生」

---

## 未來優化方向

1. **formalize reason_picked as structured object**（已實現）
2. 加入更多 secondary 標籤，如 `education`、`healthcare`、`finance` 等產業別
3. 考慮對「延續報導」建立分數機制，自動判斷是否值得入選

---

**結論**：去重機制不是「限制」，而是「把關」。讓每天的 Newsletter 保持「今天最值得讀」的品質，才是對讀者最大的尊重。

---

*本文同步發布於 [AI Daily Newsletter](https://brunolin-tw.github.io/ai-newsletter/)*
