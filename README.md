# NAME-Bench: Non-Assertive Mention Evaluation Benchmark

Data release for the paper **Not Every Mention Is a Claim: Rethinking Object Hallucination Detection in Vision-Language Models**.

**Authors:** Hsin-Ling Hsu\*, Neeraja Kirtane\*, Kuan-Hao Huang

---

Object mentions from the image captions of five vision-language models, each labelled as **concrete** (the caption
asserts the object and it is in the image), **hallucinated** (the caption asserts the object and it is not in the
image) or **non-assertive** (the caption names the object without asserting that it is present). Non-assertive
mentions are further divided into four categories:

| Category | Description | Example |
|---|---|---|
| hedged | the model marks its own uncertainty | "possibly a remote control" |
| depictive | the object appears only as a representation | "a painting of a ship" |
| purpose | the object is the goal of an action or plan (purpose clause or intensional verb) | "a bench for people to sit on", "waiting for the train" |
| negation | the model names the object to deny it | "no cars are visible" |

## Contents

```
benchmark/eval_sets/<model>.csv            evaluation set: concrete, hallucinated and non-assertive mentions
benchmark/natural/<model>_captions.csv.gz  fully labelled captions used for natural rates (one row per caption)
benchmark/natural/<model>_mentions.csv.gz  every candidate mention in those captions, with its raw label
benchmark/eval_sets_counts.csv, benchmark/natural_counts.csv
PROMPTS.md                            the three LLM labelling prompts, verbatim
```

Models: LLaVA-1.5-7B, Qwen2.5-VL-7B, Qwen2.5-VL-3B, InternVL3-8B, LLaVA-OneVision-2-8B. Each model captions COCO 2017
images with the prompt "Provide a detailed description of this image." (greedy decoding, at most 128 new tokens).
Images are not included; use `coco_image_id` / `image_url` (COCO 2017 train/val).

## How labels were produced

1. **Candidate mentions:** a regular expression over the 80 COCO categories with a curated list of synonyms and plurals
   finds every category the caption mentions (one candidate per caption and category).
2. **Usage (GPT-4o, caption text only):** for each candidate, whether the caption refers to the category at all and, if so,
   whether the mention is EXISTENTIAL (asserts presence), FUNCTIONAL (non-assertive) or AMBIGUOUS; the judge also returns
   the trigger phrase it based its decision on.
3. **Presence:** whether the category is in the image's COCO instance annotations.
4. **Label:** EXISTENTIAL + present = concrete; EXISTENTIAL + absent = hallucinated; FUNCTIONAL = non-assertive;
   AMBIGUOUS or not a mention = discarded.
5. **Category (GPT-4o, then rules):** GPT-4o assigns one of purpose, intensional, hedging, depictive, generic or other;
   GPT-4o-mini flags compounds (e.g. "hot dog"). A trigger phrase with a negation cue (no, not, none, without, absent,
   absence, lack(s/ing), nor, neither, devoid, n't) becomes **negation** (except depictive); purpose and intensional
   become **purpose**; hedging becomes **hedged**; generic, other and compounds are excluded from the benchmark.
6. The category **orange** is excluded throughout (the usage judge often accepted the colour as the fruit).

## benchmark/eval_sets/<model>.csv

One row per mention. Only mentions whose object word could be located in the caption are included (these are the
mentions evaluated in the paper). For newer models, most non-assertive mentions come from captions that were
pre-screened for likely non-assertive language, so the class balance is **not** the natural one; use `benchmark/natural/` for rates.

| Column | Meaning |
|---|---|
| mention_id | identifier, unique within the release |
| model | captioning model |
| coco_image_id, coco_split, image_url | COCO 2017 image |
| caption | the model's caption |
| concept | COCO category |
| mention, char_start, char_end | the object word and its character span in `caption` (the occurrence the label refers to) |
| trigger_phrase | phrase returned by the usage judge |
| label | concrete / hallucinated / non-assertive |
| category | hedged / depictive / purpose / negation (non-assertive only) |
| object_in_coco_annotations | whether the category is in the image's COCO instance annotations |

| Model | Concrete | Hallucinated | Non-assertive | Hedged | Depictive | Purpose | Negation |
|---|---|---|---|---|---|---|---|
| LLaVA-1.5-7B | 1,975 | 1,974 | 1,594 | 564 | 265 | 630 | 135 |
| Qwen2.5-VL-7B | 1,878 | 1,983 | 1,866 | 958 | 642 | 182 | 84 |
| Qwen2.5-VL-3B | 1,891 | 1,979 | 1,745 | 802 | 516 | 211 | 216 |
| InternVL3-8B | 1,947 | 1,321 | 678 | 198 | 307 | 43 | 130 |
| LLaVA-OneVision-2-8B | 1,919 | 1,433 | 1,314 | 561 | 450 | 172 | 131 |

## benchmark/natural/<model>_*.csv.gz

All candidate mentions in fully labelled captions: 2,000 random COCO images per newer model (captions with at least one
candidate) and all 106,572 LLaVA-1.5 captions. `label` is one of concrete, hallucinated, non-assertive, ambiguous,
not_a_mention or excluded_orange; for non-assertive mentions `category` is one of the four categories, `compound`, or
`uncategorised_<subtype>` (generic / other, excluded from the benchmark). `gpt_usage` is the raw step-1 judgement and
`regex_match` the surface form the regular expression matched (its first occurrence in the caption).

| Model | Captions | Candidate mentions |
|---|---|---|
| LLaVA-1.5-7B | 106,572 | 355,761 |
| Qwen2.5-VL-7B | 1,930 | 4,129 |
| Qwen2.5-VL-3B | 1,937 | 4,207 |
| InternVL3-8B | 1,955 | 4,626 |
| LLaVA-OneVision-2-8B | 1,893 | 3,710 |


