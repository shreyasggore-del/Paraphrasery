---
name: paraphrase-style-doc1-doc2
description: "Rewrite input text in the 'doc1-to-doc2' paraphrase style — a synonym-substitution / light-restructuring paraphrase pattern used to generate adversarial training samples for text-detection (plagiarism/AI-detection) classifiers. Trigger this whenever the user asks to 'paraphrase like doc2', 'convert doc1 style to doc2 style', generate paraphrased/spun variants of source text for a detector's training set, or create adversarial paraphrase samples. This is for training/testing detection software, not for evading academic integrity checks on the user's own submitted work — do not use this skill to help disguise text as original for submission."
---

# Paraphrase style: doc1 → doc2 (synonym-substitution spin)

This skill replicates a specific, low-level paraphrasing style observed by diffing two sample
documents (doc1 = original, doc2 = paraphrased). The purpose is to generate labeled
**original/paraphrased pairs** for training or evaluating text-detection classifiers — i.e. the
output needs to be a *faithful, precise reproduction of a common "spinning" pattern*, not a
well-written independent rewrite.

**Do not use this skill to help a user disguise their own or someone else's text as original for
submission to a plagiarism or AI-detection checker.** If that intent surfaces, stop and decline.
Generating labeled adversarial samples for a detector you are building is fine; laundering a
specific document to beat a checker is not.

## Core transformation rules (in priority order)

1. **Sentence-level 1:1 mapping.** Never merge, split, drop, or reorder sentences relative to the
   source (with one exception: rule 6). Each output sentence should correspond to exactly one
   input sentence and preserve its position in the argument.

2. **Synonym / near-synonym substitution at the word and short-phrase level.** Swap content words
   (nouns, verbs, adjectives, adverbs) for close synonyms. Function words and structure mostly stay.
   Examples from the reference pair:
   - "precious" → "valuable"
   - "essence of life" (kept — some phrases survive untouched to preserve meaning-anchors)
   - "Every living organism, from the smallest microorganism to the largest mammal, depends on
     water" → "All animals, including the smallest animals, require water" (note: this
     substitution is *imprecise* — "organism" narrowed to "animals" — reproduce this kind of
     minor semantic drift, don't correct it)
   - "regulate body temperature" → "cools the body"
   - "transports nutrients and oxygen to cells" → "carries nutrients and oxygen to the cells"
   - "reduce concentration, physical performance" → "impair concentration, physical performance"

3. **Light clause restructuring**, not full rewrites. Typical moves:
   - Split a compound clause into two short clauses, or vice versa.
   - Move a trailing clause to the front of the sentence, or front-load a subordinate clause.
   - Convert active ↔ passive occasionally ("Rivers have served as centers of trade..." →
     "Rivers have been used as trading, transport, farming and cultural hubs").
   - Preserve overall sentence length (± a few words) — don't compress or expand paragraphs.

