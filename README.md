# translate-humanize

[中文文档](README_CN.md)

An agent skill for reducing AI-detection signals in academic English paragraphs by combining a DeepL round-trip translation chain with assistant-driven `paper-spine-humanize` rewriting and terminology correction.

---

## What this skill does

1. **Translation chain** — the agent runs a helper script that translates the original text through `EN → JA → KO → ES → PT → EN`. This breaks LLM-specific phrasing while preserving meaning.
2. **Assistant humanize** — the agent rewrites the round-trip English using `paper-spine-humanize` heavy-tier principles (shorter sentences, fewer connectors, varied rhythm).
3. **Terminology correction** — the agent fixes words in `Humanized English` that became too loose, informal, or inaccurate, restoring academic precision while respecting the original terminology choices.

The humanization framework is inspired by [PaperSpine](https://github.com/WUBING2023/PaperSpine), specifically the `paper-spine-humanize` tiered writing directives.

If no DeepL API key is available, the agent can generate the translation chain itself using its own multilingual capabilities.

---

## When to use

- A paragraph or section in a dissertation/paper is flagged by an AIGC detector.
- Structural humanization alone is not enough.
- You want a repeatable, section-by-section workflow for lowering AI-detection rate.

---

## When NOT to use

- For content where any paraphrase risk is unacceptable (equations, exact numbers, legal text).
- For very long documents in one shot — split into sections first.
- When the target language is not English.

---

## Files in this skill

```text
translate-humanize/
  SKILL.md                      # Agent-facing protocol
  translate_humanize_deepl.py   # Helper script for the DeepL translation chain
  README.md
  README_CN.md
  LICENSE                       # MIT + PaperSpine attribution
  requirements.txt              # python-docx
  .env.example                  # Example environment variables
  examples/
    sample_input.docx           # Example input docx
```

---

## Installation for agents

### kimi-code

Place the skill folder where kimi-code can discover it, for example:

```text
<project-root>/.kimi-code/skills/translate-humanize/SKILL.md
```

or in the user skills directory:

```text
~/.kimi-code/skills/translate-humanize/SKILL.md
```

### Claude Code

Copy the skill folder to:

```text
~/.claude/skills/translate-humanize/
```

### Codex

Copy the skill folder to:

```text
~/.codex/skills/translate-humanize/
```

### OpenClaw

Copy the skill folder to:

```text
~/.openclaw/skills/translate-humanize/
```

---

## Requirements

- `python-docx` for the helper script.
- `DEEPL_API_KEY` environment variable when using the DeepL translation chain.

Install the single dependency if needed:

```bash
pip install python-docx
```

---

## How the agent uses this skill

The user provides an English paragraph and an output path:

```text
Use translate-humanize on:

This controlled comparison supports the stronger claim that language is not just a useful extra feature...

Save to: C:/path/to/output.docx
```

The agent then follows `SKILL.md`:

1. Create an input docx with an `Original` section.
2. Run `translate_humanize_deepl.py` to generate the translation chain (or use the agent's own LLM if no DeepL key is available).
3. Read `Translated English` from the output docx.
4. Write `Humanized English` using `paper-spine-humanize` heavy-tier principles.
5. Write `Humanized English (Corrected)` by fixing terminology drift against the original.
6. Save the final docx and present a chat-side comparison of `Original → Humanized → Corrected`.

---

## Output docx

The helper script produces a docx with the first 6 versions:

1. **Original** — input text unchanged.
2. **Japanese** — intermediate translation.
3. **Korean** — intermediate translation.
4. **Spanish** — intermediate translation.
5. **Portuguese** — intermediate translation.
6. **Translated English** — result of the full EN → JA → KO → ES → PT → EN round-trip chain.

The assistant then adds the final 2 versions, so the completed docx contains 8 versions in total:

7. **Humanized English** — assistant-written heavy-tier rewrite.
8. **Humanized English (Corrected)** — terminology-corrected final version. Recommended default.

The comparison table is delivered in the agent's chat response, not inside the docx.

---

## Helper script usage

If running the script directly:

```bash
export DEEPL_API_KEY="your-deepl-key"
python translate_humanize_deepl.py -i examples/sample_input.docx -o output.docx
```

The script reads `Original` from the input docx and writes a docx containing the 6-version translation chain (`Original` + 4 intermediate languages + `Translated English`).

It does **not** generate `Humanized English` or `Humanized English (Corrected)`. Those are added by the agent afterwards.

Without a DeepL key, the agent should skip the script and generate the translation chain using its own LLM.

---

## Humanization principles

After translation, the assistant rewrites the text to reduce common AI signals:

| Detection dimension | AI pattern | Fix |
|---|---|---|
| Sentence length | Uniform 15–25 words | Mix short (3–8 words) and long (25–45 words) sentences |
| Paragraph structure | Claim → explain → example → summary | Use causal chain, abrupt end, compare-judge |
| Connectors | Frequent `furthermore/moreover/therefore/in conclusion` | Reduce to ≤4 per 1,000 words |
| Term context | Always the same standard phrasing | Use precise academic synonyms occasionally |

Templated phrases such as `"This study demonstrates..."`, `"These findings suggest..."`, `"It is important to note..."` are also reduced or removed.

---

## Notes

- `Humanized English` must be a distinct rewrite of `Translated English`, not a copy.
- `Humanized English (Corrected)` is written by the assistant, not by a script-based replacement table. Its purpose is to restore academic precision that may have been lost during humanization.
- The chat response shows only the three-way comparison `Original → Humanized English → Corrected`, highlighting where wording became less academic or less accurate.
- Use `Humanized English (Corrected)` as the default replacement text unless you prefer the raw round-trip phrasing.

---

## License

MIT
