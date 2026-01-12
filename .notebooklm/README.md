# NotebookLM 配置

此資料夾用於管理所有 NotebookLM 筆記本的連結和配置。

## 配置文件

**檔案：** `config.json`

### 結構

```json
{
  "notebooks": [
    {
      "name": "筆記本名稱",
      "url": "NotebookLM URL",
      "description": "筆記本說明",
      "added_date": "YYYY-MM-DD",
      "topics": ["主題1", "主題2"],
      "primary": true
    }
  ],
  "default_notebook": "預設筆記本名稱"
}
```

## 目前的筆記本

### N3 Grammar Analysis
- **URL:** https://notebooklm.google.com/notebook/55da6f98-4687-4547-a1d9-a2641928f96f
- **用途:** N3 文法分析與確認
- **狀態:** 主要筆記本

## 如何新增筆記本

1. 編輯 `config.json`
2. 在 `notebooks` 陣列中新增筆記本物件
3. 如果要設為預設，更新 `default_notebook` 欄位

## 使用方式

在 Claude Code 中，可以透過以下 skills 使用 NotebookLM：
- `grammar-from-notebooklm.md` - 從 NotebookLM 提取文法
- `vocabulary-from-notebooklm.md` - 從 NotebookLM 提取單字

這些 skills 會自動使用預設的 NotebookLM 筆記本。
