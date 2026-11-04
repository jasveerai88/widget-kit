---
name: widget-kit
description: Generate the four standard small-site widgets — FAQ (semantic details/summary HTML plus FAQPage JSON-LD built from the SAME item list), pricing (tiers with verbatim string prices, highlight only when flagged, CTA links scheme-checked), countdown (ISO 8601 target WITH an explicit UTC offset required; ended_text required; a target already past is a WARNING, exit 1, and the fragment shows its ended text) and social proof (cards rendered ONLY from entries the owner supplies, inline or from a JSON/CSV entries_file — nothing is computed about them, no averages, no counts, no invented claims) — as one self-contained static HTML fragment per widget from one config JSON. `make --config CONFIG --out DIR` validates the WHOLE config first, collecting EVERY problem into one exit-2 error with NOTHING written (output folder not even created); warnings (exit 1, fragments written) are past countdowns and empty widgets. Every fragment is standalone: inline CSS, inline JS only where needed, no external requests — the only URLs are the configured pricing CTA hrefs plus the schema.org context identifier inside the FAQ JSON-LD (a vocabulary name, never fetched). All user strings are HTML-escaped; the JSON-LD is unicode-escaped so a question can never break out of the data block. Output is deterministic — re-rendering the same config is byte-identical. Exit 1 is a warning state, not a failure.
---

# widget-kit — agent skill

## Purpose

Hosted widget services rent display volume: Elfsight
meters views shared across all sites on a plan (reviewers
report one client eating all the credits, reviews
deactivated on paid plans), Fomo charges $25–149/mo by
notification volume plus white-label fees. The four
standard widgets — FAQ, pricing, countdown, social proof —
are markup, and markup is a file. This tool writes the
files from one config JSON: one paste-ready, self-contained
fragment per widget, local only, pure Python standard
library, nothing metering anything.

**The honesty core of this tool:** proof is *rendered*,
never *computed*, and FAQ schema is *the same data* as the
visible page. The proof fragment shows only the entries in
the config/file, verbatim, in order — no average rating,
no entry count, no summary claim is ever generated. The
FAQ's JSON-LD is built from the identical item list as the
visible `<details>` block, so the two can never drift.

## When to use / not use

Use when:
- A small site needs any of the four widgets without a
  hosted subscription: write a config, `make`, paste the
  fragments into the pages.
- A pipeline needs a gate: exit 0 = ship, exit 1 = ship
  with warnings worth reading (a countdown already over,
  an empty widget), exit 2 = do not ship — the error lists
  every config problem at once.
