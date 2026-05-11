# Pokemon Champions 寶可夢截圖辨識 Agent 流程

## 目標
讓 OpenClaw 在看到 Pokémon Champions 對戰截圖（尤其右側六隻隊伍）時，優先用「圖鑑比對 + 屬性圖示 + 外型特徵」辨識寶可夢，而不是只靠一般 LLM 猜測。

---

# 核心辨識流程（Pipeline）

```text
輸入截圖
↓
Step 1 裁切右側六隻隊伍區域
↓
Step 2 辨識屬性圖示（以52Poké屬性圖示作標準）
↓
Step 3 sprite / 外型比對（圖鑑資料庫）
↓
Step 4 shiny / 型態 / 顏色差異比對
↓
Step 5 候選交叉驗證（Pokédex / PokéAPI）
↓
輸出最終六隻寶可夢名單
```

---

# Step 1 — 圖像前處理
## 原則
只裁切右側六隻，不要送整張戰鬥畫面。

## 目的
降低干擾：
- UI元素
- 血條
- 技能欄
- 訓練家資訊

---

# Step 2 — 屬性圖示辨識（高優先）
參考：
https://wiki.52poke.com/wiki/屬性

## 規則
先看屬性，再看外型。

例：

冰 + 幽靈
→ 候選大幅縮小
→ 雪妖女優先

不要反過來。

---

# Step 3 — 寶可夢外型比對
## 使用資料源
- PokéAPI sprites
- 官方 artwork
- form sprites
- shiny sprites

比對：
- 輪廓
- 頭部特徵
- 配件/武器
- 身體比例

---

# Step 4 — 顏色 / 異色 / 型態處理
## 必須檢查
### 異色
若輪廓一致但顏色差異：

判定：
- Normal
- Shiny

---

## 型態
處理：
- Rotom各型態
- 地區型
- 特殊型態

不要只辨認基礎種。

---

# Step 5 — 候選交叉驗證
規則：

```text
若外型辨識結果
與屬性圖示衝突
→ 以屬性圖示優先
```

外型只是候選，
屬性用來排錯。

---

# 推薦工具 Stack

## 必裝
### OCR
Tesseract

```bash
brew install tesseract
```

---

## Vision
LLaVA

```bash
ollama pull llava
```

---

## 圖像比對
推薦：
- OpenCV
- CLIP embedding matcher

進階：
- YOLO Pokémon icon detector

---

## 圖鑑資料
PokéAPI

https://pokeapi.co

---

# 判斷策略（重要）
## 三重驗證原則
每隻都經過：

```text
外型
+ 屬性圖示
+ 圖鑑比對
```

至少兩者一致才確定。

---

# OpenClaw Prompt模板
遇到圖片時使用：

```text
請先只辨識右側六隻寶可夢。
步驟：
1. 先讀屬性圖示
2. 用屬性限制候選
3. 再用外型與圖鑑比對確認
4. 檢查是否可能為異色或特殊型態
5. 最後列出六隻名稱
先不要分析戰術。
```

---

# 若辨識不確定
輸出：

```text
高信心：
...
低信心候選：
...
```

不要硬猜。

---

# 實戰示例
## 正確示範
看到：

冰 + 幽靈

先鎖定：
- 雪妖女

再用外型確認。

而不是先看外型猜成其他寶可夢。

---

# 原則總結
## 優先順序
```text
屬性圖示
> 圖鑑比對
> 外型直覺
> 純LLM猜測
```

---

# 未來可擴充
可加入：
- 對手隊伍自動辨識
- 自動選出推薦
- 招式/道具推測
- 對戰 Matchup 分析

可進一步做成 Pokémon Champions Agent。
