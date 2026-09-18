# eclore.se DE/FI internal-link localization — final report

## Background

Shopify does not rewrite internal URLs when a page is translated. As a result, German
(`de`) and Finnish (`fi`) translations of Article, Product, and Collection `body_html`
content on eclore.se contained internal links still pointing at the untranslated Swedish
pages (e.g. `https://www.eclore.se/collections/foo` instead of
`https://www.eclore.se/de/collections/foo` or `.../fi/collections/foo`), sending DE/FI
visitors to Swedish-only content. This project found and fixed every such link across
the whole store, without touching any other text, prices, or metadata.

## Scope and result

- **Resources scanned:** all 153 Articles, 480 Products, and 65 Collections on the
  store (698 resources total), across `sv` (source), `de`, and `fi` locales.
- **Resources needing a fix:** 470 unique resources (181 Articles, 379 Products,
  48 Collections).
- **Individual link fixes applied:** 2,309 (1,153 in `de` content, 1,156 in `fi`
  content), across 608 `translationsRegister` field-level rows submitted in 134 batched
  GraphQL mutations.
- **Final verification (full re-export + compare, 2026-09-18):** **937/937** known
  link-fix rows confirmed live and correct. Zero remaining corruption, zero remaining
  unfixed rows. See `HANDOFF.md` for the detailed audit-pass methodology and findings,
  including the resolution of 110 initial false-positive "corrupted" flags (unrelated
  prose edits, links unaffected) and 19 resources that needed a clean resubmission after
  the audit caught they hadn't actually landed on first submission.

## Files in this delivery

- `eclore_de_fi_link_fix_log.csv` — every individual link fix: resourceId, resourceType,
  field, locale, old URL, new URL. 2,309 rows.
- `eclore_de_fi_missing_or_stale.csv` — informational only, not part of this project's
  scope: 333 rows where Shopify's own `outdated` flag shows a `de`/`fi` translation
  (mostly `meta_title`/`meta_description`, not `body_html`) is stale relative to a since-
  changed Swedish source. Flagged for a future translation-refresh project.
- `HANDOFF.md` — full working log: every batch, every quirk found, the corruption-
  mitigation technique used throughout (with zero corruption incidents across the
  project), and the full audit-pass writeup.

## Pre-existing data quirks found (not fixed — submitted/left verbatim per standing
policy, flagged here for a possible follow-up cleanup pass)

These are **not** link-localization bugs; they are pre-existing issues in the Swedish
source content or in earlier DE/FI translations that predate this project. Fixing them
was explicitly out of scope (translation content editing vs. link localization), so they
were left untouched and are listed here for visibility:

1. **Unprefixed bare-domain links** (`https://eclore.se` / `https://www.eclore.se` /
   `https://www.eclore.se/`, no locale segment) — the single most common quirk,
   recurring across dozens of articles, usually in a closing call-to-action link or a
   brand-name mention mid-article.
2. **Unprefixed `/pages/...` links** (e.g. `https://www.eclore.se/pages/hudvard-for-
   kanslig-hud`) — a large, very common cluster, distinct from the `/collections/`,
   `/products/`, `/blogs/` link patterns that this project's scan/fix covered.
3. **Batch 106 (Article 1002727178570, "comprehensive body lotion guide"):** a severe,
   isolated case of cross-locale link contamination — the DE translation contained
   several `/fi/`-prefixed links (should be `/de/`), and the FI translation contained
   roughly a dozen `/de/`-prefixed links (should be `/fi/`) plus one malformed domain
   (`https://www.eclore.fi/pages/har`, wrong TLD structure). This article needs a
   dedicated manual re-check, not just a spot-check.
4. **Article 1004220711242 (dry/itchy scalp guide), FI content:** brand-name typo
   "écorelta" (missing an "l", should read "éclorelta").
5. **Article 1000131133770 (men's skincare basics guide), FI content:** one link uses
   `/en/collections/for-man` (English prefix) instead of `/fi/collections/for-man`.
6. Assorted smaller issues noted in `HANDOFF.md`'s per-batch log: an HTML comment block
   wrapping table rows in one FI article body (batch 112), a duplicated-word artifact in
   one FI article (batch 077), an `http://` (not `https://`) unprefixed link used as
   both href and visible anchor text in a couple of articles (batches 088, 092), and a
   cosmetic XML-namespace typo (`w3099` instead of `w3.org/1999`) in one FI article's
   `<p>` tag (batch 088).

## Corruption incidents

**None.** Every one of the 134 submission batches across this entire project returned
`userErrors: []`. The consistent use of an `ensure_ascii`-escaped payload-split-and-
verbatim-copy technique (documented in `HANDOFF.md`) eliminated the risk of silent
transliteration corruption (e.g. "ä" → "ae") that had affected two early batches in a
prior phase of this project (`rembatch_046`, `rembatch_047`), both caught and fixed
before this window began.

## Outstanding / recommended follow-up (not part of this project's scope)

- A dedicated pass to localize the recurring unprefixed `eclore.se` / `/pages/...` links
  described above (arguably the same underlying bug class as this project's `/collections/`
  `/products/` `/blogs/` fix, just a link-path prefix this scan didn't originally target).
- A manual content re-check of Article 1002727178570 (batch 106) given the severity of
  its cross-locale contamination.
- A translation-refresh pass for the 333 `outdated`-flagged `de`/`fi` fields in
  `eclore_de_fi_missing_or_stale.csv` (mostly SEO metadata, not `body_html`).
