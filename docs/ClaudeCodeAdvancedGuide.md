# Claude Code 進階指南：從基礎到實戰完全攻略

---

## 目錄

1. [安裝與環境設定](#1-安裝與環境設定)
2. [基礎知識：觀察 Context 運作原理](#2-基礎知識觀察-context-運作原理)
3. [版本演進：延遲載入機制](#3-版本演進延遲載入機制)
4. [必改設定：優化使用體驗](#4-必改設定優化使用體驗)
5. [基本心法：Prompt 與互動技巧](#5-基本心法prompt-與互動技巧)
6. [Context 是最重要的 First Principle](#6-context-是最重要的-first-principle)
7. [Agent 四大超能力](#7-agent-四大超能力)
8. [Agentic Engineering 三部曲](#8-agentic-engineering-三部曲)
9. [Agent 開發三層次](#9-agent-開發三層次)
10. [實用小秘技](#10-實用小秘技)
11. [官方資源](#11-官方資源)

---

## 1. 安裝與環境設定

### Windows 安裝（官方推薦方式）

> ⚠️ **2025 年後官方已推出原生安裝方式，不再需要透過 npm 或第三方工具**

#### PowerShell（推薦）

```powershell
irm https://claude.ai/install.ps1 | iex
```

#### WinGet

```powershell
winget install Anthropic.ClaudeCode
```

#### 驗證安裝

```bash
claude --version
claude doctor
```

### 環境變數設定

> ⚠️ **舊版變數名稱 `ANTHROPIC_AUTH_TOKEN` 已廢除，請使用新名稱**

#### 必要變數

| 變數名稱 | 說明 | 範例 |
|----------|------|------|
| `ANTHROPIC_API_KEY` | API Key（從 console.anthropic.com 取得） | `sk-...` |

#### 選用變數

| 變數名稱 | 說明 | 範例 |
|----------|------|------|
| `ANTHROPIC_BASE_URL` | 自訂 API 端點（使用 Proxy 時） | `https://anyrouter.top` |
| `CLAUDE_CODE_GIT_BASH_PATH` | Git Bash 路徑（Windows） | `C:\Program Files\Git\bin\bash.exe` |

#### 設定方式

**PowerShell（Windows）**：
```powershell
$Env:ANTHROPIC_API_KEY = 'sk-....'
```

**Bash/Zsh（macOS/Linux）**：
```bash
export ANTHROPIC_API_KEY='sk-....'
```

### 安裝後初始設定

```bash
# 第一次執行會引導登入
claude

# 或設定 API Key
claude config set api-key

# 查看版本
claude --version
```

### 版本資訊

- **目前穩定版本**：v2.1.x（2026年2月最新為 v2.1.63）
- **VS Code 擴充**：請至 VS Code 擴充商店更新至最新版本

### 快捷鍵說明

- **Ctrl + R**：在終端機中是「反向搜尋歷史指令」（reverse-i-search），非「展開摺疊程式碼」
- 展開/摺疊程式碼請使用 VS Code 原生快捷鍵

---

## 2. 基礎知識：觀察 Context 運作原理

### 為什麼要觀察 Context？

學習 Claude Code 的第一步，是學會觀察它背後的運作機制。透過觀察：

- 交談過程（Conversation Flow）
- 資料結構（Data Structure）
- System Prompt 的變化

你可以深入理解 **Context 管理** 的重要性，進而學會如何設計有效的提示詞（Prompt）與工具。

### 核心心法

> **你懂 Context，Context 就會幫你。**
>
> 反之，如果你無視 Context 的運作原理，它就會在你不知不覺中出問題。（這就是所謂的 Context Rot / Attention Drift）

---

## 3. 版本演進：延遲載入機制

### Context 省四倍的秘密

在 v2.1.69 版本中，Claude Code 推出了革命性的**延遲載入（Deferred Tools）**機制，大幅減少 Context 的消耗。

### 比較：優化前 vs 優化後

| 項目             | 優化前         | 優化後                   |
| ---------------- | -------------- | ------------------------ |
| **Context 消耗** | 18,000 tokens  | 4,547 tokens             |
| **工具載入方式** | 啟動時全部載入 | 透過 ToolSearch 動態查詢 |

### 什麼是延遲載入？

延遲載入的核心概念是：**只有在需要的時候，才載入相關工具**。

Claude Code 現在只會在 Context 中保留一個名為 `ToolSearch` 的工具。當 Agent 需要使用某個工具時，必須先透過 `ToolSearch` 查詢可用工具，系統才會動態載入該工具。

### ToolSearch 的重要性

以下是官方對 ToolSearch 的說明（務必牢記）：

> **MANDATORY PREREQUISITE - THIS IS A HARD REQUIREMENT**
>
> You MUST use this tool to load deferred tools BEFORE calling them directly.
>
> This is a BLOCKING REQUIREMENT - deferred tools are NOT available until you load them using this tool.

翻譯：這是**強制性前提**，延遲工具必須先被發現/載入才能使用，否則呼叫會失敗。

### ToolSearch 查詢模式

#### 模式一：關鍵字搜尋（Keyword Search）

當你不確定要使用哪個工具時，用關鍵字搜尋：

```bash
# 範例
"list directory"      # 找尋列出目錄的工具
"notebook jupyter"    # 找尋 Notebook 編輯工具
"slack message"       # 找尋 Slack 訊息工具
```

- 最多返回 5 個相關工具
- 返回的工具**立即可用**，無需額外選擇步驟

#### 模式二：直接選擇（Direct Selection）

當你知道確切的工具名稱時，直接指定：

```bash
# 範例
"select:mcp__slack__read_channel"  # 載入特定 Slack 工具
"select:NotebookEdit"              # 載入 Notebook 編輯工具
"select:Read,Edit,Grep"           # 一次載入多個工具（用逗號分隔）
```

#### 模式三：必要關鍵字（Required Keyword）

當你只知道服務名稱但不知道確切工具時，用 `+` 前綴強制限定來源：

```bash
# 範例
"+linear create issue"   # 只要 Linear 的工具，且與 "create" 或 "issue" 相關
"+slack send"           # 只要 Slack 的工具，且與 "send" 相關
```

### 正確 vs 錯誤用法

#### ✅ 正確用法範例

**範例一：透過關鍵字搜尋**

```
User: I need to work with slack somehow
Assistant: [呼叫 ToolSearch，搜尋 "slack"]
Assistant: 找到 mcp__slack__read_channel
Assistant: [直接呼叫 mcp__slack__read_channel - 已自動載入]
```

**範例二：直接選擇**

```
User: Edit the Jupyter notebook
Assistant: [呼叫 ToolSearch，搜尋 "select:NotebookEdit"]
Assistant: [呼叫 NotebookEdit]
```

#### ❌ 錯誤用法範例

**範例一：未先載入就呼叫**

```
User: Read my slack messages
Assistant: [直接呼叫 mcp__slack__read_channel]
❌ 錯誤！必須先透過 ToolSearch 載入工具
```

**範例二：重複載入**

```
Assistant: [呼叫 ToolSearch 搜尋 "slack"，得到 mcp__slack__read_channel]
Assistant: [又呼叫 ToolSearch 搜尋 "select:mcp__slack__read_channel"]
❌ 錯誤！關鍵字搜尋已經載入工具，不需要再執行 select
```

### 延遲載入工具清單

以下工具目前採用延遲載入機制：

```
Agent, AskUserQuestion, Bash, Edit, EnterPlanMode, EnterWorktree,
ExitPlanMode, Glob, Grep, NotebookEdit, Read, SendMessage, Skill,
TaskOutput, TaskStop, TeamCreate, TeamDelete, TodoWrite, Write
```

### 這就是選對 Agent Harness 的價值

- Agent Harness 是你每天長時間使用的工具，攸關成敗
- 本質上就是 AI 基礎建設（Loop、Tools、Subagents 等）
- Claude Code v2.1.69 這類大改版讓你**什麼都不用做，效能就提升 4 倍**
- 一開始就選對 Agent Harness，全球頂尖工程師會幫你持續優化
- 你只需「坐享其成」，這就是所謂的「槓桿」

---

## 4. 必改設定：優化使用體驗

透過 `/config` 命令可以調整以下設定：

### 推薦設定

```json
{
  "reduce motion": true,
  "prompt suggestion": false,
  "verbose output": true,
  "default permission mode": "Bypass Permission"
}
```

### 各設定說明

| 設定                        | 建議值              | 原因                                |
| --------------------------- | ------------------- | ----------------------------------- |
| **reduce motion**           | `true`              | 減少畫面跳動與捲動後畫面錯亂的問題  |
| **prompt suggestion**       | `false`             | 預設建議功能通常無用且會干擾        |
| **verbose output**          | `true`              | 清楚看到每個 Tool Call 的請求與結果 |
| **default permission mode** | `Bypass Permission` | 降低執行到一半被中斷的機率          |

### 模型選擇 `/model`

- 一般任務使用 **Sonnet** 就足夠
- 沒必要不要使用 **Opus**（費用較高）
- 設定 `effort` 為 `medium` 即可
- 較低的 effort 會讓模型「想得少」，反應速度更快也更省錢

---

## 5. 基本心法：Prompt 與互動技巧

### 建議使用英文提問

經過 18 個月的測試驗證，**用英文提問效果最好**。

#### 技巧

1. **直接在 prompt 末尾加上 `in english`**
   - 即使你的英文不好，加上這兩字效果會顯著改善

2. **先中文提問，再請它譯成英文**
   - 範例：`先用中文描述需求，然後說 "answer in English"`

### 絕對不要自己寫 Prompt

- 你不會比 LLM 自己更懂 LLM 的運作原理
- 正確做法：告訴 AI 你想做什麼，讓 AI 自己生成 prompt
- 只要清楚解釋意圖、背景與考量，AI 通常能一次到位

#### 最佳實踐

1. 描述你想要達成的事情
2. 請 AI 寫幾份不同風格的 prompt 供你選擇
3. 記得請它也提供英文版本

### 善用 AI 生成 Prompt

- 你不會比 LLM 自己更懂 LLM 的運作原理
- 正確做法：告訴 AI 你想做什麼，讓 AI 自己生成 prompt
- 只要清楚解釋意圖、背景與考量，AI 通常能一次到位
- 建議使用 **Opus 4** 或 **Sonnet 4** 生成的 Prompt 效果較好

---

## 6. Context 是最重要的 First Principle

### Context 是神聖寶貴的稀有資源

- Claude 一般上限是 **200K tokens**（免費用戶）
- 付費用戶可升級到 **1M tokens**
- 每一滴 Context 都要省著用！

### Context Engineering 為何重要

市面上 99% 的相關文章都與**最佳化 Context** 有關，這就是為什麼 Context Engineering 極為重要的原因。

### 關鍵概念

| 概念                | 說明                                       |
| ------------------- | ------------------------------------------ |
| **Attention Drift** | LLM 的注意力隨著 Context 增長而分散        |
| **Context Rot**     | Context 過長導致模型「忘記」早期的重要資訊 |

### 核心原則

> 將 Context 當成你**最新的好朋友**，密切關心它的一切狀態！

---

## 7. Agent 四大超能力

如果今天你只能學會一件事，就是這四個超能力！

### 超能力一：Shell（指令操作）

- Agent 超會用 Shell 指令（glob, grep, awk, sed...）
- 能用 bash/zsh 做到的事，效能通常最好
- 有人說 **80% 的 RAG 都能用 glob/grep 取代**

### 超能力二：File System（檔案系統）

- 將資料外部化最重要的關鍵
- Plan、Todolist、Progress、Log 一律存硬碟
- 本質就是儲存 **Single Source of Truth**（單一真相來源）

### 超能力三：Scripting（腳本撰寫）

- Agent 超會寫程式（這就是為什麼 Vibe Coding 成為顯學）
- 主流做法：叫 AI 自己寫程式與執行，只返回最終結果

#### 為什麼要這樣做？

**目的：避免污染主線程的 Context**

**傳統做法**（錯誤）：

- 讀取三份 Markdown 文件
- 將全部內容輸入 LLM
- 這會佔用大量 Context 並造成 Attention Drift

**新做法**（正確）：

- 叫 AI 寫一支腳本（如 JS）
- 腳本從三份文件中提取「結論」部分，組合成一個字串
- 將這個簡短的字串返回給 LLM
- LLM 要處理的資料變少、Context 沒被污染、更省錢！

### 超能力四：Subagents（子代理）

- 擁有獨立 Context，可視為「免洗工具」
- 避免污染主線程的 Context
- 附帶好處：**可平行運算**，做事更有效率

#### 使用提示

```
use subagent to summarize /tmp/foo.md
```

### 綜合應用：效果更好

#### 範例一：讓 Subagent 寫程式與摘要

- 指派一個 Subagent 去寫程式與摘要
- 只需將最終結果返回給主線程
- 主線程的 Context 完全不被污染

#### 範例二：大量文件處理

- 如果你有幾百份 Markdown 要處理
- 派多個 Subagent 寫腳本同時處理
- 結果先存入硬碟，不要直接返回主線程
- 例如：存成 `summary.01.md`, `summary.02.md`...
- 等主線程需要時，用 `glob/grep` 查詢撈出相關內容

### 關鍵心法

> 除了在 CLAUDE.md 內明確指示 Agent 要善用這四項超能力外，你也要隨時在心中謹記提醒 Agent 這樣做事。
>
> **總之大原則：盡一切可能避免主線程寶貴的 Context 被污染，能不用就不用！**

---

## 8. Agentic Engineering 三部曲

### 笑話一則

> 寫程式與蓋教堂的共通之處是什麼？
>
> → 完工之後我們就開始祈禱

### 兩種不同的做法

| 做法                             | 結果                              |
| -------------------------------- | --------------------------------- |
| 只會許願然後希望奇蹟發生         | 這叫 **Vibe Coding** / 吃角子老虎 |
| 懂「許願 → 監工 → 驗收」完整套路 | 這叫 **Agentic Engineering**      |

---

### 第一部：許願（Wish）

#### 流程

1. **需求訪談**：清楚定義要做什麼
2. **背景調查**：收集相關資訊
3. **撰寫 Spec Draft**：初步規格文件
4. **Quick Pilot Run**：快速寫原型驗證可行性
5. **修正 Spec Final**：確定最終規格

> 詳情請參閱下面的「Agent 開發三層次」

---

### 第二部：監工（Monitor）

#### 核心原則

觀察過程中呼叫的工具與操作結果，**發現不對就立即中止**。

- 前提：人需坐在電腦前當「保姆」

#### Claude Code 不中斷大法

**1. Permission Mode 設定**

```json
{
  "defaultMode": "bypassPermissions",
  "skipDangerousModePermissionPrompt": true
}
```

- `bypassPermissions` = 啟動時使用 `--dangerously-skip-permissions`
- `skipDangerousModePermissionPrompt` = 盡量減少提問

**2. 禁用容易中斷的工具**

- `ExitPlanMode`
- `AskUserQuestion`

這兩個是最常導致中斷的元兇！

**3. 啟用 Ralph Loop Plugin**

```
https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum
```

**4. Auto Mode（自動模式）

- 可減少提問頻率，讓 Agent 更加自主操作
- 透過設定或命令啟用
- 可大幅降低中斷次數

#### Claude Remote（遠端遙控）

- 最新推出的遠端操作功能
- 等於「綁尿袋」，可從外面隨時連回家看進度與操作

---

### 第三部：驗收（Acceptance）

#### 學 Amazon：先寫「行銷文件」與「操作手冊」

| 文件類型     | 目的                                           |
| ------------ | ---------------------------------------------- |
| **行銷文件** | 確認需求是否想清楚、產品規劃是否完整且符合市場 |
| **使用手冊** | 確認程式功能是否完整、成品是否真的好用也實用   |

#### 範例一：Pattern Matcher

1. 寫完 Spec 後，要求先寫出 Usages Guide
2. 假裝程式已寫好，開始寫操作手冊
3. 觀察最終 Matcher 的語法與參數是否好用
4. 檢查是否覆蓋所有查詢需求
5. 通常這個階段會發現許多問題，回頭修改 Spec

#### 範例二：Usage Guide 成為驗收標準

1. 將 Usages Guide 寫成 Test Cases
2. 這些 Test Cases **另外保存**，不讓 Coding Agent 知道（稱為 Hold-out Set）
3. Coding Agent 完工後，用這些 Test Cases 驗收
4. 如果全通過，代表程式真的可靠

#### Retro 與 Review 不一樣

| 項目     | Retro（回顧）                | Review（審查）         |
| -------- | ---------------------------- | ---------------------- |
| **時機** | 寫完程式，Context 還沒消失前 | 寫完程式且測試皆通過後 |
| **視角** | 第一人稱，主觀檢討           | 第三人稱，客觀審視     |
| **重點** | 開發過程中的缺失             | 程式碼品質與最佳實踐   |

**Retro 可用的提示**：

```
now that you wrote the code, looking back at the process what did you do wrong or right?
If we were to start all over again, what would you do differently to make it better?
```

→ 叫它檢討完後**立即修正所有錯誤**，這招往往可大幅改善程式品質！

#### 不能盡信 Test Case

Agent 有時會投機取巧：

| 問題                 | 說明                                         |
| -------------------- | -------------------------------------------- |
| **無用測試**         | 寫一堆測試但沒測關鍵的「成功」與「失敗」劇情 |
| **作弊 assert**      | 寫無腦的 `assert(true)` 只為了讓測試通過     |
| **刪除失敗測試**     | 乾脆刪除一直失敗的 Test Case                 |
| **修改測試迎合結果** | 修改測試內容以符合程式實際輸出               |

**範例**：

- 正確：`assert.ok(add(1, 2), 3)` → 期望 1+2=3
- 作弊：`assert.ok(add(1, 2), 4)` → 如果結果是 4，就改測試！

> **大原則：Trust No One**

#### 每階段皆自評 Quality Score

在每個階段結束時，要求 Agent 給自己打分數作為 `eval` 標準。

#### 完美的驗收是多層次組合

| 階段     | 重點                                           |
| -------- | ---------------------------------------------- |
| **許願** | 透過各種手段在最早期就確保 Spec 正確且符合需求 |
| **監工** | 發現 Agent 需不斷中停提問，通常暗示規劃有問題  |
| **驗收** | 靠 Hold-out Set 與 Trust No One 兩大秘技把關   |

---

## 9. Agent 開發三層次

### 層次一：Plan Mode + Tasklist（輕量級）

#### 說明

使用 Claude Code 內建的 Plan Mode + Tasklist 工具。

#### 範例 Prompt

```
write a fn and test for js string reverser
must enter plan mode to make a plan, create todolist to track them,
then start working.
```

#### 產出

- 建立 Plan：`~/.claude/plans/<project-name>.md`
- 建立 Tasklist：`~/.claude/tasks/<task-id>.json`
  - 共四份 JSON
  - Tasklist 前身為 Todolist

#### 優點

- **60%** 的輕量工作可用這招無腦解決

#### 缺點

- Agent 常做到一半**忘了自己的 Plan**
- 可能**多次重建 Plan 並重複做事**
- 最糟的是**沒更新 Tasklist 內每張票的狀態**
- 如果中間當掉就不知做到哪
- `/resume` 與 `/continue` 也救不回來

---

### 層次二：TDD 導向工法（主流）

#### 說明

Test-Driven Development（TDD）導向，包含：

- 需求訪談
- 規格撰寫
- 切票細節
- 實作規範

最重要的是：寫程式時嚴格要求採 `紅-綠-重構` 循環。

#### 代表工具：Superpowers

- GitHub: https://github.com/obra/superpowers
- 上述基本開發步驟皆已內建
- 無腦依序操作它提供的 `/brainstorm`, `/write-plan`, `/execute-plan` 就好

#### 優點

- Prompt 寫得極佳，大部分該想到的事皆已內建
- 安裝完 Plugin 就會自動啟用多份 Command、Skills 與 Hooks
- 取代 Claude Code 內建的 EnterPlanMode 與 Tasklist
- **80%** 的工作可用這招解決

#### 缺點

- 流程控管是靠寫在 Prompt 內的 Graphviz 流程圖
- 等於要求**隨機變化的 LLM 控管不可變的狀態機**
- 這本質上有所衝突，不完全可靠

---

### 層次三：SDD 導向工法（重度）

#### 說明

Specification-Driven Development（規格驅動開發）

#### 核心概念

開工前先用**自然語言**定義好所有開發文件。

#### 常見功能

| 功能                           | 說明                                        |
| ------------------------------ | ------------------------------------------- |
| Specs as first-class artifacts | 規格視為一等公民（存入 Repo）               |
| Structured requirements        | 結構化需求（Stories + Acceptance Criteria） |
| Plan generation from specs     | 從規格生成計畫                              |
| Task decomposition             | 任務分解                                    |
| Traceability                   | 可追溯性（Spec → Task → Code）              |
| Automated implementation       | 自動化或引導式實作                          |
| Test generation from specs     | 從規格生成測試                              |
| Versioning                     | 版本控制                                    |

#### 知名品牌

| 名稱            | 連結                                                        |
| --------------- | ----------------------------------------------------------- |
| GitHub Spec-Kit | https://github.com/github/spec-kit/blob/main/spec-driven.md |
| OpenSpec        | https://github.com/Fission-AI/OpenSpec                      |
| Kiro            | https://aws.amazon.com/documentation-overview/kiro/         |

#### 優點

- 重度 Human-in-the-loop，由人類主導設計與驗收
- **95%** 的工作能靠這招安全實作

#### 缺點

| 問題         | 說明                         |
| ------------ | ---------------------------- |
| 巨量文件     | 產生大量文件，讀起來耗時費力 |
| 昂貴         | 過程耗用大量 Tokens          |
| 仍有出錯可能 | 既使層層把關，仍可能做歪     |

> **極吃主導者（人類）的品味與能力**
>
> - 如果本身不懂技術與需求控管，就只能被 AI 牽著鼻子走
> - 無法在早期發現規劃上的問題
> - 出錯後也無法察覺或人工修復

---

### 建議

| 建議                   | 說明                                     |
| ---------------------- | ---------------------------------------- |
| 避免「鎚子心態」       | 不要手上有了鎚子，所有東西都像釘子       |
| SDD 不見得適用所有場合 | 視情況選擇合適的層次                     |
| 先用層次一試試         | 叫 Claude Code 快速試做觀察結果          |
| 甜蜜點通常是層次二     | TDD 導向是最平衡的選擇                   |
| 不要過度規劃           | 但產出的文件一定要人工嚴密監看           |
| 早期制止與修正         | 發現不對勁要立刻停止，否則浪費時間與金錢 |

---

## 10. 實用小秘技

### 對 Code 沒感情

- 程式誰來寫都一樣
- 魔改多少次也沒差
- **只在意最終產出結果正確，驗收有過就好！**

### 善用 External Truth 追蹤進度

- Plan、Todolist、Logs 都要存硬碟
- **千萬不要信任內建的 Plan 與 Todolist**
- Agent 常跑到一半就忘記
- 仿效 Superpowers，改用**自己的外部 Plan/Todolist** 最可靠

### 關閉不需要的工具

- 用不到的工具就關掉，可省 Context
- 例如使用 Superpowers 後可關閉：
  - `EnterPlanMode`
  - `TaskCreate`
  - ...等五六支工具
- 用不到 Python 的話，連 `NotebookEdit` 也關掉

### 工作「切小塊」是不敗王道

#### 原理

- Context 容量有限
- 當 Context 變大，Agent 就會錯亂
- 最佳策略：**輕量時讓 Context 重置**（清空重來）

#### 做法

1. 工作切割成小塊
2. 每小段工作完成即存檔
3. 使用 `/clear` 重置 Context
4. 再進行下個工作

#### 以 Code Review 為例

**錯誤做法（一次做三件事）**：

```
進行 review 找出錯誤
針對所有錯誤規畫如何修復
真正進行修復
```

- 當工作複雜時，Agent 可能做到一半錯亂

**正確做法（拆成三件工作）**：

1. **Step 1**：專心進行 Review，寫出 `review.md`
2. **Step 2**：讀取 `review.md`，規畫修復並寫出 `plan.md`
3. **Step 3**：讀取 `plan.md`，依指示修復並寫出最終 `report.md`

**優點**：

- 三步可由不同 Subagent 接續進行
- 每一步皆由**乾淨的 Context** 開始
- 每步皆能專心做好一件事而不分心
- 大幅改善品質，也提升效率（少出錯就不需多次重試）

---

## 11. 官方資源

### 官方文件

| 資源 | 連結 |
|------|------|
| 官方安裝指南 | https://claudefa.st/blog/guide/installation-guide |
| 環境變數配置 | https://www.claudeinsider.com/docs/configuration/environment |
| 完整更新日誌 | https://claudefa.st/blog/guide/changelog |
| Claude Code GitHub | https://github.com/anthropics/claude-code |

### 版本查詢

```bash
# 查看當前版本
claude --version

# 檢查健康狀態
claude doctor
```

---

## 結論

Claude Code 是一個強大的 AI 編碼伴侶，但要發揮它的最大價值，需要：

1. **理解 Context** 的運作原理與重要性
2. **善用四大超能力**（Shell、FS、Scripting、Subagents）
3. **建立完整的工作流程**（許願 → 監工 → 驗收）
4. **選擇合適的開發層次**（Plan Mode → TDD → SDD）
5. **保持謹慎與批判**（Trust No One）

記住：AI 是強大的工具，但最終還是需要人類來把關與決策。
