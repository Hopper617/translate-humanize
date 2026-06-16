# translate-humanize

## Description

A reusable skill that combines two AI-detection reduction techniques in a **translation-first** order, with both humanization and terminology correction performed by the assistant:

1. **Translation round-trip** — the script translates the original text through Japanese → Korean → Spanish → Portuguese → English. The chain of back-translation disrupts model-specific phrasing while preserving the meaning.
2. **paper-spine-humanize (heavy-tier, assistant-written)** — the assistant rewrites the round-trip English output to break AI-detectable statistical patterns (uniform sentence length, repetitive connectors, templated paragraph structures, etc.).
3. **Terminology correction (assistant-written)** — the assistant fixes round-trip terminology drift while respecting the terminology choices in the original text.

The humanization framework is inspired by [PaperSpine](https://github.com/WUBING2023/PaperSpine), specifically the `paper-spine-humanize` tiered writing directives.

Use this skill when you have a paragraph, section, or short document that an AIGC detector flags, and you want a final English version that reads naturally but no longer matches typical LLM output patterns.

## When to use

- A paragraph or section in your dissertation/paper is highlighted by Turnitin / AIGC detector.
- You have already done structural humanization but the detector still flags the text.
- You want a quick, repeatable workflow for lowering AI-detection rate on isolated blocks of text.

## When NOT to use

- For factual content where precision is critical and any paraphrase risk is unacceptable (equations, exact numbers, legal text).
- For very long documents in one shot; split into sections first.
- When the target language is not English (this skill always returns English).

## Input

Provide the text you want to process. Best results for:

- Single paragraphs (50–300 words)
- Short sections (up to ~1,000 words)
- Academic/technical English prose

## Output

The helper script produces a docx file containing, in order:

1. **Original** — the input text unchanged
2. **Japanese** — Original translated to Japanese
3. **Korean** — Japanese translated to Korean
4. **Spanish** — Korean translated to Spanish
5. **Portuguese** — Spanish translated to Portuguese
6. **Translated English** — Portuguese translated back to English (the full EN → JA → KO → ES → PT → EN round-trip result)

The assistant then adds two more versions, so the final docx contains 8 versions in total:

7. **Humanized English** — assistant-written `paper-spine-humanize` heavy-tier rewrite of Translated English
8. **Humanized English (Corrected)** — assistant-corrected version of Humanized English, with terminology drift fixed while respecting the original terminology choices

In the chat response, the assistant presents a three-way comparison of **Original → Humanized English → Humanized English (Corrected)**, focusing on where the humanized version became less academic or less accurate and how the corrected version fixes it.

The file is saved at the path you specify. If the path is occupied, the skill saves to a fallback filename and tells you to rename it.

### Why a corrected version?

`Humanized English` is rewritten to sound natural and reduce AI-detectable patterns. In doing so, it can unintentionally replace precise academic terms with looser, less formal, or less accurate words. The purpose of `Humanized English (Corrected)` is to restore academic precision and fix terminology drift while keeping the natural rhythm of the humanized version.

Unlike a script-based correction, the assistant judges context: if the original uses `visual-linguistic pre-training`, the corrected version keeps it; if the original uses `vision-language pre-training`, that is kept instead. Example fixes:

| Round-trip drift | Corrected term |
|---|---|
| `constructive generalization` | `compositional generalisation` |
| `learning space` | `training manifold` |
| `movement fragments` | `action chunks` |
| `fine-motor benchmark tests` | `fine-grained manipulation benchmarks` |
| `biosensor data` | `raw sensory data` |

The assistant applies these corrections manually, guided by the original text and the field-specific vocabulary.

## Workflow

```
Original text
    ↓
[Script] Translate to Japanese
    ↓
[Script] Translate to Korean
    ↓
[Script] Translate to Spanish
    ↓
[Script] Translate to Portuguese
    ↓
[Script] Translate back to English
    ↓
Translated English  (script output ends here)
    ↓
[Assistant] Apply `paper-spine-humanize` heavy-tier rewrite
    ↓
Humanized English
    ↓
[Assistant] Apply terminology correction
    ↓
Humanized English (Corrected)
    ↓
[Assistant] Generate comparison and present it in the chat response
    ↓
Save docx with all 8 versions
```

## Humanization principles applied

After translation, rewrite the round-trip English to reduce these AI signals:

| Detection dimension | AI pattern | Fix |
|---|---|---|
| D1 — Sentence length | Single peak around 15–25 words | Mix very short (3–8 words) and long (25–45 words) sentences |
| D2 — Paragraph structure | Claim → explain → example → summary | Use question-first, compare-judge, causal chain, abrupt end |
| D3 — Information density | Uniform 65–75% density | Vary density between core and transition sentences |
| D4 — Connectors | Frequent `furthermore/moreover/additionally/therefore/in conclusion` | Reduce to ≤4 per 1,000 words; use natural transitions |
| D5 — Term context | Always the same standard phrasing | Use precise academic synonyms occasionally |

Also remove templated phrases such as:

- "This study demonstrates..."
- "These findings suggest..."
- "It is important to note..."
- "A different thread has risen..."
- "The results indicate..."

And add first-person academic narration where natural: "we observed", "we found", "our data showed", "we chose".

## Example

### Input

```text
VLAs offer a single end-to-end framework for robotic manipulation, fusing vision, language, and action. Pretraining at scale, followed by task-specific fine-tuning, gives them real promise for generalising across tasks and robot bodies. The limits are just as real. High-quality, diverse robotic datasets are scarce compared with web-scale corpora. Pretraining biases can cripple performance when tasks drift outside the training distribution. And most VLAs remain vision-centric; they stumble in contact-rich manipulation where force or tactile sensing would help. One genuine strength stands out: compositional generalisation. The model recombines learned skills according to language instructions to produce novel task combinations. How robust and broad that ability is, however, remains unresolved.
```

### Translated English (after round-trip)

```text
VLAs combine vision, language, and action into a single end-to-end pipeline. It sounds clean. You do large-scale pre-training, add task-specific fine-tuning, and hope the same model transfers across tasks and even across robot bodies. But reality is messier. Robotic datasets are small compared with web-scale text corpora, and the available data is rarely diverse enough. As soon as the task drifts outside the training distribution, pre-training biases hit hard. Most VLAs remain vision-centric, so contact-rich manipulation—where force or tactile feedback matters—still trips them up. The capability that truly impresses is compositional generalisation: the model can recombine learned skills under new language instructions to create task combinations it has never seen. Yet whether that ability is robust or merely narrow cleverness remains an open question.
```

### Humanized English

```text
VLAs bundle vision, language, and action into one end-to-end pipeline. That sounds clean. Scale up pretraining, add task-specific fine-tuning, and the hope is that the same model will transfer across tasks and even across robot bodies. The reality is messier. Robotic datasets are tiny next to web-scale text corpora, and the data that does exist is rarely diverse enough. Once the task drifts outside the training distribution, pretraining biases bite hard. Vision dominates most VLAs, so contact-rich manipulation—where force or tactile feedback matters—still trips them up. The one capability that genuinely impresses is compositional generalisation: the model can recombine learned skills under new language instructions to create task combinations it never saw. Whether that ability is robust or just narrow cleverness is still an open question.
```

### Humanized English (Corrected)

Same as Humanized English, but with any drifted technical terms restored. In this example no corrections were needed, so the corrected version is identical to Humanized English.

## Usage command

When the skill is active, invoke it by providing the text and the desired output path:

```
Use translate-humanize on:
[your text here]

Save to: C:/path/to/output.docx
```

If no path is given, the skill saves to `translate_humanize_output.docx` in the current working directory.

## Notes

- **The helper script only produces the translation chain.** It does not write `Humanized English` or `Humanized English (Corrected)`.
- **Humanized English is written by the assistant, not auto-generated by the script.** For best results the assistant must rewrite Translated English using `paper-spine-humanize` heavy-tier principles.
- **Humanized English (Corrected) is also written by the assistant**, not by a script-based replacement table. Its purpose is to restore academic precision and fix words that became too loose or inaccurate during humanization, while preserving the original author's terminology choices.
- **Humanized English must be a distinct rewrite of Translated English.** Do not copy Translated English into Humanized English.
- Always review the Translated English output. Back-translation can occasionally soften nuances or shift emphasis.
- Check **Humanized English (Corrected)** for technical-term accuracy. Use it as the default replacement text unless you prefer the raw round-trip phrasing.
- If Humanized English drifts too far from your intent, use the Translated English version instead; it already carries some AI-rate reduction benefit.
- For long documents, apply this skill section-by-section rather than all at once.
