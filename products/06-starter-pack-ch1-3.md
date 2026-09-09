# 台灣 SME《AI 自動化啟動包》v1｜第 1–3 章正文（可下載）

適用工具：n8n（雲端或自架）＋ Google 表單／試算表＋（選用）LINE Official Account  
目標：本週跑通最小閉環「資料進來 → 寫入 → 通知人」。

---

# 第 1 章｜選題指南：先做哪一條自動化？

## 1.1 為什麼不要一次做十條
自動化的複利來自「一條跑穩再擴」。同時開三條，通常卡在授權、例外流程與沒人驗收。本包要求：**先選 1 條當本週唯一目標。**

## 1.2 ROI 三分法（打分，選最高分那條）
每條候選流程用 1–5 分自评，加總：

| 指標 | 問自己 | 高分長相 |
|------|--------|----------|
| 頻率 | 一週發生幾次？ | 每天多次 |
| 痛感 | 出錯代價？ | 漏單、客訴、老闆罵 |
| 標準化 | 步驟固定嗎？ | 幾乎同一套動作 |
| 資料已在線上 | 是否已在表單／LINE／Sheet？ | 不用先掃紙本 |
| 你控制得了工具 | 你改不改得了表單／帳號？ | 你有編輯權 |

**分數 ≥ 18：** 本週就做。  
**12–17：** 可做，但先簡化步驟。  
**＜12：** 先整理流程，不要急著上 n8n。

## 1.3 台灣 SME 最值得先做的 8 條（建議優先序）

1. **詢價表單 → Sheet 建檔 → 通知業務**（最推新手）  
2. **LINE 關鍵字 → 自動回覆＋記一筆工單**  
3. **每日／每週營運摘要推給老闆**  
4. 報名／付款提醒（課程、活動、訂金）  
5. 客戶說「進度呢」→ 查 Sheet 回模板訊息  
6. 訂單進 Sheet → 粗對帳草稿（異常才人工）  
7. 員工請假／報修表單 → 通知＋狀態欄  
8. 內容發布提醒（社群檔期表 → 到期通知）

**新手預設選 1。** 已有 LINE OA 且常被重複問同樣問題 → 選 2。老闆每天追數字 → 選 3。

## 1.4 一票否決（有這些先別做）
- 沒有任何人願意當「測試窗口」  
- 資料全在私人手機通訊錄、不肯進 Sheet  
- 想一次取代整個 ERP  
- 期望「完全不用人」卻流程每天例外十種  

## 1.5 本週任務卡（請複製填寫）
- 本週唯一流程：〔〕  
- 成功長這樣：〔例：每筆新表單 1 分鐘內 Sheet 有列＋LINE 有通知〕  
- 負責人：〔〕  
- 使用藍圖：☐A ☐B ☐C  
- 預定上線日：〔〕  

---

# 第 2 章｜準備功課（做藍圖前 30 分鐘）

