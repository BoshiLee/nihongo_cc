# 從 NotebookLM 提取文法並生成學習檔案

此 skill 用於從 NotebookLM 提取特定集數的文法，生成完整的 N3 文法學習 CSV 檔案。

## 使用時機

當用戶提供：
- NotebookLM 中的集數名稱（如 `#49`、`第49集`）
- 或特定主題名稱（如 `駅伝`、`敬語`）

## 工作流程

### Step 1: 從 NotebookLM 查詢文法

使用 `/notebooklm` skill 查詢該集的重要文法：

```bash
cd /home/boshi/.claude/skills/notebooklm && python scripts/run.py ask_question.py \
  --question "請列出第[X]集中使用到的所有重要文法，包含每個文法的解釋、例句和用法說明"
```

若回答不完整，追問：

```bash
cd /home/boshi/.claude/skills/notebooklm && python scripts/run.py ask_question.py \
  --question "繼續列出第[X]集中的其他重要文法，除了[已列出的文法]之外，還有哪些N3文法？請包含解釋和例句"
```

### Step 2: 整理文法清單

從 NotebookLM 回答中提取：
- 文法句型（Pattern）
- 基本說明（Explanation）
- 例句（Example Sentence）- 需加上振假名 `漢字[讀音]`
- 翻譯（Example Translation）
- 補充說明（Additional Notes）
- 注意事項（Pitfalls）

### Step 3: 創建初稿 CSV

在 `Grammar/N3/` 資料夾創建檔案，命名格式：`N3 - ep[集數].csv`

CSV 標題行：
```csv
Pattern,Explanation,Example Sentence,Example Translation,Additional Notes,Pitfalls
```

初稿內容範例：
```csv
〜について,表示「關於～的內容」。用法：名詞＋について, 駅伝[えきでん]について 話[はな]したいと 思[おも]います。,我想聊關於驛站接力賽。,「について」是最常用的「關於」表達。,❌ 與「にとって」混淆
```

### Step 4: 使用 grammar-analyzer.md 補充詳細解析

依照 `grammar-analyzer.md` skill 的格式，為每個文法添加：

#### Explanation 欄位追加：

```html
<br><br>🧠 文法解析：<br>
<ul>
  <li><div><b>句子片段</b></div>
    <ul>
      <li><div><b>單字</b>：詞性，意思是「中文意思」。</div></li>
      <li><div><b>助詞</b>：格助詞/提示助詞，在這裡表示<b>具體功能</b>。</div></li>
    </ul>
  </li>
</ul>

<br><br>📘 動詞補充（以「動詞」為例 → <b>動詞類型</b>）<br>
<table><tbody>
  <tr><td>變化</td><td>動詞普通形</td><td>說明</td></tr>
  <tr><td>肯定（現在・未來）</td><td>辭書形</td><td>基本意思</td></tr>
  <tr><td>否定</td><td>ない形</td><td>不...</td></tr>
  <tr><td>過去</td><td>た形</td><td>...了</td></tr>
  <tr><td>過去否定</td><td>なかった形</td><td>沒...</td></tr>
  <tr><td><b>て形（若本句使用）</b></td><td><b>て形</b></td><td><b>用途說明</b></td></tr>
  <tr><td>意向形</td><td>意向形</td><td>來...吧</td></tr>
</tbody></table>
```

#### Example Translation 欄位追加：

```html
<br><br>🧾 單字速記：<br>
<ul>
  <li><b>單字</b>：中文意思</li>
  <li><b>助詞</b>：功能說明</li>
</ul>
```

#### Additional Notes 欄位追加：

```html
<br><br>📝 語感補充：<br>
<ul>
  <li><b>相似文法對比</b>
    <ul>
      <li>A：特點/用法</li>
      <li>B：特點/用法</li>
    </ul>
  </li>
  <li><b>使用場景</b>：具體說明</li>
</ul>
```

### Step 5: 推送到 Git

```bash
git add "Grammar/N3/N3 - ep[X].csv"
git commit -m "Add N3 grammar from episode [X] ([主題])

Grammar points covered:
- [文法1], [文法2], [文法3]...

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
git push
```

## 重要規則

### CSV 格式規則
- ⚠️ 禁止使用真正的換行符，必須使用 `<br>` 標籤
- ⚠️ 每行必須有 6 個欄位
- ⚠️ 漢字必須加振假名：`漢字[讀音]`，前面加空格

### 振假名格式
```
正確：今朝[けさ]、 薬[くすり]を 飲[の]み 忘[わす]れました
錯誤：今朝、薬を飲み忘れました
```

### 動詞變化規則速查

| 動詞類型 | 肯定 | 否定 | 過去 | 過去否定 | て形 | 意向形 |
|---------|------|------|------|----------|------|--------|
| 五段動詞 | 〜う | 〜わない | 〜った | 〜わなかった | 〜って | 〜おう |
| 上一段動詞 | 〜る | 〜ない | 〜た | 〜なかった | 〜て | 〜よう |
| 下一段動詞 | 〜る | 〜ない | 〜た | 〜なかった | 〜て | 〜よう |
| カ變動詞 | 来る | 来ない | 来た | 来なかった | 来て | 来よう |
| サ變動詞 | する | しない | した | しなかった | して | しよう |

## 範例：完整流程

**用戶輸入：** 「幫我整理 #49 的文法」

**執行步驟：**

1. `/notebooklm 請列出第49集中使用到的所有重要文法`
2. 整理出 13 個文法點
3. 創建 `Grammar/N3/N3 - ep49.csv`
4. 為每個文法添加 🧠📘🧾📝 詳細解析
5. `git add && git commit && git push`

## 輸出檔案位置

```
nihongo_cc/
└── Grammar/
    └── N3/
        └── N3 - ep[X].csv
```
