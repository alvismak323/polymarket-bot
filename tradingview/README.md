# PX Trend Futures：TradingView 策略 → Pionex 合約自動交易

`pionex_trend_futures.pine` 係一個 TradingView Pine Script v6 **strategy**，
設計成配合 **Pionex 信號機器人 (Signal Bot)** 用 Webhook 自動開平倉。

## 策略邏輯

| 部分 | 規則 |
|---|---|
| 開多 | 快 EMA(21) 上穿慢 EMA(55)，收市價喺 200 EMA 之上，ADX(14) > 20 |
| 開空 | 快 EMA 下穿慢 EMA，收市價喺 200 EMA 之下，ADX > 20 |
| 止蝕 | 入場價 ∓ 2 × ATR(14) |
| 止盈 | 入場價 ± 3 × ATR（可以改用移動止蝕） |
| 反向信號 | 直接反手；如果嗰個方向已關閉，就只平倉 |
| 倉位 | 止蝕觸發時最多虧本金 1%，名義價值上限 5 倍本金 |
| 成本 | 回測已計手續費 0.05% 同 2 tick 滑價 |

信號只用已收盤 K 線計算，所以唔會 repaint。所有參數都可以喺策略「設定 → 輸入」度改。

## 設定步驟

### 1. 將策略加入 TradingView
1. 打開 TradingView 圖表，揀你要交易嘅合約，例如 `BINANCE:BTCUSDT.P` 或者 Pionex 有嘅永續合約。
   建議由 1 小時或 4 小時圖開始。
2. 下面打開 **Pine Editor**，將 `pionex_trend_futures.pine` 全部內容貼入去。
3. 撳 **儲存**，再撳 **加入圖表**。
4. 喺 **Strategy Tester** 睇回測結果，調好參數先好上真錢。

### 2. 喺 Pionex 開信號機器人
1. Pionex App 或網頁：交易機器人 → **信號機器人 (Signal Bot)** → 建立 / 用 TradingView 信號。
2. Pionex 會畀你兩樣嘢：**Webhook URL** 同 **Alert Message（JSON 模板）**。
3. 喺 Pionex 設定交易對、**槓桿**（建議 ≤ 3x）同投入金額，然後啟動機器人。

### 3. 喺 TradingView 開 Alert
1. 圖表上撳 **Alert（鬧鐘）** → Condition 揀 **PX-Trend**，再揀 **Order fills only**（只喺成交時觸發）。
2. **Message**：**原封不動**貼入 Pionex 畀你嘅 JSON 模板。
   入面嘅 `{{strategy.order.action}}`、`{{strategy.position_size}}` 等佔位符會由呢個策略自動填寫。
3. **Notifications** → 剔 **Webhook URL**，貼入 Pionex 畀你嘅 URL。
4. Expiration 揀 Open-ended（永不過期），然後建立。

> ⚠️ TradingView 要**付費計劃**（Essential 或以上）同開咗 **2FA** 先可以用 Webhook。

## 注意事項
- **實際落單金額同槓桿由 Pionex 機器人設定決定**。TradingView 回測入面嘅倉位大小只係用嚟評估策略，唔一定等於 Pionex 實際下單數量。
- TradingView 圖表嘅交易所同 Pionex 價格會有少少差異，止蝕止盈會喺 TradingView 價格觸發，再由 Pionex 以市價執行。
- 改完策略參數之後要**刪除舊 Alert 再重新建立**，因為 Alert 會鎖定建立嗰陣嘅參數。
- 高槓桿好容易爆倉。先用細金額試行一至兩星期，確認開平倉同 TradingView 一致先加大注碼。
- 回測表現唔代表將來收益。呢個策略唔構成投資建議。
