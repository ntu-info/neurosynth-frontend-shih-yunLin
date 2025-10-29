# Neurosynth 即時術語查詢前端

本專案提供一個針對 Tren 老師後端 (`https://mil.psy.ntu.edu.tw:5000`) 的靜態前端。使用者在「搜尋 Neurosynth 術語」欄位輸入關鍵字即可同步獲得以下資訊：

- 所有可用術語的即時篩選結果
- 選定術語的共現詞（AND / OR / NOT / 括號工具可直接插入布林查詢）
- 對應的研究列表（支援年份排序、Load more 分頁、布林關鍵字高亮）

整體以 Tailwind CSS 建構，並將原本 inline JS 拆分至 `public/assets/app.js`，方便維護與部署。

---

## 功能亮點

- **即時術語搜尋**：不需送出表單，即時以關鍵字過濾 Tren 後端 `/terms` 回傳的 3,000+ 術語。
- **布林查詢工具列**：點擊共現詞再搭配 AND / OR / NOT / `(`/`)` 按鈕即可插入查詢語句，搜尋結果即時更新。
- **多主機備援**：預設連線 `https://hpc.psy.ntu.edu.tw:5000`，若逾時會自動改用 `https://mil.psy.ntu.edu.tw:5000`，並在快取命中時顯示。
- **快取與高亮**：布林查詢結果會快取，命中後 150ms 內顯示，且在研究卡片標題、期刊、作者欄位標示出所有關鍵字。
- **靜態部署友善**：Tailwind 透過 `npm run build` 產生 `public/assets/app.css`，整個 `public/` 目錄可直接部署至任何靜態網站服務。

---

## 環境需求

- [Node.js](https://nodejs.org/) 18 或以上
- 建議使用 `npm` 9 以上（專案已提供 `package-lock.json`）

---

## 安裝與開發

```bash
# 安裝依賴
npm install

# 開發模式：監聽 Tailwind，輸出至 public/assets/app.css
npm run dev
```

開發時建議搭配簡易靜態伺服器（例如 `npx serve public` 或 VS Code Live Server）開啟 `public/index.html` 以避免 `file://` 無法呼叫 API 的問題。

---

## 建置（Production Build）

```bash
npm run build
```

指令會執行：

```
tailwindcss -i ./src/styles.css -o ./public/assets/app.css --minify
```

完成後 `public/` 內的 `index.html` + `assets/` 即為最終靜態檔案，可直接部署。

---

## 部署建議

### GitHub Pages

1. 將專案推送到 GitHub repository。
2. 在 `Settings → Pages` 中設定來源：
   - 若選擇 `Deploy from a branch`，可將 `public/` 內容複製到 `docs/` 資料夾，再在 Pages 設 `Branch: main / Folder: /docs`。
   - 或改用 GitHub Actions，自動執行 `npm run build` 並發布 `public/`。
3. 儲存後稍待片刻，即可取得公開網址 `https://<username>.github.io/<repo>/`。

### 其他靜態主機

- **Netlify / Vercel / Cloudflare Pages**：設定 build 指令 `npm run build`，deploy directory 指向 `public`。
- **自架 Nginx / Apache**：直接將 `public` 內容上傳至 DocumentRoot，確保 `app.css`、`app.js` 路徑正確。

---

## 專案結構

```
public/
  index.html          # 編譯後的主頁
  assets/
    app.css           # Tailwind 編譯結果
    app.js            # 主要互動邏輯（Fetch / Cache / UI Render）
src/
  styles.css          # Tailwind 指令與自訂 layer
tailwind.config.js    # Tailwind 組態（色票、fonts、content path）
postcss.config.js     # PostCSS + Autoprefixer
gpt-5-codex.pdf       # 與 LLM (GPT-5 Codex) 的完整對話紀錄，作為開發過程備份
```

---

## 主要互動流程

1. **載入術語**：啟動時呼叫 `/terms`，結果快取於記憶體；輸入框會帶出最多 24 筆符合的術語。
2. **選取術語**：點擊術語或直接輸入後，會更新：
   - 相關術語：呼叫 `/terms/<term>`，顯示共現詞與 `co / J` 指標。
   - 布林查詢：自動填入術語並執行 `/query/<term>/studies`，可再搭配 AND/OR/NOT/括號。
3. **布林查詢狀態**：若使用者手動編輯查詢欄位，系統會記錄 `queryDirty=true`，除非使用者清除或再次點選術語，避免自動覆寫正在編輯的條件。
4. **高亮與分批顯示**：研究結果初始顯示 25 筆，如有更多可按「顯示更多」。所有匹配的關鍵字會以 `<mark>` 突顯。

---

## FAQ

- **Q：為什麼我輸入 `AND` 後沒有高亮？**  
  A：高亮僅針對查詢詞內的關鍵字（排除 AND/OR/NOT/括號），如果某關鍵字未出現在標題、期刊或作者欄位，仍可能存在於 API 回傳的其他欄位（例如摘要）。

- **Q：如何切換後端主機？**  
  A：`public/assets/app.js` 內 `API_BASES` 陣列可自由調整優先順序或加入更多 mirror。

- **Q：佈署後出現 CORS 錯誤？**  
  A：請確認佈署的網域是否允許存取 Tren 老師的 API；若伺服器限制較嚴，需透過後端代理或老師提供的 whitelist 處理。

---

## 授權

此專案為課程作業，授權依原課堂要求；若需重用、增修或公開，請先確認課程規範或與作者聯繫。
