# 購案商機週報產生器

自動從 Asana 購案商機專案抓取本週（週一～週日）資料，整理成業務週會 Overview，並更新到指定的 Notion 頁面。

---

## 執行流程

### 步驟一：確認目標 Notion 頁面

如果使用者沒有在指令後面直接提供 Notion 頁面 URL 或 ID，**必須先詢問使用者**：

> 「請提供要寫入的 Notion 頁面 URL 或 Page ID，例如：
> `https://www.notion.so/2026-03-09_-31bddaf9032b80e2a1ebc2fadee649b8`
> 或直接提供 Page ID：`31bddaf9032b80e2a1ebc2fadee649b8`」

等使用者提供後再繼續。

### 步驟二：執行週報腳本

執行以下指令抓取 Asana 資料：

```bash
python3 /Users/fongmingchong/asana-daily-digest/weekly_report.py
```

腳本會輸出 JSON 格式的週報資料，包含：
- `period`：週報期間（週一 00:00 ～ 今日當下）
- `new_tasks`：本週新建商機
- `section_moves`：本週階段移動（含 forward/backward 判斷）
- `forward_count` / `backward_count`：前進/後退件數
- `poc_tasks`：目前在 POC 的案件清單
- `poc_entered`：本週新進入 POC 的案件
- `invoice_entered`：本週進入售後服務（開票）的案件
- `invoice_total`：本週開立發票合計金額
- `stuck_tasks`：停滯超過 14 天的進行中案件
- `closed_this_week`：本週結案

### 步驟三：解讀資料並撰寫週報內容

根據 JSON 資料，撰寫以下 Notion Markdown 格式的週報內容：

```
## 📊 購案商機週報（{period}）

### 本週概覽
[表格：新建商機數、有階段移動案件數、POC 中案件數、開立發票合計、停滯案件數]

---

### 💰 本週開立發票明細
[如有開票，列出明細與合計；如無則標示「本週無開立發票」]

---

### 🔄 本週案件階段移動
[依 forward / backward / 其他 分類呈現，附 Asana 超連結]
[若後退件數 > 前進件數，加上 ⚠️ 提醒說明是否為業務停滯或後台流程修正]

---

### 🔴 停滯中（進行中且超過 14 天未更新）
[表格：案件名稱、目前階段、停滯天數；無則標示「本週無停滯案件」]

---

### 🎯 目前 POC 中的案件（N 件）
[列表，附 Asana 超連結]
[若本週有新進入 POC，額外標示 ✨ 新進入]

---

### 🆕 本週新建商機（N 件）
[表格：案件名稱、目前階段、建立者；附 Asana 超連結]

---

*資料來源：Asana 購案商機專案，由 Claude Agent 自動抓取（{今日日期}）*
```

**注意事項：**
- 所有 Asana 任務連結格式：`https://app.asana.com/0/1208705115282620/{task_gid}`
- 使用 Notion Markdown 格式（表格、清單、連結）
- 語言：繁體中文

### 步驟四：寫入 Notion 頁面

使用 `notion-fetch` 先取得頁面目前內容，判斷是否已有週報區塊：

**情境 A：頁面已有舊的週報區塊（`## 📊 購案商機週報`）**
- 使用 `notion-update-page` 的 `replace_content_range` 指令，將舊週報區塊替換成新內容
- `selection_with_ellipsis` 使用週報標題的前後幾個字定位

**情境 B：頁面沒有週報區塊**
- 使用 `notion-update-page` 的 `insert_content_after` 指令，找到適當位置（通常是「購案進度」標題後）插入週報內容
- 如果找不到參考位置，使用 `replace_content` 追加到頁面末尾前

### 步驟五：確認完成

回報執行結果：
- ✅ 成功：說明更新的 Notion 頁面、週報期間、各類別數量摘要
- ❌ 失敗：說明錯誤原因並提供處理建議

---

## 常用常數參考

```
ASANA_TOKEN: 從環境變數 ASANA_TOKEN 取得
ASANA_PROJECT_GID: 1208705115282620  # 購案商機
ASANA_WORKSPACE_GID: 1202280977140605  # 思邁智能股份有限公司
INVOICE_SECTION_NAME: 售後服務(已開立發票、未收款)
AMOUNT_FIELD_GID: 1211061298683016  # 實際成交金額(稅後)
```

## 腳本位置

```
/Users/fongmingchong/asana-daily-digest/weekly_report.py
```

## PIPELINE 階段順序（index 越大 = 越接近成交）

```
新商機未聯絡 → 商機確認中 → 需求釐清 → 初次介紹 → 需求評估 →
提供初步報價 → 簽署合作意向書或保密協定 → POC → 最終報價 →
等待訂單 → 部署交付中 → 交付與實施中 → 交付實施完成 →
售後服務(已開立發票、未收款) → 已開發票且已收款 → 詢問是否續約
```