- An agent assembles widgets from data it already holds (a
  client's prices, real testimonials on file) and needs
  them validated and escaped before they reach a page.

Do NOT use when:
- You need live data — auto-pulled Google reviews, real
  order popups, inventory-aware countdowns. This tool is
  static by design; re-run it with fresh exports instead.
- You are tempted to invent proof. There is no config
  shape that makes the tool fabricate entries, ratings or
  totals — and hand-editing a fragment to add an entry the
  owner doesn't have is fraud, not formatting.
- You need forms, galleries, chat or popups — out of
  scope, deliberately. Four widgets.

## Command

```bash
# Validate the config, then write one fragment per widget
python3 src/widget_kit.py make --config CONFIG.json \
    --out DIR [--json]
```

Exit 2 means NO files were written and the output folder
was not created. Exit 1 means the fragments WERE written
and `warnings` says why.

## Config schema (exact)

Top level: `{"widgets": [...]}` — required, non-empty.

Every widget: `type` (required: `faq`, `pricing`,
`countdown` or `proof` — anything else is a problem
naming the valid types), `id` (optional; must match
`^[a-z0-9][a-z0-9-]*$`; defaults to the type; the output
file is `<id>.html` and no two widgets may share one),
`title` (optional string; faq/pricing/proof heading).

**faq** — `items` (required list) of `{"q", "a"}`, both
required non-empty strings. Empty list = warning.

**pricing** — `tiers` (required list). Per tier: `name`
(required string), `price` (required STRING, rendered
exactly as written — a JSON number is a problem, because
the tool does no currency math or reformatting), `period`
(optional string), `features` (required list of strings,
may be empty), `highlight` (optional boolean — the only
highlighting signal; no badge text is ever added),
`cta_label` + `cta_url` (optional pair — both or neither;
the URL is structurally checked offline: schemes https,
http, mailto, tel only, `javascript:`/`data:` refused,
http(s) needs a host, no whitespace). Empty list =
warning.

**countdown** — `target` (required string: ISO 8601
datetime WITH an explicit UTC offset such as
`2027-06-01T09:00:00-07:00`, or trailing `Z`; a bare date
or naive datetime is a problem), `ended_text` (required
non-empty string — shown when the moment has passed,
including immediately if the target is already past at
generation time), `label` (optional string above the
timer). Past at generation = warning, exit 1, ended
variant rendered (no timer, no script).

**proof** — exactly one of `entries` (inline list) or
`entries_file` (string; `.json` or `.csv`; resolved
relative to the config file's directory; JSON is a bare
list or `{"entries": [...]}`; CSV needs `name` and `text`
columns, `date`/`rating` optional). Per entry: `name`
(required string), `text` (required string), `date`
(optional, real `YYYY-MM-DD`), `rating` (optional integer
1–5 — strings, floats, booleans and out-of-range values
are problems). Empty entries = warning.

## The fragments (exact)

Each fragment starts with a one-line HTML comment (tool,
version, widget type), then ONE root element carrying all
of its CSS in an inline `<style>` (class prefix `wk-`) and,
where needed, inline vanilla JS. No `src=` references, no
`<link>`, no `@import`, no external anything.

- **faq**: `<section class="wk-faq">` with one
  `<details class="wk-faq-item"><summary>q</summary>
  <div class="wk-faq-a">a</div></details>` per item
  (works with JS disabled), then
  `<script type="application/ld+json">` holding
  `{"@context": "https://schema.org", "@type": "FAQPage",
  "mainEntity": [...]}` from the same list, dumped with
  `&`, `<`, `>` unicode-escaped.
- **pricing**: `<section class="wk-pricing">`, one
  `.wk-tier` card per tier (extra class `wk-tier-hl` only
  on flagged tiers), price + optional period, feature
  list, optional CTA `<a href>`.
- **countdown**: `<div class="wk-cd" data-target="EPOCH_MS">`
  (epoch derived ONLY from the configured target), label,
  timer spans (placeholders until JS runs), a static
  `Ends D Mon YYYY, HH:MM (UTC±HH:MM)` line that serves as
  the no-JS fallback, a hidden ended paragraph, and an
  inline script that ticks each second and swaps to the
  ended text at zero. Past-at-generation fragments carry
  the ended text alone — no `data-target`, no script.
- **proof**: `<section class="wk-proof">`, one
  `<figure class="wk-proof-card">` per entry IN INPUT
  ORDER: optional star row (`★`/`☆` with an aria-label
  "Rated N out of 5"), blockquote text, name caption,
  optional date. NO totals, averages or counts anywhere.

Every user-supplied string is HTML-escaped in text and
attributes. Output is deterministic: same config, same
bytes (the countdown embeds its target, never the
generation time).

## Output JSON schema (`--json`)

`{"tool": "widget-kit", "version", "command": "make",
"config_file", "output_dir", "fragments": [{"widget",
"id", "file", "bytes", "summary"}], "warnings": [...]}` —
`bytes` is the exact byte length of each written file;
`summary` is a countable fact (e.g. `"3 question(s)"`,
`"target 1 Jun 2027, 09:00 (UTC-07:00)"`,
`"3 supplied entries"`).

On error, stdout is a single object: `{"error": "<msg>"}`
with exit code 2 — including argparse usage errors in
`--json` mode. Validation failures add `"problems"`:
the full list of problem strings, in widget order. Never
plain text, never a traceback.

## Exit codes

- `0` — clean: every fragment written, no warnings.
- `1` — warnings: fragments WERE written; a countdown
  target is in the past and/or a widget's content list is
  empty. Read the warnings before publishing.
- `2` — ERROR: config missing or not valid JSON, at least
  one validation problem (all of them listed), an unknown
  widget type, an unsafe CTA link, an unreadable entries
  file, an unwritable output path, an invalid flag. NO
  fragment was written and no output folder was created.
  No output facts may be quoted from a failed run.

## Limits & honesty rules the agent must respect

- **Proof is rendered, never computed.** Never describe
  the proof fragment as showing an average, a total or a
  count — it contains none, by design. Never add entries,
  ratings or summary lines the owner's data doesn't hold,
  in the config or by editing output. SAMPLE entries in
  the bundled examples are placeholders, not testimonials:
  never present them as real.
- **Visible FAQ == JSON-LD.** Both come from one list.
  If you edit a question, edit the config and re-run —
  never hand-patch one side of the fragment, which
  recreates the schema-spam drift the tool exists to
  prevent.
- **A past countdown is a warning, not an error and not
  nothing.** Exit 1 with the ended text already rendered
  is the designed outcome for an expired event; surface
  the warning, don't "fix" it by moving the target unless
  the owner gives you a new real date.
- **Prices are the owner's strings.** The tool renders
  them verbatim and validates nothing about their truth.
  Never reformat, convert or round a price.
- **Exit 2 means nothing exists.** Do not reconstruct
  fragments by hand from a refused config — the collected
  problems are exactly why the gate exists. Fix the
  config and re-run.
- Local only: the tool reads only the config file (and
  the entries file it names) and sends nothing anywhere;
  the fragments make no requests either.
