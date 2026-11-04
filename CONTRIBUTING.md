# Contributing

Contributions are welcome — bug reports, fixes and small focused features.

## Ground rules

- **Standard library only.** widget-kit's promise is
  "clone and run". Pull requests that add a third-party
  dependency will not be merged.
- **Local only, and nothing fetches.** No network calls,
  no telemetry, no uploads — in the tool or in the
  fragments it generates. A fragment that loads an
  external stylesheet, font, image or script breaks the
  product's reason to exist; the test suite scans the
  generated fragments and pins their URL set to the
  config's own links.
- **Proof is rendered, never computed.** No average
  ratings, no entry counts on the page, no summary claims
  — and no sample entries that could pass for real
  testimonials. Features that summarise, rank or pad the
  owner's proof will not be merged, however convenient
  they look in a demo.
- **Visible FAQ == JSON-LD, always.** The two are
  generated from one list. Any change that lets them be
  edited or generated separately recreates the
  schema-spam drift this tool exists to prevent.
- **Validation collects, it does not bail.** One run must
  report every problem in the config. Changing a check to
  raise on first failure — or writing some fragments
  before a late problem surfaces — breaks the "nothing
  written on exit 2" contract.
- **Countdown targets keep their offsets.** Accepting a
  naive datetime means guessing a timezone, and a guessed
  countdown counts to the wrong moment. The offset
  requirement is the feature.
- **Escaping is not optional.** Any new place user text
  reaches a fragment goes through the same HTML escaping,
  in text and in attributes — and JSON-LD content through
  the unicode escaping too — with a test that puts markup
  in the input.
- **Both directions tested.** A behaviour change needs a
  test where good input succeeds and bad input fails
  cleanly — not just one side. Byte-size expectations come
  from the written files, not from constants copied out
  of a run.

## Getting started

```bash
git clone https://github.com/jasveerai88/widget-kit
cd widget-kit
python3 -m unittest discover -s tests -v
python3 src/widget_kit.py make \
    --config examples/widgets.json --out site-widgets
```

## Pull requests

1. Fork, branch, and keep the change to one topic.
2. Add or update tests under `tests/`; the full suite must pass.
3. Note the change in `CHANGELOG.md` under an "Unreleased" heading.
4. Describe the behaviour change with a real before/after run if
   the CLI output changes.
