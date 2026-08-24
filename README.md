# AI 短影片提示詞產生器

輸入一句話的中文畫面構想，優化成適合 OpenAI Sora、Google Veo、Runway、可靈 Kling 使用的短影片生成提示詞。

🔗 **線上使用**：<https://m255525.github.io/ai-video-prompt-studio/>

✅ **免費公開使用，無需授權序號。**

## 這是什麼

- **共用一組畫面與鏡頭欄位**：主題／主體動作、場景／地點、秒數、鏡頭運動、影片風格、光線氛圍、情緒氛圍、長寬比、音效／配樂提示（選填）、不希望出現的元素（選填），四個目標模型分頁共用，只需填一次。
- **四個目標模型分頁，格式真的不同**：
  - **Sora**：中文標籤條列＋建議整合成英文長句
  - **Veo**：依 Google 官方建議的 Subject／Context／Camera／Style／Ambiance 結構逐行陳述
  - **Runway**：逗號分隔英文關鍵詞組，鏡頭運動標籤置於最前方括號
  - **Kling**：中文為主的敘述句（可靈原生支援中文）
- **兩段式輸出**：「🔧 組成提示詞」是純前端字串組裝，不需金鑰、不連網；「🚀 送給 AI 優化」選用，串接你自己的 LLM API，把主題與場景翻譯、擴寫並依平台慣例調整格式。

## 功能

- **內建 5 組不同類型情境範例**：產品展示（香氛蠟燭特寫廣告）、自然風景空拍（山巒日出）、人物敘事（咖啡師手沖咖啡）、美食料理（熱炒鑊氣）、科幻／奇幻場景（賽博龐克飛行載具），一鍵套用快速上手
- **BYOK**：支援 Claude／OpenAI／Gemini／OpenRouter 四選一，API 金鑰只存在瀏覽器 localStorage，不經過任何後端伺服器
- **已儲存的提示詞**：可將組成的提示詞（連同 AI 優化結果）存成有名字的紀錄，之後載入、複製、下載 .txt 或刪除
- **無序號授權**：免費公開使用
- 響應式版面，桌機／平板／手機皆可使用；可加入主畫面（PWA）

## 怎麼用

1. 開啟 <https://m255525.github.io/ai-video-prompt-studio/>
2. 填「主題／主體動作」（必填）與「場景／地點」，視需要套用範例或選擇秒數／鏡頭運動／影片風格／光線氛圍／情緒氛圍／長寬比
3. 選一個目標模型分頁（Sora／Veo／Runway／Kling），按「🔧 組成提示詞」取得可複製的關鍵字草稿；或展開「API 連線設定」貼上你自己的金鑰，按「🚀 送給 AI 優化」取得正式提示詞
4. 滿意的結果可在「已儲存的提示詞」取名儲存

詳細操作說明見 [manual.html](https://m255525.github.io/ai-video-prompt-studio/manual.html)。

### API 金鑰申請網址

| 服務商 | 申請網址 |
|---|---|
| Claude（Anthropic） | <https://console.anthropic.com/> |
| OpenAI | <https://platform.openai.com/api-keys> |
| Gemini（Google AI Studio） | <https://aistudio.google.com/apikey> |
| OpenRouter | <https://openrouter.ai/keys> |

## 技術架構

純前端單檔工具，**沒有任何建置流程、框架、npm 依賴**：

| 項目 | 做法 |
|---|---|
| 提示詞組裝 | 純前端字串模板，不連網（主題／場景文字本身不翻譯） |
| AI 優化 | 瀏覽器直接 `fetch` 你選擇的 LLM 服務商官方 API（無後端代理） |
| 金鑰儲存 | `localStorage`，只在使用者自己的瀏覽器裡 |
| 頂部跑馬燈 | 與工作區其他工具共用同一個公告來源（可選、失敗不影響主功能） |

## 本機開發

不需要任何建置工具或安裝依賴，純靜態檔案：

```bash
git clone https://github.com/M255525/ai-video-prompt-studio.git
cd ai-video-prompt-studio
python -m http.server 8000
```

開啟 `http://localhost:8000`。

## 檔案結構

```
index.html                     主程式（跑馬燈 + 共用欄位 + 4目標模型分頁 + AI優化 + 儲存清單）
manual.html                    操作手冊
launcher.py                    可攜式桌面版啟動器（PyInstaller 打包用）
CLAUDE.md                      開發筆記／架構決策紀錄
```

## 隱私與資料

本 repo 公開的只有程式碼。你填寫的主題與鏡頭設定、組成的提示詞、AI 優化結果只存在自己瀏覽器的 localStorage；按下「送給 AI 優化」時，這些內容會直接連線送到你選擇的 AI 服務商，不經過本工具作者或任何第三方伺服器。

## 授權/用途

本工具免費公開使用，歡迎自由使用；請勿以本工具名義進行未經授權的冒用或不當宣稱。
