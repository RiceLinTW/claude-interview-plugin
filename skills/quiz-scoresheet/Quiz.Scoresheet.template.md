# Quiz Scoresheet — {name}

**面試日期：** {date}　　**面試官：** {interviewer}

> 每題附題庫參考答案，供對照打分。評分權重：正確性 40% / 完整性 30% / 清晰度 20% / 深度 10%。
> 分級：9–10 優秀 / 7–8 良好 / 5–6 及格 / 3–4 不佳 / 0–2 不及格。

---

{foreach category in grouped_questions}
## {Category}

{foreach question in category}
### Q{N}: {title} `({db_file_short} Q{db_N})`

**參考答案：**
{concise summary of the DB Answer section for this question}

| 分數 (0–10) | 備註 |
|:-----------:|------|
|             |      |

{/foreach}
{/foreach}

---

## 總覽

| 正確 (≥7) | 部分正確 (3–6) | 錯誤 (1–2) | 未答 (0) |
|:---------:|:--------------:|:----------:|:--------:|
| 　        | 　             | 　         | 　       |

**總分：** 　／100　（{level} 通過門檻 ≥ {threshold}）

> 整體評語見 `Record.md`。

