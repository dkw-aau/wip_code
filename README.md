# wip_code

This repository contains code examples for the Web Information Processing Course at Aalborg University.

## NER Hard Cases Demo

The notebook `ner_hard_cases_demo.ipynb` demonstrates how different NER systems (spaCy, Hugging Face Transformers, Flair, and a toy baseline) behave on a small set of deliberately difficult examples, including boundary ambiguity, polysemy, noisy text, rare entities, metonymy, and domain-specific entities.

### Scoring: Exact vs. Relaxed (Overlap)

The notebook evaluates each system under two matching schemes.

**Exact matching** counts a prediction as a true positive only when the predicted span has *exactly* the same character-level start offset, end offset, and entity label as the gold span. Every field must match precisely.

**Relaxed (overlap) matching** counts a prediction as a true positive as long as the predicted span *shares at least one character* with the gold span **and** has the correct entity label. The span boundaries do not need to align.

#### How the scores are computed

For each example, `score_one` iterates over the gold spans and tries to find an unused predicted span that satisfies the matcher. It accumulates:

- **TP** – gold spans matched by a prediction
- **FP** – predicted spans with no matching gold span (`len(pred) − TP`)
- **FN** – gold spans with no matching prediction (`len(gold) − TP`)

These counts are micro-aggregated across all examples, then converted to precision, recall, and F1.

#### How relaxed scores differ from exact scores

| Situation | Exact | Relaxed |
|---|---|---|
| Predicted `"Apple Inc. headquarters"` (ORG), gold `"Apple Inc."` (ORG) | ❌ FP + FN | ✅ TP (spans overlap, label matches) |
| Predicted `"President Barack Obama"` (PER), gold `"Barack Obama"` (PER) | ❌ FP + FN | ✅ TP |
| Predicted `"Apple Inc."` (LOC), gold `"Apple Inc."` (ORG) | ❌ FP + FN | ❌ FP + FN (label mismatch) |

Relaxed scores are always **≥** exact scores, because every exact match is also a relaxed match. The gap between the two scores directly measures how often a model identifies the correct entity type but gets the span boundaries wrong — which is the most common failure mode for the boundary-ambiguity examples in this notebook.

## Random Table Finder (`random-table-finder.html`)

`random-table-finder.html` is a self-contained, single-page web application that fetches and displays a random table from Wikipedia.

### How it works

1. **Fetch a random article** – Clicking *"Find me a wikipedia table"* calls the [MediaWiki API](https://www.mediawiki.org/wiki/API:Random) (`action=query&list=random`) to retrieve a random English Wikipedia article title.
2. **Fetch the article HTML** – The parsed HTML of that article is retrieved via another MediaWiki API call (`action=parse&prop=text`).
3. **Extract tables** – All `<table>` elements with more than one row are extracted. Empty or layout-only tables are skipped.
4. **Display a random table** – One table is chosen at random and rendered in the page. A link to the source Wikipedia article is shown below the table.
5. **Retry logic** – If the selected article contains no qualifying tables, the page automatically tries another article, up to **10 attempts**, before showing an error message.

### Additional details

- Clicking *"Find another table"* repeats the entire process and replaces the currently displayed table.
- Links inside the rendered table are rewritten to absolute Wikipedia URLs and opened in a new tab.
- Images within the table are hidden via CSS to keep the display clean.
- The page requires no build step or external dependencies — open `random-table-finder.html` directly in any modern browser.
