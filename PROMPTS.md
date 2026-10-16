# Labelling prompts

All calls use temperature 0 and JSON output. The judges see only the caption text, never the image or its annotations.
`FUNCTIONAL` is the internal label for a non-assertive mention; step 2 uses an earlier five-way scheme from which the four
categories are derived by the rules in README.md.

## Step 1: usage (GPT-4o, six captions per call)

System:
```
You are a precise linguistic analyst. Output ONLY valid JSON -- no markdown, no text outside the JSON.
```

User:
```
You will analyze {N} SEPARATE captions below. Each has its own caption index and its own list of target concepts -- treat each one completely independently; do not let one caption's content influence another's classification.

For EACH target concept in EACH caption, analyze how that concept is used in ITS OWN caption.

For each concept:
STEP 1 -- Mention check: does the caption actually refer to this concept (including synonyms)?  If not, concept_mentioned = false.

STEP 2 -- If mentioned, classify the mention TYPE (based on wording only, NOT on whether the object is really in any image):

  EXISTENTIAL -- the caption asserts this concept EXISTS as a physical object/entity in the depicted scene.
     e.g. "a person is standing", "there are two cars parked", "a cat sits on the mat".

  FUNCTIONAL  -- the concept appears ONLY in a functional, purposive, relational, or hypothetical role, WITHOUT asserting it is physically present in this scene.
     e.g. "a bridge built for PEOPLE to cross"  (people = purpose)
          "a stool used for SITTING"            (no person asserted)
          "a road where CARS usually drive"     (no specific car asserted)
          "a rack designed to hold BOTTLES"     (no bottle asserted)
          "large enough to fit a TRUCK"         (hypothetical capacity)
          "no PEOPLE are visible"               (asserted absence)

  AMBIGUOUS   -- genuinely unclear, or used both ways.

Captions to analyze:

--- Caption index 0 ---
Caption: "{caption}"
Target concepts for this caption:
    - "{concept}"
...

Return ONLY this JSON object, with one entry in "captions" per caption index above (do not skip any index, even if a concept is not mentioned):
{
  "captions": [
    {
      "index": <caption index integer>,
      "results": [
        {
          "concept": "<one of that caption's target concepts>",
          "concept_mentioned": true | false,
          "trigger_phrase": "exact substring from caption, or null",
          "mention_type": "EXISTENTIAL" | "FUNCTIONAL" | "AMBIGUOUS" | null,
          "reasoning": "3-6 words max, e.g. 'purpose clause, no object asserted'",
          "confidence": "HIGH" | "MEDIUM" | "LOW"
        }
      ]
    }
  ]
}
```

## Step 2: category of a non-assertive mention (GPT-4o)

System (steps 2 and 3):
```
You are a precise linguistic annotator. Output ONLY valid JSON.
```

User:
```
Classify the linguistic mechanism by which the following object mention avoids asserting the object's existence in an image.

Full caption: "{caption}"
Trigger span: "{trigger_phrase}"
Concept word: {concept}

Categories:
- purpose: The object is mentioned as the intended user/beneficiary of a purpose clause (e.g. 'a cup for holding toothbrushes', 'a bench for people to sit on'). The construction explains what an anchor object is FOR.
- intensional: The object appears inside a non-veridical/intensional context governed by a verb like want, wait for, hope, look for, expect (e.g. 'waiting for the train', 'looking for a taxi'). The mention does not entail the object's presence.
- hedging: The mention is marked with epistemic uncertainty (possibly, likely, maybe, seems, appears) about whether the object is present (e.g. 'possibly a bird', 'what appears to be a dog').
- depictive: The object is mentioned as depicted content (a sign, picture, logo, text, or symbol) rather than a real physical instance (e.g. 'a sign that says no dogs', 'a picture of a cat').
- generic: The object is referred to as a kind or in a generic/habitual statement rather than a specific instance in the scene (e.g. 'a road where cars usually drive', 'a place popular with tourists', 'no people are visible').

If the span could fit multiple categories, pick the SINGLE best/primary one. If none fit well, use "other".

Output JSON: {"subtype": "purpose|intensional|hedging|depictive|generic|other", "confidence": "high|medium|low", "reason": "one short phrase"}
```

## Step 3: compound check (GPT-4o-mini)

User:
```
In the phrase below, a word or phrase matching the concept "{concept}" was matched. Determine whether this match is actually part of a LARGER compound expression whose overall meaning is different from "{concept}" alone (e.g., "hot dog" means a food item, not "dog"; "toilet bowl" means a fixture, not a "bowl"; "orange juice" is a drink, not the fruit "orange").

Phrase: "{trigger_phrase}"
Matched concept: {concept}

If the match is part of such a compound (the combined phrase has a distinct, different meaning from the bare concept), identify the FULL compound expression.
If the match is NOT part of a compound (concept is used in its ordinary/standalone sense, possibly with an ordinary adjective like "large orange cat" where orange is a colour, or "red car"), say so.

Output JSON: {"is_compound": true/false, "compound_phrase": "the full compound or null", "compound_meaning": "brief gloss of what the compound refers to, or null"}
```
