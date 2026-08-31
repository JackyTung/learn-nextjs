# 字體載入最佳化與 0 CLS (Cumulative Layout Shift) 筆記

## 什麼是 CLS (版面偏移)？
字體從網路下載需要時間。當網頁剛載入時，通常會先用系統的預設字體來顯示文字。當自訂字體下載完成並「切換 (Swap)」上去時，由於兩種字體的幾何尺寸（寬度、高度、行距）不同，會導致文字佔用的空間改變，進而把下方的圖片或按鈕往下擠，造成畫面跳動。這就是所謂的 **CLS (Cumulative Layout Shift)**。

---

## 過去的傳統解法 (無 Next.js 時代)
過去為了追求 0 CLS，工程師需要在效能與美觀之間做出妥協，或是進行繁瑣的手工設定。主要有以下 4 種作法：

### 1. 手動使用 CSS `size-adjust` (最費工)
手動計算目標字體與備用字體（Fallback Font）的尺寸差異，並在 `@font-face` 中使用 `size-adjust`、`ascent-override`、`descent-override` 等屬性，把備用字體「拉伸/縮小」成跟目標字體一模一樣大的邊界框。
這樣即使字體發生切換，版面的長寬也不會改變。
* **缺點**：需要依賴外部工具（如 Font Style Matcher）憑肉眼或手工精算，非常耗時。

### 2. 控制載入策略：`font-display: optional`
設定 `@font-face` 的 `font-display` 屬性為 `optional`。
瀏覽器只會給予極短的時間（約 100 毫秒）等待字體下載。如果來不及，瀏覽器就會永久使用備用字體，即使字體後來下載完了也**絕對不會替換**上去（會默默存入快取供使用者下次換頁時使用）。
* **優點**：保證 0 CLS，保護閱讀體驗。
* **缺點**：犧牲了首次造訪者的視覺體驗（看不到漂亮字體）。

### 3. 資源預先載入：`<link rel="preload">`
在 HTML 的 `<head>` 最上方加入 `<link rel="preload" href="..." as="font">`，強制瀏覽器以最高優先級提早下載字體，盡可能縮短等待的空窗期。
* **缺點**：如果預先載入太多字體，會拖慢網頁其他更重要資源（如 JS, CSS）的載入速度。

### 4. 終極解法：系統字體 (System Fonts Stack)
完全放棄 Web Fonts，直接宣告使用作業系統內建的字體：
`font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto...`
* **優點**：因為完全不需下載網路資源，載入時間為 0，CLS 也是完美的 0。

---

## 現代解法：Next.js `next/font` 的黑魔法
Next.js 內建的 `next/font` 模組，將上述的最佳實踐完全自動化，讓開發者不用再做任何手動設定，就能享受完美的 0 CLS。

### 核心優勢與底層原理：

1. **全自動的 `size-adjust` 精密計算**
   Next.js 會在執行 `npm run build` (打包建置) 的階段，自動在背景分析你選擇的字體度量 (Font Metrics)。它會幫你生成一段帶有精算過 `size-adjust` 數值的 CSS，讓你的備用字體長寬比例與目標字體完全吻合，藉此達成完美的 0 CLS。

2. **自動 Self-hosting (本地代管字體)**
   如果你使用的是 Google Fonts，Next.js 會在建置時自動將字型檔從 Google 伺服器下載下來，並跟著你的專案一起打包送出。
   這省去了連線到 `fonts.googleapis.com` 的 DNS 解析與網路連線時間，而且使用者的瀏覽器不會發送任何請求給 Google，完全消除了隱私追蹤 (GDPR) 的疑慮。

3. **極致的開發體驗 (DX)**
   開發者只需寫兩行程式碼：
   ```javascript
   import { Inter } from 'next/font/google'
   const inter = Inter({ subsets: ['latin'] })
   ```
   接著把 `inter.className` 套用到 HTML 標籤上即可，無需手動下載檔案、寫 CSS 宣告或微調比例，繁瑣的工程細節 Next.js 全部幫你處理妥當！
