# kellylawyer.com.tw 全站掃描

目標：抓出**文章內容錯誤、頁面記載錯誤、連結放錯**。

---

## 現況：掃描未完成（環境連外被封鎖）

真正的全站掃描需要逐頁抓 HTML。本 session 的連外政策把**所有**外部主機擋掉了，不只本站：

```
curl https://kellylawyer.com.tw/  → CONNECT tunnel failed, response 403
curl https://example.com/         → HTTP 000（同樣被擋）
curl https://www.google.com/      → HTTP 000（同樣被擋）
WebFetch kellylawyer.com.tw       → EGRESS_BLOCKED
WebFetch example.com              → EGRESS_BLOCKED
```

proxy 自述（`curl "$HTTPS_PROXY/__agentproxy/status"`）：

```json
"recentRelayFailures": [
  { "kind": "connect_rejected",
    "detail": "gateway answered 403 to CONNECT (policy denial or upstream failure)",
    "host": "kellylawyer.com.tw:443" }
]
```

**解法**：把執行環境的 network policy 改成允許連外（或至少允許 `kellylawyer.com.tw`），
再開一個 session 重跑。設定方式見
<https://code.claude.com/docs/en/claude-code-on-the-web>。

目前 `pages.md` 的清單是用搜尋索引建的，可直接當掃描名單用。

---

## 已可確認的問題（不需抓頁面就看得到）

### 1. 〈離婚協議書怎麼寫〉title 標籤結尾多了一段 slug　【確定】

搜尋索引回傳的 `<title>` 是：

```
離婚協議書怎麼寫？律師詳解7大必填與注意事項 - 李珮瑄律師divorce-agreement
```

站名「李珮瑄律師」後面直接黏著 `divorce-agreement`，中間沒有空格或分隔符號。
其他頁面都是乾淨的「… - 李珮瑄律師」。應為 SEO plugin 的 title 欄位被誤填，
或自訂欄位殘留。兩次不同的搜尋都回傳同一字串，不是索引雜訊。

**修正**：編輯該文的 SEO title，刪掉尾巴的 `divorce-agreement`。

### 2. 四篇文章的 slug 結尾是 `-2`　【需查證】

```
infringement-of-spousal-rights-taiwan-lawyer-2   外遇蒐證怎麼做才合法？
divorce-litigation-precautions-taiwan-lawyer-2   離婚訴訟要注意什麼？
divorce-process-timeline-taiwan-lawyer-2         離婚程序要多久？
separation-divorce-lawsuit-taiwan-2              分居多久可以離婚？
```

WordPress 只有在「同名 slug 已被佔用」時才會自動加 `-2`。代表這四篇各自
**曾經有一篇同名文章**。要查：

- 原文還在線上嗎？→ 兩篇內容雷同＝重複內容，會互相稀釋排名，要擇一 301。
- 原文已被丟到垃圾桶／草稿？→ slug 被佔著沒釋放，且站內舊連結可能還指向原文（404）。

**查法**：後台文章列表搜尋標題關鍵字，看有無同名的草稿/垃圾桶文章；
並確認去掉 `-2` 的網址（例：`/divorce-process-timeline-taiwan-lawyer/`）回傳什麼狀態碼。

### 3. slug 命名規則不一致　【體質問題】

站上同時存在英文 slug 與中文 slug（中文 slug 在網址列會變成一長串 `%E9%9B%A2...`）。
其中〈監護權律師費用怎麼算？律師解析3大收費模式與行〉的 slug 結尾是「與行」，
像是「與行情」被截斷。中文 slug 不算錯誤，但分享連結醜、易被截斷、不利追蹤，
新文章建議統一用英文 slug（既有文章要改必須做 301，否則會斷連結）。

---

## 恢復連外後要跑的掃描項目

- [ ] 以 `sitemap.xml` 重建完整頁面清單，跟 `pages.md` 對照補齊漏網頁面
- [ ] **連結**：全站內部連結逐一發 request，抓 404 / 301 鏈 / 指向 `http://` 的舊連結 /
      指向 localhost 或測試站的連結；外部連結抓死連（法條、判決書連結特別容易失效）
- [ ] **頁面記載**：電話、地址、Email、營業時間、律師證號在每一頁是否一致
      （目前搜尋索引顯示：桃園市桃園區大興西路二段6號12樓之3、03-3024567、kelly49871620@gmail.com）
- [ ] **收費記載一致性**：`/services/`、`/inheritance-service/`、各收費文章的金額是否互相矛盾
      （已知索引顯示遺囑諮詢 6,000/時、拋棄繼承 20,000 起、繼承訴訟 80,000 起/審級；
      監護權諮詢 3,000–8,000、書狀 15,000–40,000、一審 80,000–200,000——需確認頁面間沒有打架）
- [ ] **文章內容**：法條條號與現行法比對、判決字號查證（如〈認諾是什麼〉的「士林地院115訴377」）、
      金額／期間／年份數字、錯字漏字
- [ ] **重複內容**：`-2` 系列與原文比對，決定合併或 301
- [ ] **title / meta description**：長度、重複、像 #1 那樣的殘留字串
- [ ] **圖片**：破圖、alt 缺漏
- [ ] **表單**：聯絡表單是否真的送得出去、收件信箱是否正確
