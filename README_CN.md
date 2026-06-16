# translate-humanize

[English](README.md)

一个 agent skill，通过 DeepL 往返翻译链 + assistant 驱动的 `paper-spine-humanize` 改写 + 术语修正，降低学术英文段落的 AI 检测特征。

---

## 这个 skill 做什么

1. **翻译链** —— agent 运行辅助脚本，将原文经过 `EN → JA → KO → ES → PT → EN` 往返翻译。这能打散 LLM 特有的句式，同时保留原意。
2. **Assistant humanize** —— agent 使用 `paper-spine-humanize` heavy-tier 原则改写回译后的英文（更短句子、更少连接词、节奏变化）。
3. **术语修正** —— agent 修正 `Humanized English` 中变得过于松散、口语化或不够准确的词，恢复学术准确性，同时尊重原文作者的术语选择。

本 skill 中的人类化框架参考自 [PaperSpine](https://github.com/WUBING2023/PaperSpine)，特别是其中的 `paper-spine-humanize` 分层写作指令。

如果没有 DeepL API key，agent 可以利用自身的多语言能力自行生成翻译链。

---

## 适用场景

- 论文/报告中的某段被 AIGC 检测器标红。
- 单纯结构人类化效果不够。
- 希望对孤立段落进行可重复的降 AI 率处理。

---

## 不适用场景

- 任何不能承受改写风险的内容（公式、精确数字、法律文本）。
- 一次性处理很长的文档 —— 应先拆分。
- 目标语言不是英文的场景。

---

## skill 文件结构

```text
translate-humanize/
  SKILL.md                      # 面向 agent 的协议
  translate_humanize_deepl.py   # DeepL 翻译链辅助脚本
  README.md
  README_CN.md
  LICENSE                       # MIT + PaperSpine 署名
  requirements.txt              # python-docx
  .env.example                  # 环境变量示例
  examples/
    sample_input.docx           # 示例输入 docx
```

---

## 安装到 agent

### kimi-code

将 skill 文件夹放到 kimi-code 可以发现的位置，例如：

```text
<项目根目录>/.kimi-code/skills/translate-humanize/SKILL.md
```

或用户级 skill 目录：

```text
~/.kimi-code/skills/translate-humanize/SKILL.md
```

### Claude Code

复制 skill 文件夹到：

```text
~/.claude/skills/translate-humanize/
```

### Codex

复制 skill 文件夹到：

```text
~/.codex/skills/translate-humanize/
```

### OpenClaw

复制 skill 文件夹到：

```text
~/.openclaw/skills/translate-humanize/
```

---

## 依赖

- 辅助脚本需要 `python-docx`。
- 使用 DeepL 翻译链时需要环境变量 `DEEPL_API_KEY`。

如需安装唯一依赖：

```bash
pip install python-docx
```

---

## agent 如何使用

用户提供一段英文和目标路径：

```text
用 translate-humanize 处理下面这段：

This controlled comparison supports the stronger claim that language is not just a useful extra feature...

保存到：C:/path/to/output.docx
```

agent 按 `SKILL.md` 执行：

1. 创建带 `Original` 章节的输入 docx。
2. 运行 `translate_humanize_deepl.py` 生成翻译链（如无 DeepL key，则用 agent 自身 LLM 生成）。
3. 从输出 docx 读取 `Translated English`。
4. 使用 `paper-spine-humanize` heavy-tier 原则写 `Humanized English`。
5. 对照原文修正术语漂移，写 `Humanized English (Corrected)`。
6. 保存最终 docx，并在聊天中展示 `Original → Humanized → Corrected` 对照表。

---

## 输出 docx

辅助脚本首先生成包含前 6 个版本的 docx：

1. **Original** —— 原文 unchanged。
2. **Japanese** —— 中间翻译。
3. **Korean** —— 中间翻译。
4. **Spanish** —— 中间翻译。
5. **Portuguese** —— 中间翻译。
6. **Translated English** —— 完整 EN → JA → KO → ES → PT → EN 往返翻译链结果。

assistant 随后补充最后 2 个版本，最终 docx 共 8 个版本：

7. **Humanized English** —— assistant 手写的 heavy-tier 改写。
8. **Humanized English (Corrected)** —— 术语修正后的最终版本，建议默认使用。

对照表只在 agent 聊天回复中展示，不写入 docx。

---

## 辅助脚本用法

如需直接运行脚本：

```bash
export DEEPL_API_KEY="your-deepl-key"
python translate_humanize_deepl.py -i examples/sample_input.docx -o output.docx
```

脚本从输入 docx 读取 `Original`，输出包含 6 个版本的翻译链 docx（`Original` + 4 种中间语言 + `Translated English`）。

脚本**不会**生成 `Humanized English` 和 `Humanized English (Corrected)`，这两个版本由 agent 后续补充。

没有 DeepL key 时，agent 应跳过脚本，使用自身 LLM 生成翻译链。

---

## 人类化原则

翻译后，assistant 会改写文本以降低常见 AI 信号：

| 检测维度 | AI 特征 | 修正方向 |
|---|---|---|
| 句长 | 统一 15–25 词 | 长短句交替，3–8 词短句 + 25–45 词长句 |
| 段落结构 | Claim → explain → example → summary | 使用因果链、突兀结尾、对比判断 |
| 连接词 | 频繁使用 `furthermore/moreover/therefore/in conclusion` | 每 1000 词不超过 4 个 |
| 术语上下文 | 总是同一标准表达 | 偶尔使用精确学术同义词 |

同时减少或删除模板短语，如 `"This study demonstrates..."`、`"These findings suggest..."`、`"It is important to note..."`。

---

## 注意事项

- `Humanized English` 必须是对 `Translated English` 的单独改写，不能直接复制。
- `Humanized English (Corrected)` 由 assistant 手写，不能依赖脚本自动替换表；其目的是恢复 humanize 过程中损失的学术准确性。
- 聊天回复中只展示 `Original → Humanized English → Corrected` 三版对照，重点说明哪些词变得不够学术或准确。
- 默认使用 `Humanized English (Corrected)`，除非更喜欢原始回译措辞。

---

## 许可证

MIT
