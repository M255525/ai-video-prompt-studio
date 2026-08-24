# CLAUDE.md — ai-video-prompt-studio（AI 短影片提示詞產生器）

「AI 短影片提示詞產生器」——單檔前端工具，把一句話的中文畫面構想，優化成適合 AI 短影片生成模型使用的提示詞：OpenAI Sora、Google Veo、Runway、可靈 Kling。填一次主題／場景／秒數與鏡頭／風格設定，四個目標模型分頁共用；可以純前端組成關鍵字草稿，也可以串接使用者自己的語言模型取得中英對照的正式提示詞。

與 `行銷內容工具/ai-image-prompt-studio/`（圖片生成提示詞優化）是姊妹專案，同一套「共用欄位＋多目標模型分頁只改組裝格式」的架構與同一套 BYOK 呼叫 LLM 的手法，服務對象換成影片生成提示詞。**與 `ai-image-prompt-studio` 內建的「🎬 五秒影片腳本參考」不衝突**——那是「圖轉動態片段」的附加建議小功能（以已產生的圖片提示詞為輸入），本專案是獨立的、以文字生成完整影片提示詞為核心目的的工具（更完整的秒數／鏡頭／場景欄位、四個真正的 text-to-video 模型分頁）。也與 `ai-prompt-generator` 的「影音框架」（服務對象是 Pexels 素材腳本，貼給 `ai-video-studio` 的傳統合成管線用）不衝突，欄位設計與目標受眾完全不同。

## 架構

單一 `index.html`：內嵌 CSS/JS、無外部資源、無建置步驟。視覺主題是深色「片場／膠捲」風格（`--bg #150e08` + 圓點網格背景 + 橙色 `--accent #f97316`），與姊妹專案（`ai-image-prompt-studio` 洋紅 `#ec4899`、`ai-prompt-generator` 天藍 `#38bdf8`、`product-title-generator` 藍 `#3b82f6`、`ai-music-prompt-studio` 紫 `#a855f7`）刻意做出色彩區隔，方便一眼分辨是哪個工具。

- **共用欄位 + 多目標模型分頁**（比照 `ai-image-prompt-studio` 的結構）：單一組共用欄位 `state.fields`（主題／場景／秒數／鏡頭運動／影片風格／光線氛圍／情緒氛圍／長寬比／音效配樂提示／不希望出現的元素／既有腳本補充）搭配四個只改變「組裝格式」的目標模型分頁（`TARGETS`：`sora`/`veo`/`runway`/`kling`）。使用者填一次欄位，四個分頁共用，只有「組成提示詞」「送給 AI 優化」的輸出格式規則依分頁不同。
- 鏡頭運動／影片風格／光線氛圍／情緒氛圍的選項陣列（`CAMERA_OPTIONS`／`STYLE_OPTIONS`／`LIGHTING_OPTIONS`／`MOOD_OPTIONS`）每個選項都同時定義中文標籤（`zh`）與英文提示詞片語（`en`）；長寬比（`RATIO_OPTIONS`）與秒數（`DURATION_OPTIONS`）只有 `zh`（不需要英文組裝片語，秒數直接用數字）。新增選項時注意補齊 `zh`/`en`。
- **`assembleForTarget()` 是四個分頁真正各自不同的組裝模板**（這是與 `ai-image-prompt-studio` 最大的差異點——image-studio 只有 Midjourney 分頁的參數語法特別，其餘三分頁格式雷同；本專案四個分頁刻意各自代表一種常見的影片提示詞慣例）：
  - `sora`：中文標籤條列（【主題】【場景】【時長】…），附警語建議整合成英文長句
  - `veo`：依 Google 官方建議的 **Subject／Context／Camera／Style／Ambiance** 結構逐行陳述，每行附中文標籤＋英文片語
  - `runway`：逗號分隔關鍵詞組，**鏡頭運動標籤放在最前面並以方括號包住**（`[camera en], topic, scene, style...`），結尾附 `duration:`/`aspect ratio:` 關鍵字
  - `kling`：**中文為主的敘述句**（可靈原生支援中文），鏡頭與氛圍詞彙置於句尾
  - `aiPromptForTarget()` 的 `formatRule` 字串依分頁對應調整；`kling` 分頁的輸出規則比其他三個多一段——要求「中文說明＋可靈中文提示詞＋English Prompt (optional)」三段式（其餘三分頁是「中文說明＋English Prompt」兩段式），因為可靈的主要輸入語言就是中文，不像 Sora/Veo/Runway 最終要用英文。新增/修改任一分頁的格式時，`assembleForTarget()`（純組裝）與 `aiPromptForTarget()`（AI優化的格式規則）是兩個獨立函式，要一起檢查是否仍互相呼應。
