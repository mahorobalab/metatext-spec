# MetaText Specification (`.metatext.jsonc`)

[日本語 (README_ja.md)](./README_ja.md) | [Specification (SPEC.md)](./SPEC.md) | [Web Page](https://mahorobalab.github.io/metatext-spec/)

**MetaText** is an open JSONC sidecar metadata specification (`.metatext.jsonc`) for plain-text tabular and log files (`CSV`, `TSV`, `Log`, `JSONL`, and plain text).

It lets editors, viewers, and CLI/IDE tools persist layout, parsing rules, filter presets, row bookmarks/markers, conditional highlights, and cell styling next to the data file (`<filename>.metatext.jsonc`) without modifying the raw data.

## Schema & Examples

- JSON Schema (v1): [`schemas/v1/metatext.schema.json`](./schemas/v1/metatext.schema.json)
- Public Schema URL: `https://mahorobalab.github.io/metatext-spec/schemas/v1/metatext.schema.json`
- Examples:
  - [`examples/spec_setting.csv.metatext.jsonc`](./examples/spec_setting.csv.metatext.jsonc) (headerless CSV + styles + rules)
  - [`examples/metrics.tsv.metatext.jsonc`](./examples/metrics.tsv.metatext.jsonc) (TSV + column formatting + sort)
  - [`examples/events.jsonl.metatext.jsonc`](./examples/events.jsonl.metatext.jsonc) (read-only JSONL flattening + latency filter + severity rules)
  - [`examples/app.log.metatext.jsonc`](./examples/app.log.metatext.jsonc) (read-only log parser + severity rules + presets + markers)

## Quick Usage

Place `<file>.metatext.jsonc` alongside your data file:

```jsonc
{
  "$schema": "https://mahorobalab.github.io/metatext-spec/schemas/v1/metatext.schema.json",
  "version": "1.0",
  "format": "csv",
  "table": {
    "freezeColumns": 1,
    "defaultRowHeight": 28
  }
}
```

## Reference Implementation

- [MetaText Editor (`mahorobalab/metatext-editor`)](https://github.com/mahorobalab/metatext-editor)

## License

MIT License — see [`LICENSE`](./LICENSE).
