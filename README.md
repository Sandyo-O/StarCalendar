# 星曆 StarCalendar

結合傳統命理與 AI 解讀的多元占卜網站。輸入生辰或抽一副牌，系統會先用程式精確排出命盤，再交給 Google Gemini 以命理師的口吻給出解讀，解讀後還能繼續追問。

### 🔮 [立即使用 → tarot-rag.onrender.com](https://tarot-rag.onrender.com/)

不用安裝任何東西，點開連結就能直接占卜。

> 網站部署在 Render 免費方案，閒置一段時間後會進入休眠，第一次打開可能需要等 30 秒到 1 分鐘喚醒，之後就會恢復正常速度。

## 功能

| 項目 | 說明 |
| --- | --- |
| 🃏 塔羅占卜 | 22 張大阿爾克那，支援正逆位；提供單張牌（今日指引）、三張牌（過去・現在・未來）、凱爾特十字（完整深度解讀）三種牌陣 |
| 🀄 八字命理 | 依國曆生日與時辰排出四柱、五行、十神與大運 |
| 🌌 紫微斗數 | 使用 [iztro](https://github.com/SylarLong/iztro) 排出完整命盤 |
| ♈ 西洋星座 | 依出生日期解讀星座性格與運勢 |
| 💞 合盤分析 | 以八字與紫微比對兩人的契合度 |
| ✍️ 姓名學 | 依姓名進行解讀 |
| 🖐️ 面相／手相 | 上傳照片（或文字描述）進行趣味解讀 |
| 💬 延伸對話 | 解讀完成後可以繼續和占卜師聊天追問 |

### 命盤計算的細節

- **真太陽時校正**：選擇出生縣市後，會依台澎金馬各縣市經度（以東經 120 度為基準，每差 1 度約 4 分鐘）自動校正出生時辰。
- **時辰未知也能排**：未提供確切時辰時，時柱僅供參考，AI 會在解讀中提醒。
- **不讓 AI 自己編干支**：四柱、十神、大運等資料都由程式算好再送給模型，模型只負責解讀。
- **面相解讀的界線**：提示詞明確要求不做醫療診斷、不做智力或道德評價、不涉及種族刻板印象，僅作趣味詮釋。

## 技術架構

- **後端**：Node.js + [Express 5](https://expressjs.com/)
- **AI**：[Google Gemini](https://ai.google.dev/)（`@google/genai`，模型 `gemini-flash-lite-latest`）
- **命理計算**：[iztro](https://github.com/SylarLong/iztro)（紫微斗數）、[lunar-javascript](https://github.com/6tail/lunar-javascript)（農曆、八字）
- **檔案上傳**：multer（照片存在記憶體，上限 8 MB）
- **前端**：原生 HTML / CSS / JavaScript，無框架
- **部署**：[Render](https://render.com/)

## 專案結構

```
.
├── server.js           # Express 伺服器、API 路由、Gemini 呼叫
├── lib/
│   ├── bazi.js         # 八字排盤
│   ├── ziwei.js        # 紫微斗數排盤
│   └── calendar.js     # 曆法工具、時辰換算、真太陽時校正
└── public/
    ├── index.html      # 網站頁面
    ├── app.js          # 前端互動邏輯
    ├── style.css       # 樣式
    └── data/tarot.js   # 塔羅牌義與牌陣資料
```

## 本機執行

只是想用的話，直接打開[線上版](https://tarot-rag.onrender.com/)就可以了。以下是想自己架設或修改程式時的步驟。

需要 Node.js 18 以上版本，以及一組 [Gemini API 金鑰](https://aistudio.google.com/apikey)。

```bash
git clone https://github.com/Sandyo-O/StarCalendar.git
cd StarCalendar
npm install
```

在專案根目錄建立 `.env`：

```env
GEMINI_API_KEY=你的金鑰
# PORT=3000   # 選填，預設 3000
```

啟動：

```bash
npm start
```

打開 <http://localhost:3000> 即可使用。

## API

以下是網站後端提供給前端的接口。所有解讀與對話都由伺服器呼叫 **Google Gemini API**（模型：`gemini-flash-lite-latest`）產生，Gemini 金鑰只保存在伺服器端，不會傳到瀏覽器。

| 方法 | 路徑 | 說明 |
| --- | --- | --- |
| `POST` | `/api/reading` | 占卜解讀。以 `method` 指定類型：`tarot`、`bazi`、`ziwei`、`astro`、`compat`、`name` |
| `POST` | `/api/face` | 面相／手相解讀，以 `multipart/form-data` 上傳 `photo` |
| `POST` | `/api/chat` | 延伸對話，傳入 `history`（對話紀錄）與 `message` |

為了保護 API 額度，每個 IP 每小時最多 20 次請求。

## 免責聲明

本網站的解讀內容由 AI 生成，僅供娛樂與自我探索參考，不構成醫療、法律、財務或任何專業建議。