- **內建 5 組情境範例**（`PRESETS`，完全虛構），刻意涵蓋 5 種不同類型主題以展示欄位彈性：質感香氛蠟燭產品特寫廣告（產品展示）、山巒日出空拍紀實（自然風景空拍）、街頭咖啡師沖煮咖啡（人物敘事）、熱炒店鑊氣爆炒鏡頭（美食料理）、賽博龐克城市夜景飛行載具穿梭（科幻／奇幻場景）。新增範例時比照這個「涵蓋不同類型」的精神，不要都塞同一種主題。
- 狀態存 `localStorage`（key: `vidPromptState`）：`{activeTarget, fields:{topic,scene,duration,cameraMovement,style,lighting,mood,ratio,audioNote,negative,existingPrompt}, assembled:{sora,veo,runway,kling}, aiOutput:{...}}`——欄位是單一物件，組成結果與 AI 結果按分頁各自保留，切換分頁不會互相覆蓋。
- **核心互動模型**（與 `ai-image-prompt-studio` 相同精神）：「🔧 組成提示詞」純前端字串組裝（不需金鑰、不連網）——主題／場景文字**保留原始語言、不會被翻譯**，只有鏡頭／風格／光線／情緒標籤是已對照好的英文/中文片語；「🚀 送給 AI 優化」把畫面與鏡頭設定送給 BYOK LLM，這一步才會真的把主題翻譯成英文（或針對 Kling 產出精緻中文＋附英文）。
- **BYOK AI 串接**：與 `ai-image-prompt-studio`／`ai-prompt-generator`／`Prompt` 同一套 `callLLM()` 模式（改動時互相參照）——瀏覽器直連 `fetch()`：Claude 需 `anthropic-dangerous-direct-browser-access: true` header；Gemini 金鑰放 `x-goog-api-key` header；OpenAI/OpenRouter 用 Bearer。設定（provider/model/apiKey）存 `localStorage`（key: `vidPromptApiConfig`）——**金鑰只落在使用者本機瀏覽器，絕不可寫進程式碼**。逾時 180 秒；429/500/503/529 自動重試最多 2 次（間隔 8、16 秒）。
- **已儲存的提示詞**：`localStorage`（key: `vidPromptSavedItems`），每筆 `{id, target, name, savedAt, fieldValues, assembledPrompt, aiOutput}`。載入時會同時還原共用欄位＋切換回對應的目標模型分頁。
- `manual.html` 操作手冊：四目標模型分頁介紹／操作步驟／「組成」與「AI優化」差異說明／已儲存的提示詞／AI 串接說明／隱私說明／使用警語／創作者資料／授權限制。**創作者經歷內容與 `ai-image-prompt-studio/manual.html`、`icap-generator/manual.html`、`sbir-generator/manual.html`、`phoenix-loan-generator/manual.html`、`Prompt/manual.html`、`ai-prompt-generator/manual.html` 為同一份，更新其中一邊時同步其餘各邊。**

## 不套用序號授權（與姊妹專案的差異點）

**這是與 `ai-image-prompt-studio`／`ai-prompt-generator`／`product-title-generator`／`ai-music-prompt-studio` 最大的差異**：使用者明確要求本工具**不套用序號授權**，直接公開使用（比照 `coffee-ig-planner` 的模式），沒有 `#licenseGate`、沒有 `Code.gs`、沒有綁定 Google Sheet 驗證後端。頁面裡完全沒有序號相關的 HTML/CSS/JS。日後若使用者要求補上序號授權，可直接參照 `ai-image-prompt-studio/index.html` 的 `#licenseGate` 區塊＋對應 IIFE 搬過來，`Code.gs`／`SETUP-授權伺服器設定.md` 也可整份複製調整文字。

## 頂部共用跑馬燈

`#marqueeBar` 內容抓自工作區既有的共用授權伺服器（`https://script.google.com/macros/s/AKfycbwKX0.../exec`，與 `Prompt`／`ai-prompt-generator`／`ai-image-prompt-studio` 等共用同一個 Google Sheet），做法完全比照姊妹專案——跟「本工具沒有序號授權」無關，跑馬燈是完全獨立的系統。`localStorage` key `vidPromptMarquee`，每 20 分鐘背景重抓一次。改跑馬燈內容直接編輯共用 Sheet 即可，不需要部署任何 Apps Script。

## PWA 加入主畫面

比照姊妹專案：`manifest.json`＋`icons/`（橙色 `#f97316` 背景、播放三角形圖示，用 Pillow 產生，見下方指令）＋`service-worker.js`（network-first＋同源快取備援）。頁尾「📲 加入主畫面」按鈕獨立 IIFE，含 iOS／macOS Safari 的分享選單／Dock 提示文案，`notify()` 不依賴跨 `<script>` 區塊的 `showToast()`（見 `ai-image-prompt-studio/CLAUDE.md` 的踩坑記錄，這裡從一開始就用獨立實作避開該 bug class）。