## 2.1 帳號清單
- Google 帳號（可建立表單／Sheet，並能授權 n8n）  
- n8n：[n8n Cloud](https://n8n.io) 試用或自架  
- （藍圖 B）LINE Developers 帳號＋ Official Account  

## 2.2 建兩個 Sheet 分頁（建議命名）
1. `raw_inbox`：原始寫入（不要手改）  
2. `ops`：給人看的狀態（待處理／處理中／完成）  

欄位最小集（詢價例）：`timestamp | name | contact | message | source | status | owner | note`

## 2.3 n8n 基本操作（你只需要會這些）
1. 新建 Workflow → 加節點 → 連線  
2. 每個節點先用 **Test step**  
3. 右上角 **Active** 才會自動跑  
4. 出錯看 **Executions**  

## 2.4 安全底線
- 不要把 API Token 貼到公開 GitHub  
- 測試用假資料；正式前清測試列  
- LINE／Google 授權用公司帳，避免綁個人後離職全掛  

---

# 第 3 章｜三套藍圖（照做）

下列步驟以 n8n 雲端介面為準；節點名稱若隨版本微調，對功能選即可。

---

## 藍圖 A｜詢價表單 → Google Sheet → 通知（Email 或 LINE）

**成果：** 客人送出 Google 表單後，Sheet 自動多一列，負責人立刻收到通知。

### A0. 你要先有的東西
- Google 表單（欄位：姓名、聯絡方式、需求內容）  
- 連結到的 Google Sheet（或表單「回應」自動產生的試算表）  
- 通知管道二選一：Gmail **或** LINE Notify／LINE Messaging（新手先用 **Gmail** 最快）

### A1. Google 表單（5 分鐘）
1. 建立表單，打開「收集電子郵件」可選。  
2.「回應」→ 連結試算表。  
3. 送出一筆**測試資料**，確認 Sheet 有列。

### A2. n8n：Trigger
1. New workflow，名稱：`A_inquiry_to_sheet_notify`  
2. 加節點 **Google Sheets Trigger** 或 **Google Sheets**（依版本：若無 Trigger，改用 **Webhook**＋表單「通知」腳本；最穩新手路徑如下「A2-alt」）。  

**A2-alt（最穩、建議）：用「表單提交 → Apps Script 打 Webhook」或直接用 n8n《Google Sheets》定時拉新列。**  
新手最小可行：**Schedule Trigger 每 5 分鐘** + **Google Sheets（Read）** + **過濾新列**。  
若要即時：n8n 加 **Webhook** 節點，複製 URL，用 Google 表單附加「通知到 webhook」的擴充／Apps Script（見 A2-script）。

**A2-script（Apps Script 範例，貼在表單綁定的試算表擴充功能）：**
```javascript
function onFormSubmit(e) {
  const url = 'https://YOUR_N8N_WEBHOOK_URL';
  const body = e.namedValues;
  UrlFetchApp.fetch(url, {
    method: 'post',
    contentType: 'application/json',
    payload: JSON.stringify(body),
    muteHttpExceptions: true
  });
}
```
安裝後：試算表 → 擴充功能 → Apps Script → 貼上 → 儲存 → 觸發條件選「表單提交時」。

### A3. n8n：寫入營運表（可選但建議）
- 節點 **Google Sheets → Append**  
- 試算表選 `ops`  
- 對應：`name/contact/message/timestamp`，`status` 預設 `待處理`

### A4. n8n：通知
**路徑 1 Gmail（建議先跑通）：**  
- 節點 **Gmail → Send**  
- 收件者：負責人 Email  
- 主旨：`[新詢價] {{name}}`  
- 內文：聯絡方式＋需求＋Sheet 連結  

**路徑 2 LINE：**  
- 先申請 LINE Messaging API 或 LINE Notify  
- n8n 用 **HTTP Request** POST 訊息（Token 放 n8n Credentials）  
- 文案同 Email，縮短即可  

### A5. 測試清單
- [ ] 送測試表單 → 5 分鐘內（或即時）出現通知  
- [ ] `ops` 多一列且 status=待處理  
- [ ] 故意填空值：流程不要整條死掉（必要欄位在表單設必填）  
- [ ] Workflow **Active**  

### A6. 上線後第一週規則
- 每天看 `ops` 待處理是否清得完  
- 誰負責改 status：只指定一人  
- 不要在 `raw_inbox` 手改資料  

---

## 藍圖 B｜LINE 關鍵字 → 自動回覆＋工單列

**成果：** 客人在 LINE OA 打「營業時間」「報價」「人工」，自動回覆；同時 Sheet 記一筆。

### B0. 前置
1. 有 LINE Official Account  
2. LINE Developers → 建立 Messaging API Channel  
3. 取得：Channel access token（long-lived）、Channel secret  
4. Webhook URL 先留空，等 n8n 開好再填  
5. 關閉「加入好友自動回應／關鍵字回應」裡會衝突的舊規則（或只留歡迎語）

### B1. n8n Webhook
1. 節點 **Webhook**，Method POST，路徑例：`line-webhook`  
2. 複製 Production URL  
3. 貼到 LINE Developers → Messaging API → Webhook URL → Verify → Enable  
4. **重要：** 初測可先用 n8n 的 Test URL，Verify 過後再改 Production 並 Active  

### B2. 驗證簽章（建議）
LINE 會帶 `x-line-signature`。正式環境請用官方文件驗證；MVP 可先做關鍵字分流，但**上線前務必補簽章驗證**，避免被伪造請求灌水。

### B3. 解析事件
- 節點 **IF** 或 Code：只處理 `events[0].type == message` 且 `message.type == text`  
- 取出：`replyToken`、`userId`、`text`  

### B4. 關鍵字表（先做 3 個）

| 使用者輸入包含 | 自動回覆 | 是否建工單 |
|----------------|----------|------------|
| 營業／時間 | 我們營業時間為週一到五 10:00–18:00… | 否 |
| 報價／價格 | 請留下公司名＋需求，專人 1 個工作天內回… | 是 |
| 人工／客服 | 已為你轉人工，請稍候… | 是 |

n8n：**Switch** 或多個 IF。

### B5. 回覆訊息
- **HTTP Request**  
- URL：`https://api.line.me/v2/bot/message/reply`  
- Header：`Authorization: Bearer {token}`  
- Body：
```json
{
  "replyToken": "{{replyToken}}",
  "messages": [{ "type": "text", "text": "你的回覆文字" }]
}
```

### B6. 建工單（當「報價」「人工」）
- **Google Sheets Append** 到 `ops`  
- 欄位：`timestamp | userId | text | status=待處理 | owner=客服`

### B7. 通知內部
- 工單建立時同步 Gmail／LINE 通知內部群（可用第二個 Push API 或 Email）

### B8. 測試清單
- [ ] 用手機加好友，打「營業時間」有自動回  
- [ ] 打「報價」有回＋Sheet 有列＋內部有通知  
- [ ] 亂打其他字：回「已收到，稍後專人回覆」並可選建工單  

### B9. 常見坑
- Verify 失敗：URL 必須 HTTPS、節點要能秒回 200  
- 有回兩次：OA 後台關鍵字與 n8n 重複，關掉一邊  
- replyToken 只能用一次、時效短：邏輯裡先回覆再寫 Sheet，或先快速 reply  

---

## 藍圖 C｜每日營運摘要推播給老闆

**成果：** 每天固定時間，把 Sheet 裡「今日新列／待處理數」整理成一段文字推給老闆。

### C0. 前置
- 已有 `ops` Sheet（可接藍圖 A/B 的產出）  
- 老闆接收管道：Email 或 LINE  

### C1. 欄位約定
`ops` 至少要有：`timestamp`（或日期）、`status`  
建議 status 枚舉：`待處理`｜`處理中`｜`完成`

### C2. n8n 流程
1. **Schedule Trigger**：例每天 18:00（時區設 `Asia/Taipei`）  
2. **Google Sheets → Get Many / Read**：讀 `ops`  
3. **Code 或 Item Lists**：統計  
   - 今日新增筆數  
   - 目前 `待處理` 筆數  
   - （可選）完成筆數  
4. **Gmail 或 LINE Push** 送出摘要  

### C3. 摘要文案模板
```
【每日營運摘要】{{date}}
今日新進：{{newCount}} 筆
待處理：{{pendingCount}} 筆
完成：{{doneCount}} 筆
請看表：{{sheetUrl}}
```

### C4. Code 節點示例（概念）
```javascript
const items = $input.all();
const today = new Date().toLocaleDateString('en-CA', { timeZone: 'Asia/Taipei' }); // YYYY-MM-DD
let newCount = 0, pending = 0, done = 0;
for (const i of items) {
  const row = i.json;
  const status = row.status || '';
  if (status === '待處理') pending++;
  if (status === '完成') done++;
  const ts = String(row.timestamp || row.date || '');
  if (ts.startsWith(today) || ts.includes(today)) newCount++;
}
return [{ json: { date: today, newCount, pendingCount: pending, doneCount: done } }];
```
（實際欄位名請對你的 Sheet 調整；日期格式不一致時先統一成文字日期。）

### C5. 測試清單
- [ ] 手動 Execute 一次，老闆收得到  
- [ ] 故意留 2 筆待處理，數字正確  
- [ ] Schedule 時區是台北，不是 UTC  
- [ ] Active  

### C6. 進階（可選）
- 待處理 > 10 時主旨加「⚠︎ 堆積」  
- 週末不推：Schedule 加 cron 週一到五  
- 分業務 owner 各寄自己的待辦  

---

## 第 3 章結語｜你怎樣算「畢業」
同時滿足：
1. 藍圖 A **或** B 有一條在真實資料跑至少 3 天  
2. 出錯過一次，你知道去 n8n Executions 哪裡看  
3. 老闆／窗口知道 Sheet 誰在維護  

畢業後兩條路：  
- 自己複製 A/B 模式做第 2 條流程  
- 或請人做成長包（2–3 條串聯＋培訓＋30 天支援）把時間買回來  

— 第 1–3 章正文結束；第 4 章起（節點清單、報價話術、升級路徑）見完整啟動包。
