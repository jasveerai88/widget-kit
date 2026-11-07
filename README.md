> **Merged into [site-kit](https://github.com/jasveerai88/site-kit)** — this tool now ships as the `widget-kit` module of the site-kit suite (v1.0.0, also on PyPI). New work happens there; this repo stays as the original standalone release.

# widget-kit

**A widget should not be rented by the view.**

Elfsight charges $5–6/mo for a single widget app and
$15–36/mo for all apps — and the view allowance is *shared
across every site on the plan*. G2 reviewers report "one
client can eat all the credits in a widget" and reviews
being deactivated even on paid plans. Fomo charges
$25–149/mo by notification volume, plus $75/mo for agency
white-label, with complaints about auto-upgrades and
overage fees. But a widget is markup, and markup is a file.

`widget-kit` generates the four boring widgets every small
site needs — an **FAQ** block with matching FAQPage
JSON-LD, a **pricing** table, a **countdown** bar and a
**social-proof** card row — as static HTML fragments you
own. One config in, one self-contained fragment per widget
out: paste them into any page, on any host, forever, for
free. There is no account and no script phoning home, so
there is nothing to meter.

- Pure Python 3 standard library — no dependencies, no install step
- Local only — the tool never fetches the network; the
  fragments load nothing either (no external stylesheets,
  fonts, images or scripts — the only URLs anywhere are
  your own pricing links)
- Script-friendly exit codes: `0` = clean, `1` = warnings
  (fragments written — a countdown already past, an empty
  widget), `2` = error, nothing written

## The config

One JSON file holding a `widgets` list. Each widget makes
one fragment file, `<id>.html`, in the output folder:

```json
{
  "widgets": [
    {"type": "faq", "id": "faq", "title": "Questions",
     "items": [{"q": "Do you deliver?", "a": "Yes, ..."}]},
    {"type": "pricing", "id": "pricing",
     "tiers": [{"name": "Basic", "price": "$19",
                "period": "/mo", "features": ["..."],
                "highlight": true,
                "cta_label": "Start",
                "cta_url": "https://example.org/join"}]},
    {"type": "countdown", "id": "launch",
     "target": "2027-06-01T09:00:00-07:00",
     "label": "Spring sale ends in",
     "ended_text": "The spring sale has ended."},
    {"type": "proof", "id": "proof",
     "entries_file": "proof-entries.json"}
  ]
}
```

| Field | Meaning |
|---|---|
| `widgets` | required, non-empty list of widget objects |
| `type` | one of `faq`, `pricing`, `countdown`, `proof` |
| `id` | optional; lowercase letters, digits, dashes — names the output file (`<id>.html`) and defaults to the type. Two widgets may not share an output filename |
| `title` | optional heading (faq / pricing / proof) |

### faq

`items` is a list of `{"q", "a"}` — both required. The
fragment is semantic HTML (`<details>`/`<summary>`), so it
**works with JavaScript disabled**, plus a FAQPage JSON-LD
block generated from **the same list**. Schema that drifts
from the visible page is the classic FAQ rich-result
failure; here the two cannot drift, because there is one
source.

### pricing

Each tier: `name` and `price` required; `period` optional
(`"/mo"`); `features` a required list of strings (may be
empty); `highlight` an optional boolean; `cta_label` and
`cta_url` an optional pair (both or neither). **Prices
render exactly as written** — `price` is a string like
`"$49"` or `"Free"`; the tool does no currency math and
never reformats your price. A tier is highlighted **only**
when you flag it, and no badge text ("Most popular",
"Best value") is invented for it. CTA links are checked
structurally (https/http/mailto/tel only — a `javascript:`
link is refused).

### countdown

`target` is an ISO 8601 datetime **with an explicit UTC
offset** (`"2027-06-01T09:00:00-07:00"`, or `Z`). A bare
date or a naive datetime is refused: a countdown that
guesses a timezone counts to the wrong moment.
`ended_text` is required — it is what the fragment shows
once the moment passes, swapped in by the inline script
when the timer reaches zero. With JavaScript off, the
fragment still shows the target date as static text. If
the target is **already past** when you generate, the
fragment renders the ended text immediately and the run
exits 1 with a warning — never a silent countdown to a
finished event. The fragment embeds the target only, never
the generation time, so re-rendering is byte-identical.

### proof

Entries come inline (`"entries": [...]`) or from an
`entries_file` — JSON (a list, or `{"entries": [...]}`) or
CSV (header `name,text,date,rating`) — resolved relative
to the config file. Each entry: `name` and `text`
required; `date` optional (`YYYY-MM-DD`); `rating`
optional integer 1–5, rendered as stars.

**Honesty is the design.** The fragment renders only the
entries you supply, verbatim, in your order. It computes
*nothing* about them — no average rating, no entry count
on the page, no "500+ happy customers" line, ever. If a
number like that appears on your site, you typed it into a
text field yourself. Publish only proof you actually
have. The bundled samples are labelled SAMPLE throughout
for exactly that reason.

## Validation collects everything

A bad config fails **once, with the full list**: every
problem the validator finds — across every widget — is
reported together, the run exits 2, and **nothing is
written** (the output folder is not even created). The
bundled `examples/bad-config.json` holds six planted
problems; the run below lists all six.

Warnings are different: the fragments *are* written and
the run exits 1. A warning is a countdown whose target
has already passed, or a faq/pricing/proof widget with an
empty content list (its fragment holds only its heading).

## Usage

```bash
# Generate every widget in the config
python3 src/widget_kit.py make \
    --config examples/widgets.json --out site-widgets

# The same, for a script or an agent (one JSON object)
python3 src/widget_kit.py make \
    --config examples/widgets.json --out site-widgets \
    --json
```

Then paste each fragment file into a page wherever the
widget belongs. Every fragment is standalone — inline CSS,
inline JS where it needs any — so fragments can be mixed
across pages and sites freely.

## Use with AI agents

Every run takes `--json`: one JSON object with the same
facts and exit codes as the text view (errors come back as
`{"error": ...}`, with a `problems` list when validation
fails; argparse usage errors included):

```bash
python3 src/widget_kit.py make \
    --config examples/widgets.json --out site-widgets \
    --json
```

Agents: read [SKILL.md](SKILL.md) first — the config
schema per widget, the JSON output schema, the exit codes,
and the honesty rules (visible FAQ == JSON-LD; proof is
rendered, never computed; a past countdown is a warning,
not a failure).

## Real runs (verbatim)

The bundled sample is a fictional bakery (The Daily
Crumb): a 3-question FAQ, Crumb Club pricing in 3 tiers,
a bake-sale countdown, and 3 SAMPLE proof entries:

```text
$ python3 src/widget_kit.py make --config examples/widgets.json --out site-widgets
widget-kit v0.1.0 — make · config widgets.json
Wrote 4 fragment(s) into site-widgets:
  faq.html — 2,221 bytes · faq · 3 question(s)
  pricing.html — 2,214 bytes · pricing · 3 tier(s)
  bake-sale.html — 1,988 bytes · countdown · target 1 Jun 2027, 09:00 (UTC-07:00)
  proof.html — 2,050 bytes · proof · 3 supplied entries
Warnings: none
(exit code: 0)
```

The same run as one JSON object:

```text
$ python3 src/widget_kit.py make --config examples/widgets.json --out site-widgets --json
{"tool": "widget-kit", "version": "0.1.0", "command": "make", "config_file": "widgets.json", "output_dir": "site-widgets", "fragments": [{"widget": "faq", "id": "faq", "file": "faq.html", "bytes": 2221, "summary": "3 question(s)"}, {"widget": "pricing", "id": "pricing", "file": "pricing.html", "bytes": 2214, "summary": "3 tier(s)"}, {"widget": "countdown", "id": "bake-sale", "file": "bake-sale.html", "bytes": 1988, "summary": "target 1 Jun 2027, 09:00 (UTC-07:00)"}, {"widget": "proof", "id": "proof", "file": "proof.html", "bytes": 2050, "summary": "3 supplied entries"}], "warnings": []}
(exit code: 0)
```

Scanning all four fragments for URLs returns exactly the
two pricing links the config asked for — plus the
schema.org context identifier inside the FAQ JSON-LD,
which names a vocabulary, is part of the structured data,
and is never fetched:

```text
$ grep -hoE 'https?://[^" <>]+' site-widgets/*.html | sort -u
https://example.org/crumb-club/join
https://example.org/crumb-club/join-household
https://schema.org
```

The bad config — six problems, one report, nothing
written (the output folder is never created):

```text
$ python3 src/widget_kit.py make --config examples/bad-config.json --out bad-out
ERROR: config validation failed with 6 problem(s):
  - widgets[0] (faq).items[0]: missing required field 'a'
  - widgets[1]: unknown widget type 'chat' (valid types: faq, pricing, countdown, proof)
  - widgets[2] (pricing).tiers[0]: field 'price' must be a string rendered exactly as written, e.g. "$49" (got 49)
  - widgets[2] (pricing).tiers[0]: 'cta_label' and 'cta_url' must be given together (both or neither)
  - widgets[3] (countdown): missing required field 'ended_text'
  - widgets[3] (countdown): field 'target' must carry an explicit UTC offset, e.g. "2027-06-01T09:00:00-07:00" (got '2027-06-01T09:00:00') — a countdown that guesses a timezone counts to the wrong moment
(exit code: 2)
```

And a countdown whose day has come and gone — a warning,
the fragment written with its ended text showing:

```text
$ python3 src/widget_kit.py make --config examples/past-countdown.json --out past-out
widget-kit v0.1.0 — make · config past-countdown.json
Wrote 1 fragment(s) into past-out:
  sale.html — 772 bytes · countdown · target 1 Jan 2020, 00:00 (UTC-08:00)
Warnings:
  ! widgets[0] (countdown): countdown target '2020-01-01T00:00:00-08:00' is already in the past — the fragment will show its ended_text
(exit code: 1)
```

## Tests

```bash
python3 -m unittest discover -s tests -v
# Ran 48 tests — OK
```

The suite covers: the happy path for all four widgets;
the FAQ contract (visible Q&A extracted independently and
compared to both the config and the JSON-LD — all three
must agree); escaping in both directions (a `<script>`
question renders as text and parses back out of the
JSON-LD exactly); pricing (prices verbatim, highlight
only on the flagged tier, no invented badge text, CTA
scheme refusals); countdown (epoch pinned to a
hand-computed value, `Z` and offset forms agreeing, past
targets warning with ended text, naive targets refused);
proof (rendered count and order match the input exactly,
no computed claims anywhere in the fragment, CSV and JSON
entry files, bad ratings and dates refused); validation
(the six-problem config reported in one run with nothing
written, duplicate output filenames, bad ids); the
fragment contract (no `src=`, no external references
beyond the configured links); determinism (two runs are
byte-identical); and the clean-error shape in both modes.

## Dual use

- **For site owners (the Sponsors surface):** the four
  standard widgets as files you own — the free, permanent
  answer to per-view metering and per-site widget rent.
- **For its author:** standard widgets for Focus Brilliant
  client sites and the author's own landing pages,
  generated from each client's own data instead of a
  hosted widget account per client.

## Known limits

- **Static by design.** No live data: the proof widget
  shows the entries in your file (export fresh entries and
  re-run for fresh proof), the countdown counts to a fixed
  target, and there is no auto-pulled review feed — that
  needs API keys and a server, which is the hosted product
  this tool deliberately is not.
- **One config, one folder of fragments.** No templating
  language, no theme system beyond each fragment's inline
  CSS — edit the generated file if you need custom design;
  it's yours.
- **The countdown's clock is the visitor's.** Like every
  client-side countdown, it counts against the device's
  clock; the configured offset pins the *target moment*,
  not the visitor's timezone.
- **Four widgets.** FAQ, pricing, countdown, proof. Scope
  is capped on purpose — galleries, forms and popups are
  other tools' jobs.

## Privacy

What it touches: only the config file (and the proof
entries file it names). What it sends: nothing — there is
no network code in this tool. The fragments send nothing
either: no analytics, no trackers, no fonts, no requests
of any kind — a visitor's browser loads only the page you
pasted them into.

## Support

If this replaces a metered widget subscription for you,
consider sponsoring: **https://github.com/sponsors/jasveerai88**

## Licence

MIT — see [LICENSE](LICENSE).
