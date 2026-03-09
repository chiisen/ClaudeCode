# `/cost` 命令：查看 Claude Code 使用費用與效能統計

## 命令功能說明

`/cost` 是 Claude Code 內建的 slash command，用於查看當前工作階段的資源消耗情況。

## 使用方式

```bash
> /cost
```

## 輸出範例

```
  ⎿  Total cost:            $0.2836
     Total duration (API):  2m 37.1s
     Total duration (wall): 55m 27.4s
     Total code changes:    426 lines added, 0 lines removed
     Usage by model:
            claude-sonnet:  144 input, 7.0k output, 327.5k cache read, 21.3k cache write
         claude-3-5-haiku:  805 input, 31 output, 0 cache read, 0 cache write
```

## 欄位說明

| 欄位名稱 | 意義 |
|---------|------|
| **Total cost** | 此次對話累積的花費（美元） |
| **Total duration (API)** | API 實際運算耗時 |
| **Total duration (wall)** | 牆上時鐘經過的總時間（包含等待回應、思考等） |
| **Total code changes** | 程式碼變動行數（新增 / 刪除） |
| **Usage by model** | 各模型的使用量統計 |

## 模型使用量詳解

### claude-sonnet（主要模型）
- **input**：輸入的 token 數量（使用者 prompt + context）
- **output**：輸出的 token 數量（Claude 的回覆）
- **cache read**：從快取讀取的 token 數量（較便宜）
- **cache write**：寫入快取的 token 數量（較便宜）

### claude-3-5-haiku（輕量模型）
- 用於簡單快速的任務，如檔案搜尋、簡單查詢
- 費用較低，適合不需深度思考的場景

## 省錢小技巧

1. **減少無意義的對話**：每次互動都會消耗 tokens
2. **利用快取**：重複查詢相同檔案時，cache read 可大幅降低成本
3. **適時使用 Haiku**：簡單任務切換到 Haiku 模型可省錢
4. **合併操作**：尽量将相关操作集中在一次对话中完成

## 相關命令

- `/status`：查看當前工作階段的狀態與設定
- `/model`：切換使用的模型
