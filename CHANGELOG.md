# Changelog

## v0.1.0 — 2026-11-04

First release.

- **One config JSON in, one owned fragment per widget
  out** — `make --config CONFIG --out DIR` generates the
  four standard small-site widgets as self-contained
  static HTML fragments: inline CSS, inline vanilla JS
  only where needed, no external requests, no account, no
  view metering. The bundled bakery sample's four
  fragments total 8,473 bytes.
- **faq** — semantic `<details>`/`<summary>` items that
  work with JavaScript disabled, plus a FAQPage JSON-LD
  block generated from the SAME item list, so the schema
  can never drift from the visible page; the data block
  is unicode-escaped so a question can never break out
  of it.
- **pricing** — tiers with prices as strings rendered
  exactly as written (no currency math, no reformatting),
  highlight only on config-flagged tiers with no invented
  badge text, and optional CTA links checked structurally
  (https/http/mailto/tel only; `javascript:` refused).
- **countdown** — target must be an ISO 8601 datetime
  with an explicit UTC offset (naive values refused: a
  countdown that guesses a timezone counts to the wrong
  moment); `ended_text` is required; the no-JS fallback
  is a static target-date line; a target already past
  generates the ended variant with a warning (exit 1),
  never a silent countdown to a finished event. The
  fragment embeds the target only — re-rendering is
  byte-identical.
- **proof** — social-proof cards rendered ONLY from the
  entries supplied (inline, or a JSON/CSV entries file):
  verbatim, in input order, with optional dates and
  1–5 star ratings. Nothing is computed — no average, no
  count, no summary claim — ever. Bundled entries are
  labelled SAMPLE throughout.
- **Validation collects every problem** — one run reports
  all of them (the bundled bad config's six, across four
  widgets), exits 2, and writes nothing, not even the
  output folder. Warnings (past countdown, empty widget)
  exit 1 with fragments written.
- **Escaping everywhere** — every user-supplied string is
  HTML-escaped in text and attributes; a `<script>` in a
  FAQ question renders as text on the page and as
  `\u003c` escapes inside the JSON-LD.
- **`--json` on the command** — one JSON object with the
  same facts and exit codes as the text view; errors are
  `{"error": ...}` (validation failures add the full
  `problems` list), argparse usage errors included.
- **`SKILL.md`** at the repo root for AI coding agents:
  the config schema per widget, the output JSON schema,
  exit codes, and the honesty rules (proof rendered never
  computed; visible FAQ == JSON-LD; a past countdown is a
  warning).
- Bundled examples (`examples/`): a full four-widget
  config for a fictional bakery, proof entries in JSON
  and CSV, a six-problem bad config, and a past countdown.
- 48 stdlib-unittest tests: both directions for every
  widget, the FAQ visible/JSON-LD/config three-way
  agreement, the six-problem collection, determinism
  byte-for-byte, the no-external-references fragment
  contract, and the clean-error shape in both modes.
- Pure Python standard library; local only; MIT licence.