4. **Deliberately mild grammar/precision degradation.** The output should read as competent but
   not polished — a notch below the source in fluency. Introduce the kinds of small errors a
   thesaurus-driven spin tool produces:
   - Slightly wrong word choice ("unsuitable" → "impenetrable", "essential for all organs" instead
     of "every organ depends on it")
   - Redundant or flattened phrasing ("makes up about 71%... and is present in..." vs. the
     original's more precise "covering approximately 71%... water is found in...")
   - Occasional loss of nuance/precision (numbers, qualifiers) — e.g. "mild dehydration" becoming
     an oddly specific "dehydration as light as 1.0%"
   Do not introduce grammar errors so severe they'd read as broken; keep it fluent-but-flawed.

5. **Strip paragraph breaks.** Regardless of the source's paragraph structure, output the entire
   result as continuous text with no paragraph breaks (this matches the observed doc2 pattern). If
   the calling context needs paragraph breaks preserved instead, the user must say so explicitly —
   default to stripping them.

6. **Truncate the tail slightly.** Drop the final sentence or two of the source, especially closing
   remarks that summarize/moralize (e.g. cultural/symbolic closing statements). This reproduces the
   observed doc2 behavior of cutting off just before the source's final concluding sentences.

7. **Net word count should increase slightly** (roughly +3% to +6% over the source), despite
   content being truncated at the tail — the substitution/restructuring pattern tends to pad
   individual sentences even as the ending is cut short.

## Generation 2+: recursive paraphrase of already-paraphrased text

When this style is applied to text that is *itself already a doc1→doc2-style paraphrase* (rather
than an original source), the pattern intensifies rather than repeating identically. Observed
(doc3, itself a paraphrase → doc4, a paraphrase of that paraphrase):

- **Clause restructuring becomes more aggressive.** Beyond word-level synonym swaps, whole clauses
  get reordered or recast — e.g. a simple negative conditional ("Without fire, X would have
  progressed much slower") becomes a more elaborate counterfactual ("If man had not learned how to
  make fire, then his civilization would have made little headway"). Temporal/adverbial phrases get
  fronted more often ("Thousands of years ago early humans...").
- **Occasional genuinely incoherent clause merges appear**, not just imprecise word choice — e.g. a
  fragment like "Large wildfires spread fire, engineers use fire for welding..." that doesn't
  parse logically. Reproduce this as a rare (not constant) defect — on the order of one such
  glitch per document, not per sentence.
- **Notation/unit substitution**: spelled-out scientific terms may convert to symbolic form (e.g.
  "carbon dioxide, methane and nitrogen oxides" → "CO₂, CH₄ and N₂O"). Treat this as another
  substitution class alongside plain synonyms.
- **Nominalization creep**: verb phrases get replaced with clunkier noun-heavy constructions —
  e.g. "reducing dependence on fossil fuel combustion" → "minimization of reliance on the
  combustion of fossil fuels." This makes the text sound more formal but less fluent; use it as
  one more degradation mode, not on every sentence.
- **Misplaced or redundant discourse connectives.** Insert "however," "therefore," etc.
  mid-clause with slightly awkward comma placement, and occasionally double them up in close
  proximity — e.g. "overburning or improper fire management can, however, lead to..." followed
  soon after by another unrelated "however." Sprinkle sparingly; overuse reads as noise rather
  than a spin artifact.
- **Appositive → parenthetical conversion**: recast "X, also known as Y," structures as "Y (X)" —
  e.g. "Forest fires, also known as wildfires," → "Wildfires (forest fires)."
- **Word count keeps climbing** with each generation, and the *growth rate itself increases* with
  generation depth — roughly +3-6% for generation 2, rising toward +8-9% by generation 3 in
  observed samples (969→1054 words, doc5→doc6). Don't cap it at the generation-2 rate if asked to
  run further generations. Tail-truncation does **not** necessarily recur each generation — a
  later generation may reproduce the prior generation's ending exactly rather than cutting further.
- When asked to run this skill multiple times / for multiple "generations" on the same text, treat
  each pass as compounding: increase restructuring aggressiveness, incoherence-glitch likelihood,
  nominalization frequency, and word-count inflation slightly with each successive generation,
  rather than resampling the same intensity every time.

## Workflow

1. Read the source text and split it into sentences (preserve original order/index).
2. Apply rules 2–4 sentence by sentence. Keep a mental (or literal, if scripting) map of
   original → paraphrased sentence so pairs stay aligned for labeling.
3. Concatenate all sentences into one continuous block (rule 5).
4. Drop the last 1–2 sentences of the source from the output (rule 6).
5. Sanity-check word count is within the target band (rule 7). Adjust phrasing slightly if not.
6. If producing a labeled dataset, emit `{"original": ..., "paraphrased": ..., "sentence_map": [...] }`
   style pairs rather than just prose, so the pair can be fed directly into detector training/eval.

## Notes for dataset generation

- Precision matters more than variety here: the point is to reproduce *this specific* spin
  pattern consistently across many source texts, not to maximize paraphrase creativity. If asked
  to generate many samples, keep applying the same rule set rather than drifting toward a more
  fluent, independent-rewrite style.
- If the user wants a *different* paraphrase intensity/style (e.g. deeper syntactic rewriting,
  or higher-fluency LLM-style paraphrase) for contrast in the detector's training set, treat that
  as a separate style profile — don't blend it into this one silently.
