# 從 NotebookLM 提取實用單字並生成學習檔案

此 skill 用於從 NotebookLM 提取特定集數的實用單字（非 N3 標準單字），生成 CSV 檔案。

## 使用時機

當用戶提供：
- NotebookLM 中的集數名稱（如 `#47`、`第47集`）
- 或特定主題名稱（如 `自己肯定感`、`敬語`）
- 需要提取日本人日常使用但不在 JLPT 標準範圍內的實用單字

## 工作流程

### Step 1: 從 NotebookLM 查詢單字

使用 `/notebooklm` skill 查詢該集的實用單字：

```bash
cd /home/boshi/.claude/skills/notebooklm && python scripts/run.py ask_question.py \
  --question "請列出第[X]集中出現的實用單字，特別是日本人日常會用但不在N3範圍內的詞彙。請包含每個單字的讀音、意思、例句和用法說明"
```

若回答不完整，追問：

```bash
cd /home/boshi/.claude/skills/notebooklm && python scripts/run.py ask_question.py \
  --question "繼續列出第[X]集中的其他實用單字，除了[已列出的單字]之外，還有哪些口語或慣用表現？"
```

### Step 2: 整理單字清單

從 NotebookLM 回答中提取：
- 單字/表現（Expression）- 含振假名標記
- 音調（PitchAccent）- 若不確定可留空
- 例句（Example）- 含振假名標記
- 類型/備註（Type）- 如「口語」「慣用表現」「擬態語」等
- 意思（Meaning）
- 例句翻譯（ExampleTranslate）
- 讀音（Yomikata）

### Step 3: 創建 CSV 檔案

在 `Vocabulary/Practical/` 資料夾創建檔案，命名格式：`ep[集數]-[主題].csv`

CSV 標題行（必須完全符合）：
```csv
Expression,PitchAccent,Example,Type,Meaning,ExampleTranslate,Yomikata
```

範例內容：
```csv
Expression,PitchAccent,Example,Type,Meaning,ExampleTranslate,Yomikata
へこむ,, 自分[じぶん]は 全然[ぜんぜん]できないなって、へこんでいた 時期[じき]がありました。,口語常用，原意為「凹陷」,感到沮喪、意志消沉,曾有一段時期覺得自己什麼都做不好而感到沮喪。,へこむ
ぐるぐる,, 頭[あたま]の 中[なか]で、ぐるぐる 考[かんが]えてしまうことがあるんですよ。,擬態語，形容不斷繞圈,腦袋轉個不停、鑽牛角尖,有時候腦袋會一直轉個不停地想。,ぐるぐる
```

### Step 4: 推送到 Git

```bash
git add "Vocabulary/Practical/ep[X]-[主題].csv"
git commit -m "Add practical vocabulary from episode [X] ([主題])

Vocabulary covered:
- [單字1], [單字2], [單字3]...

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
git push
```

## 重要規則

### CSV 欄位說明
| 欄位 | 說明 | 範例 |
|------|------|------|
| Expression | 單字或表現（含振假名） | 振[ふ]り 回[まわ]される |
| PitchAccent | 音調標記（可留空） | |
| Example | 日文例句（含振假名） | 頭[あたま]の 中[なか]で、ぐるぐる 考[かんが]えてしまう。 |
| Type | 類型或補充說明 | 口語、擬態語、慣用表現 |
| Meaning | 中文意思 | 腦袋轉個不停 |
| ExampleTranslate | 例句翻譯 | 有時候腦袋會一直轉個不停地想。 |
| Yomikata | 讀音（平假名） | ぐるぐる |

### 漢字振假名標注規則

#### 標注格式
```
漢字[讀音]
```

#### 空白字元規則
- 每個漢字（或漢字詞組）前面必須加上一個空白字元

#### 範例
| 錯誤 | 正確 |
|------|------|
| 今朝、薬を飲み忘れました | 今朝[けさ]、 薬[くすり]を 飲[の]み 忘[わす]れました |
| 電車で行く | 電車[でんしゃ]で 行[い]く |
| 自己肯定感 | 自己肯定感[じここうていかん] |
| 振り回される | 振[ふ]り 回[まわ]される |

### 單字分類標準
優先收錄以下類型：
- **口語表現**：日常會話常用但教科書少見
- **擬態語/擬聲語**：如 ぐるぐる、キラキラ、バリバリ
- **慣用表現**：如 胸[むね]を 張[は]って、 切[き]りがない
- **流行用語**：近年常用的新詞彙
- **文化相關詞彙**：如 反省会[はんせいかい]、 相槌[あいづち]を 打[う]つ

### 格式注意事項
- ⚠️ 欄位順序必須完全符合：`Expression,PitchAccent,Example,Type,Meaning,ExampleTranslate,Yomikata`
- ⚠️ 每行必須有 7 個欄位（可留空但逗號必須有）
- ⚠️ 若內容包含逗號，需用雙引號包覆
- ⚠️ 所有漢字必須加振假名標記：`漢字[讀音]`
- ⚠️ 漢字前必須加空格

## 範例：完整流程

**用戶輸入：** 「幫我整理 #47 的實用單字」

**執行步驟：**

1. `/notebooklm 請列出第47集中日本人常用但不在N3範圍內的實用單字`
2. 整理出 17 個實用單字
3. 創建 `Vocabulary/Practical/ep47-自己肯定感.csv`
4. 為所有漢字加上振假名標記
5. `git add && git commit && git push`

## 輸出檔案位置

```
nihongo_cc/
└── Vocabulary/
    └── Practical/
        └── ep[X]-[主題].csv
```

## 與其他 Skill 的關係

- **grammar-from-notebooklm.md**：用於提取文法，存放於 `Grammar/N3/`
- **vocabulary-from-notebooklm.md**（本 skill）：用於提取實用單字，存放於 `Vocabulary/Practical/`
- **grammar-analyzer.md**：提供振假名標記規則參考
