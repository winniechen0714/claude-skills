# 購案商機每日摘要管理

管理「購案商機每日更新摘要」的 GitHub Actions 自動化排程，支援查看執行狀態、手動補發，以及診斷問題。

---

## 專案資訊

```
GitHub Repo:   winniechen0714/asana-daily-digest
Workflow 檔案: .github/workflows/daily-digest.yml
腳本路徑:      digest.py
排程時間:      每日 UTC 22:00（台灣時間 06:00）
掃描區段:      台灣時間「昨日 06:00」～「今日 06:00」（固定錨點，不受 cron 延遲影響）
發送目的地:    Slack #購案商機每日更新
```

---

## 執行流程

### 情境 A：查看最近執行狀態

```bash
gh run list --repo winniechen0714/asana-daily-digest --workflow=daily-digest.yml --limit 5
```

輸出格式化摘要，包含：
- 最近 5 次執行的狀態（✅ success / ❌ failure / 🔄 in_progress）
- 觸發時間（台灣時間）
- 執行耗時

### 情境 B：手動補發（今日）

```bash
gh workflow run daily-digest.yml --repo winniechen0714/asana-daily-digest
```

觸發後等待約 30 秒，再查詢執行結果確認成功。

### 情境 C：查看最近一次執行 Log

```bash
gh run view --repo winniechen0714/asana-daily-digest --log $(gh run list --repo winniechen0714/asana-daily-digest --limit 1 --json databaseId --jq '.[0].databaseId')
```

### 情境 D：診斷失敗原因

1. 取得最近失敗的 run ID
2. 查看詳細 log，找出錯誤訊息
3. 常見原因：
   - `ASANA_TOKEN` 過期 → 更新 GitHub Secret
   - `SLACK_WEBHOOK_URL` 失效 → 更新 Slack Webhook
   - Asana API rate limit → 等待後重試

---

## 排程設定說明

| 項目 | 值 | 說明 |
|------|-----|------|
| Cron | `0 22 * * *` | UTC 22:00，避開 UTC 00:00 高峰 |
| 台灣時間 | 06:00 | 上班前送達 |
| since | 昨日 06:00 TW | 固定錨點，不受延遲影響 |
| until | 今日 06:00 TW | 固定錨點 |

> ⚠️ GitHub Actions cron 在 UTC 00:00 整點有嚴重延遲（實測曾達 144 分鐘）。
> 改為 UTC 22:00 後通常在 5 分鐘以內觸發。
> 詳見：`docs/risks.md` RISK-003

---

## 追蹤內容說明

| 類別 | 說明 |
|------|------|
| 💰 前日開立發票總額 | 偵測「售後服務(已開立發票、未收款)」區段有移動的任務（移入或移出），加總實際成交金額(稅後) |
| 🆕 新建立的任務 | 新建的 Task（排除子任務），依建立者姓名排序 |
| 🔄 任務階段移動 | Section 變更紀錄（僅限購案商機專案，不含開立發票區段） |
| 💬 新評論 | 任務內新增的人工評論（排除系統自動化評論）；子任務顯示「母任務 > 子任務」 |

---

## 相關常數

```
ASANA_PROJECT_GID:     1208705115282620  # 購案商機
ASANA_WORKSPACE_GID:   1202280977140605  # 思邁智能股份有限公司
INVOICE_SECTION_NAME:  售後服務(已開立發票、未收款)
AMOUNT_FIELD_GID:      1211061298683016  # 實際成交金額(稅後)
```

## GitHub Secrets 設定位置

```
Repository → Settings → Secrets and variables → Actions
- ASANA_TOKEN       Asana Personal Access Token
- SLACK_WEBHOOK_URL Slack Incoming Webhook URL
```
