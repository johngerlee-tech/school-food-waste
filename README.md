# 🌱 溪洲國小午餐智慧惜食與廚餘追蹤系統 (V4.2 - stable05)

![GitHub Pages](https://img.shields.io/badge/Deployment-GitHub%20Pages-brightgreen)
![Firebase](https://img.shields.io/badge/Backend-Firebase%20v10.12.2-orange)
![Google Gemini](https://img.shields.io/badge/AI%20Engine-Gemini%203.8%20Flash-blue)
![Tailwind CSS](https://img.shields.io/badge/Styling-Tailwind%20CSS-38B2AC)

一套為國小校園現場量身打造的**「智慧惜食量化追蹤、AI 菜單影像辨識、食材適口性深度交叉診斷與全校榮譽榜評比」**決策平台。

本專案採現代化 **單一檔案 (Single-File SPA)** 架構設計，免除任何 Node.js 建置與編譯步驟，完美結合 **GitHub Pages** 靜態代管、**Google Firebase (Auth & Firestore)** 雲端資料庫與 **Google Gemini AI 多模態模型**。

---

## 📌 目錄
- [系統核心特點](#-系統核心特點)
- [四大身分權限矩陣](#-四大身分權限矩陣)
- [六大功能模組介紹](#-六大功能模組介紹)
- [技術堆疊與底層架構](#-技術堆疊與底層架構)
- [Cloud Firestore 資料庫結構 (Schema)](#-cloud-firestore-資料庫結構-schema)
- [快速部署指南 (GitHub Pages + Firebase)](#-快速部署指南-github-pages--firebase)
- [安全規則設定 (Security Rules)](#-安全規則設定-security-rules)
- [維護基準與版本記錄](#-維護基準與版本記錄)

---

## 🌟 系統核心特點

1. **以公克 (g/人) 精準量化**：摒棄傳統「倒幾桶」的粗略估計，即時按各班用餐人數計算人均廚餘公克數。
2. **免個人帳號之「🍱 廚餘長快速過磅通道」**：各班值日生或廚餘長憑後台設定之專用通行碼即可就地驗證，隨到、隨秤、隨班即時存檔。
3. **AI 多模態菜單辨識**：管理員上傳午餐菜單圖檔（JPG/PNG），自動調用 `gemini-3.8-flash` 辨識各日五道菜色（主食、主菜、副菜、蔬菜、湯品）並填入系統。
4. **雙引擎菜單關聯深度診斷**：
   * **雲端 Gemini 營養師診斷**：針對全校廚餘超標日，由 AI 交叉比對食材特性（骨刺佔比、排斥性蔬菜、粗糧口感）與年段接受度，提供午餐廚房 2~3 點具體改善建議。
   * **在地啟發式專家規則引擎保底**：斷網或無 API Key 時自動無縫切換，保證診斷功能不中斷。
5. **Cache-First 快取架構**：診斷紀錄自動持久化存入 Firestore，優先載入歷史存檔，極速開機且節省 API 呼叫額度。
6. **客觀事由多人討論串**：超標日提供獨立討論串，供管理員與工作小組補述當日現場狀況（如運動會、校外教學、盛菜調控等）。
7. **彈性秤重範圍配置**：可於後台自訂常態秤重星期（週一至週日獨立勾選）與適用班級（預設排除一年級），非適用項目自動防呆反白鎖定且不計入統計分母。

---

## 🔐 四大身分權限矩陣

| 功能 / 分頁模組 | 訪客模式 (Visitor) | 🍱 廚餘長模式 (Monitor) | 👥 工作小組 (Staff) | 🛡️ 系統管理員 (Admin) |
| :--- | :---: | :---: | :---: | :---: |
| **身分驗證方式** | 免登入 | 通行碼驗證（免帳號） | Google / Email 帳密登入 | Google / Email 帳密登入 |
| **1. 每日登記** | 唯讀檢視 | 逐班輸入、修改與儲存 | 依後台勾選決定是否開放 | 全權輸入、修改與儲存 |
| **2. 週評比與進步獎** | 唯讀 / 廣播詞複製 | 唯讀 / 廣播詞複製 | 唯讀 / 廣播詞複製 | 唯讀 / 廣播詞複製 |
| **3. 每月榮譽榜** | 唯讀檢視 | 唯讀檢視 | 唯讀檢視 | 唯讀檢視 |
| **4. 菜單查詢 / 維護** | 僅查詢公告菜單 | 僅查詢公告菜單 | 僅查詢公告菜單 | AI 圖檔辨識、手動編修與儲存 |
| **5. 菜單關聯分析** | 🔒 權限攔截 | 🔒 權限攔截 | 完整檢視、啟動診斷、留言 | 完整檢視、強制重跑、留言管理 |
| **6. 系統與匯入設定** | 🔒 權限攔截 | 🔒 權限攔截 | 🔒 權限攔截 | 全權配置與成員授權 |

---

## 💻 六大功能模組介紹

### 1. 每日登記 (Daily Entry)
* **自動週次與國曆對照**：依開學日自動計算當前學年與週次；秤重星期標籤自動精確對應國曆日期（如 `週一 (09/14)`，以紫底粗體醒目呈現）。
* **全校／班級停餐免秤重**：支援特定日期全校停餐一鍵遮罩；支援個別班級獨立勾選停餐。
* **逐班獨立儲存防呆**：各班資料列最末端設置獨立操作按鈕：
  * 未儲存：綠色 `💾 儲存`，輸入框保持可編輯。
  * 已儲存：整列自動轉淺灰，輸入框鎖定，按鈕切換為灰色 `✏️ 修改`。
* **非秤重日防呆**：非開放秤重星期顯示淡琥珀色警告橫幅，全頁輸入框自動進入唯讀保護。

### 2. 週評比與進步獎 (Weekly Awards)
* **加權人均計算**：排除免秤班級與停餐日，以當週累計總重與用餐總人次計算真實加權人均（g/人）：
  $$\text{週加權人均 (g/人)} = \frac{\sum \text{各日廚餘重量 (g)}}{\sum \text{各日用餐總人次}}$$
* **各年段週冠軍**：低年級（1~2年級）、中年級（3~4年級）、高年級（5~6年級）自動分組評選冠軍。
* **週進步獎判定**：自動比對前一週數據，減幅 $\ge 5.0\%$ 即獲頒進步獎標籤：
  $$\text{減幅 (\%)} = \frac{\text{前週人均} - \text{本週人均}}{\text{前週人均}} \times 100\%$$
* **一鍵產生廣播稿**：自動生成適合升旗朝會或午餐廣播之文案，支援一鍵複製。

### 3. 每月榮譽榜 (Monthly Honor)
* 支援 9 ~ 12 月動態切換（精確對應：9月第1~4週、10月第5~8週、11月第9~12週、12月第13~17週）。
* 彙整全月累計總重與總人次，計算全月加權人均，頒發各年段「月惜食總冠軍」👑。

### 4. 菜單查詢與 AI 維護 (Menu Management)
* 公告週一、週二、週四、週五菜色明細。
* **AI 圖檔辨識**：管理員上傳營養午餐菜單圖檔（JPG/PNG），自動透過 Gemini 多模態 API 解析為標準 JSON 格式並回填輸入框。

### 5. 菜單關聯異常分析 (AI Diagnosis)
* **動態警示比對**：當日全校人均超過動態門檻（預設 120 g/人）時觸發紅框警示。
* **快取優先 (Cache-First)**：已診斷結果持久化存於 Firestore `menu_diagnoses`，再次檢視秒開且零浪費 API 配額。
* **客觀事由討論串**：
  * 僅於「有數據且超標」之卡片下方顯示。
  * 支援多人依序留言（姓名置於時間前，格式如：`👤 姓名 2026/09/14 12:30`）。
  * 管理員具全權維護權限；工作小組可就地修改/刪除本人留言，修改後自動加註 `(已編輯)`。

### 6. 系統與匯入設定 (System & Settings)
* **集中託管 Gemini API Key**：金鑰加密儲存於雲端資料庫，同仁無須於個人端輸入，附「🧪 測試連線」驗證機制。
* **成員名冊管理**：管理員可自訂管理員信箱與工作小組名冊（含姓名、Email 與獨立勾選「允許每日登記」）。
* **廚餘長通行碼設定**：後台自訂現場過磅密碼，具備👁️顯示切換。
* **彈性秤重範圍配置**：可自訂哪些星期（1~7）與班級（101~602）納入常態秤重。
* **學期行事曆與門檻**：設定開學日（自動推算週次）、結業日與自訂警示門檻（g/人）。
* **歷史 CSV 批次匯入**與班級預設人數管理。

---

## 🛠️ 技術堆疊與底層架構

* **前端介面**：HTML5 + Vanilla JavaScript (ES Module) + Tailwind CSS (CDN)。
* **後端服務**：Google Firebase v10.12.2 (Authentication + Cloud Firestore)。
* **AI 診斷核心**：Google Generative Language API (`gemini-3.8-flash`)。
* **架構亮點**：
  * **Zero-Failure DOM**：頁面啟動 0.01 秒內以記憶體靜態陣列同步預渲染 12 班骨架，無任何網路依賴，杜絕白畫面。
  * **沙盒化生命週期**：透過 `isSystemInitializing` 旗標嚴格阻絕開機連鎖 `onchange` 事件，徹底防範腳本死鎖崩潰。
  * **全域函式優先提升 (Hoisting)**：關鍵跳頁與彈窗函式掛載於 `window` 最頂層，全面套用 Try-Catch 容錯。

---

## 🗄️ Cloud Firestore 資料庫結構 (Schema)

```text
school-food-waste (Firestore Database)
├── daily_records (集合：各班每日秤重明細)
│   └── {academicYear}_{semester}_{week}_{weekday}_{classId}
│       ├── academicYear: 115
│       ├── semester: 1
│       ├── week: 2
│       ├── weekday: 1 (1~6, 0)
│       ├── classId: "201"
│       ├── className: "二年甲班"
│       ├── stage: "低年級"
│       ├── weightKg: 1250 (公克)
│       ├── eaterCount: 24 (用餐人數)
│       ├── perCapitaGram: 52.1
│       ├── isSkipped: false
│       └── updatedAt: Timestamp
│
├── daily_status (集合：每日全校停餐狀態)
│   └── {academicYear}_{semester}_{week}_{weekday}
│       ├── isSkipped: boolean
│       └── updatedAt: Timestamp
│
├── weekly_menus (集合：每週公告菜單)
│   └── {academicYear}_{semester}_{week}
│       ├── academicYear: 115
│       ├── week: 2
│       └── days: { "1": { main, dish1, dish2, vege, soup }, "2": { ... } }
│
├── menu_diagnoses (集合：AI 診斷與討論串快取)
│   └── {academicYear}_{semester}_{week}
│       ├── academicYear: 115
│       ├── week: 2
│       ├── days: {
│       │     "1": {
│       │       hasRecordedData: true,
│       │       isSpike: true,
│       │       per: 135.2,
│       │       aiDeduction: "...",
│       │       notes: [
│       │         { id, text, author, authorEmail, time, isEdited }
│       │       ]
│       │     }
│       │   }
│       └── savedAtFormatted: "2026/09/14 13:00"
│
├── system_roles (集合：權限名單)
│   └── main
│       ├── admins: ["admin1@gmail.com", "admin2@gmail.com"]
│       └── staffs: [{ name: "王老師", email: "teacher@school.edu.tw", canDaily: true }]
│
├── system_config (集合：系統組態)
│   ├── main (year, semester, startDate, endDate, threshold)
│   ├── gemini_api (apiKey)
│   ├── waste_monitor (password)
│   └── weighing_rules (activeWeekdays: [1,2,4,5], activeClassIds: ["201", ...])
│
└── class_settings (集合：各班預設人數)
    └── {classId} -> { classId: "201", defaultCount: 24 }
```

---

## 🚀 快速部署指南 (GitHub Pages + Firebase)

### 步驟 1：取得程式碼並建立 GitHub 儲存庫
1. 在 GitHub 建立一個全新的公開或私有儲存庫（如：`school-food-waste`）。
2. 將本專案的 `index.html` 推送至 `main` 分支根目錄。

### 步驟 2：設定 Firebase 專案
1. 進入 [Firebase Console](https://console.firebase.google.com/) 建立專案。
2. 啟用 **Authentication**：
   * 啟用 **Google** 登入。
   * 啟用 **電子郵件/密碼** 登入。
3. 建立 **Cloud Firestore** 資料庫（建議選擇 `asia-east1 (Taiwan)` 或預設地區）。
4. 於專案設定中取得 Web App 的 `firebaseConfig` 物件，並置換 `index.html` 中的組態金鑰。

### 步驟 3：啟用 GitHub Pages 免費託管
1. 進入 GitHub 儲存庫的 **Settings** ➔ **Pages**。
2. 在 **Branch** 選擇 `main` 分支與 `/ (root)` 資料夾後點擊 **Save**。
3. 等待約 1~2 分鐘，即可取得全校公用網址：
   `https://<您的帳號>.github.io/school-food-waste/`

---

## 🛡️ 安全規則設定 (Security Rules)

為確保訪客可公開查閱報表，同時讓廚餘長通行碼模式與管理員順暢寫入，請前往 **Firebase Console ➔ Firestore Database ➔ 規則 (Rules)** 發布以下規則：

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 每日過磅數據與停餐狀態
    match /daily_records/{recordId} {
      allow read: if true;
      allow write: if true;
    }
    match /daily_status/{statusId} {
      allow read: if true;
      allow write: if true;
    }

    // 菜單與診斷快取
    match /weekly_menus/{menuId} {
      allow read: if true;
      allow write: if true;
    }
    match /menu_diagnoses/{diagId} {
      allow read: if true;
      allow write: if true;
    }

    // 系統配置、成員角色與預設人數
    match /system_config/{configId} {
      allow read: if true;
      allow write: if true;
    }
    match /system_roles/{roleId} {
      allow read: if true;
      allow write: if true;
    }
    match /class_settings/{classId} {
      allow read: if true;
      allow write: if true;
    }
  }
}
```

---

## 📌 維護基準與版本記錄

* **基準版本代號**：`stable05`
* **版本維護守則**：
  * 本儲存庫後續之擴充與維護，一律以此版本之程式架構、DOM 節點命名與 Firestore 集合脈絡為準。
  * 涉及已正常運作之核心區塊（AI 診斷、開機沙盒、權限體系）若需異動，須先進行影響範圍評估。