重新產生圖示：
```bash
python -c "
from PIL import Image, ImageDraw
BG = (249, 115, 22, 255); FG = (21, 14, 8, 255)
def make_icon(size, path, maskable=False):
    img = Image.new('RGBA', (size, size), BG)
    d = ImageDraw.Draw(img)
    pad = size * (0.30 if maskable else 0.22)
    h = size - pad*2
    cx, cy = size/2, size/2
    tri = [(cx - h*0.32, cy - h*0.42), (cx - h*0.32, cy + h*0.42), (cx + h*0.48, cy)]
    d.polygon(tri, fill=FG)
    img.save(path)
make_icon(192, 'icons/icon-192.png')
make_icon(512, 'icons/icon-512.png')
make_icon(512, 'icons/icon-maskable-512.png', maskable=True)
make_icon(180, 'icons/apple-touch-icon.png')
"
```

## 隱私與警語

無伺服器端經手使用者資料；欄位內容、組成結果、AI 優化結果、已儲存清單皆只存在使用者瀏覽器的 localStorage。首頁與手冊皆明列使用警語：AI 優化結果需自行查核、請勿輸入真實個資或機密資料、各平台秒數／格式僅為參考慣例（以當下官方文件為準）。修改功能時這些警語需一併檢視是否仍準確。

## 桌面版 exe（VideoPromptStudio/）

`launcher.py` 把 index/manual 打包進 exe，執行時於 `127.0.0.1:8798` 起本機伺服器並開預設瀏覽器（**固定 8798 埠**——工作區埠號分配見根目錄 `CLAUDE.md`，8798 為建置時確認未使用的最低空號，8765–8797 皆已被工作區其他專案佔用）。**無序號檢查**（與 `ai-image-prompt-studio/launcher.py` 的差異）。修改 index.html／manual.html 後 exe 不會自動更新，需重建：

```powershell
$proj = "C:\Users\mark_\AI Test\行銷內容工具\ai-video-prompt-studio"
cd $proj
python -m PyInstaller --onefile --console --name VideoPromptStudio `
  --distpath "$proj\VideoPromptStudio" --workpath "$env:TEMP\pyi-build-videoprompt" --specpath "$env:TEMP" `
  --add-data "$proj\index.html;." --add-data "$proj\manual.html;." `
  launcher.py
```

尚未建置（本次規劃/開發階段未執行 PyInstaller 打包，僅備妥 `launcher.py`）。exe 未簽章，首次執行可能遇 SmartScreen 或 Windows Smart App Control 警告（見全域記憶 `windows-smart-app-control-dll-blocks`）；測試 exe 時注意 PyInstaller onefile 會有父子兩個程序，需要 `taskkill //IM VideoPromptStudio.exe //F` 才殺得乾淨。

## 響應式設計

版型沿用 `ai-image-prompt-studio` 已驗證過的流體設計（`main`/`.hero p.lead` 皆為 `max-width`）：900px 以下（平板）`.field-grid`/`.api-grid` 維持雙欄；600px 以下（手機）收成單欄＋放大觸控熱區（`.btn` 至少 44px）。已用 Playwright 在 375×800 視窗實測 `document.documentElement.scrollWidth` 未超出 `window.innerWidth`，無橫向捲動。

## 已驗證項目（本次開發階段）

- 用 `python -m http.server 8798` 起本機伺服器，Playwright 實測：套用範例（賽博龐克城市場景）正確帶入所有欄位；切換 4 個分頁分別按「組成提示詞」，確認四種格式（Sora 中文標籤條列／Veo 官方 Subject-Context-Camera-Style-Ambiance 結構／Runway 方括號鏡頭前綴＋逗號關鍵詞／Kling 中文敘述句）皆如預期產出、彼此明顯不同；重新整理頁面後欄位與分頁狀態正確從 localStorage 還原；「儲存」寫入已儲存清單成功（測試後已 `localStorage.clear()` 清除測試資料）；375px 手機寬度下無橫向捲動。
- **本次未做**：「送給 AI 優化」的實際 LLM 呼叫（需要真實 API 金鑰，未測試）；exe 尚未用 PyInstaller 建置與雙擊驗證；尚未 commit／建立公開 GitHub repo／部署 GitHub Pages（需使用者另外確認，屬於對外可見動作）；根目錄 `CLAUDE.md` 專案清單與 `專案目錄.docx` 尚待補列。

## 指令

無建置/測試指令。修改 `index.html` 或 `manual.html` 後直接用瀏覽器開啟驗證，或暫起 `python -m http.server <port>` 測完關閉。修改內嵌 `<script>` 後可用以下方式快速檢查語法：

```bash
python -c "
import re
html = open('index.html', encoding='utf-8').read()
open('_check.js','w',encoding='utf-8').write(re.findall(r'<script>(.*?)</script>', html, re.S)[0])
"
node --check _check.js
```
