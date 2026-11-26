# MetaText Sidecar Specification (`.metatext.jsonc`) v0.1

> Status: Draft v0.1

## 1. Overview & Goals

**MetaText** defines an open, human-readable sidecar metadata format (`.metatext.jsonc`) for plain-text tabular and line-oriented files (`csv`, `tsv`, `log`, `jsonl`, `text`).

- **Non-destructive separation**: Keep the source data file 100% clean (preserving encoding, line endings, leading zeros, and raw log lines) while storing presentation, parsing, filtering, bookmarks, and styling metadata in an adjacent `.metatext.jsonc` file.
- **Git-friendly**: Small text diffs in JSONC format allow teams to review and share column layouts, conditional rules, filter presets, and bookmarked investigation notes without touching gigabytes of raw data.
- **Zero-Config**: Tools MUST be able to open any source file without a sidecar present, creating `.metatext.jsonc` only when metadata is modified or explicitly saved.

---

## 2. File Naming & Discovery

For any source file `<path>`, the canonical sidecar metadata path is:

```text
<path>.metatext.jsonc
```

Examples:
- `data.csv` → `data.csv.metatext.jsonc`
- `metrics.tsv` → `metrics.tsv.metatext.jsonc`
- `app.log` → `app.log.metatext.jsonc`
- `events.jsonl` → `events.jsonl.metatext.jsonc`

---

## 3. JSONC Preservation & Interoperability Rules

1. Sidecar files use **JSON with Comments (JSONC)** (line comments `//`, block comments `/* */`, and trailing commas allowed).
2. Editors modifying `.metatext.jsonc` **MUST** perform targeted/AST edits (such as Microsoft `jsonc-parser` `modify`/`applyEdits`) rather than re-serializing the entire object, so existing comments and formatting remain intact.
3. Unknown properties inside `extensions` or root-level `x-*` keys **MUST NOT** be deleted when another tool saves unrelated properties.

---

## 4. Schema Structure

JSON Schema (draft-07): [`schemas/v0.1/metatext.schema.json`](schemas/v0.1/metatext.schema.json)

### 4.1 Core Top-Level Properties

| Property | Type | Description |
| --- | --- | --- |
| `$schema` | `string` | Schema URL (`https://mahorobalab.github.io/metatext-spec/schemas/v0.1/metatext.schema.json`) |
| `version` | `"0.1"` | Schema version |
| `format` | `"csv" \| "tsv" \| "log" \| "jsonl" \| "text"` | Explicit target format (auto-detected from extension if omitted) |
| `readOnly` | `boolean` | Open target in read-only mode by default (recommended `true` for log files) |
| `csv` | `object` | Dialect configuration for CSV/TSV files |
| `jsonl` | `object` | Flattening and encoding configuration for JSON Lines (`.jsonl` / `.ndjson`) files |
| `log` | `object` | Parsing and display configuration for log/text files |
| `table` | `object` | Table-wide layout (`fontFamily`, `fontSize`, `defaultRowHeight`, `headerHeight`, `freezeColumns`) |
| `view` | `object` | Persistent non-destructive view state (`sort`, `activeFilter`, `filterContextRows`, `filterPresets`, `markers`) |
| `styles` | `Record<string, CellStyle>` | Named style dictionary |
| `columns` | `Record<string, ColumnConfig>` | Per-column settings keyed by header name, capture group name, or 0-based index string |
| `rules` | `ConditionalRule[]` | Ordered conditional formatting rules |
| `ranges` | `RangeStyleRule[]` | Rectangular `<startRow>:<startCol>..<endRow>:<endCol>` style ranges |
| `rows` | `Record<string, RowConfig>` | Per-row overrides keyed by 0-based raw data row index |
| `cells` | `Record<string, StyleRef>` | Per-cell overrides keyed by `<rowIndex>:<colKey>` |
| `extensions` | `object` | Namespaced tool-specific extensions |
| `x-*` | `any` | Vendor-prefixed root extension fields |

### 4.2 Format-Specific Sections

#### `csv`
- `hasHeader` (`boolean`, default `true`): Whether row 0 is treated as column headers.
- `delimiter` (`"," | "\t" | ";" | "|"`): Field delimiter.
- `encoding` (`"utf-8" | "utf-8-bom" | "shift-jis"`): Character encoding preserved on write.
- `lineEnding` (`"lf" | "crlf"`): Line ending preserved on write.

#### `jsonl`
- `flattenDepth` (`integer`, `0..8`, default `2`): Maximum nesting depth for dot-notation flattening (e.g. `user.name`). Deeper nested objects stay serialized as compact JSON strings.
- `arrayMode` (`"json" | "join"`, default `"json"`): How array values are rendered in cells (`"json"` for compact JSON string, `"join"` for comma-separated primitives).
- `encoding` (`"utf-8" | "utf-8-bom" | "shift-jis"`): Character encoding preserved on write.
- `lineEnding` (`"lf" | "crlf"`): Line ending preserved on write.

#### `log`
- `linePattern` (`string`): Regular expression splitting each log line into columns (named or numbered capture groups).
- `columnNames` (`string[]`): Fallback column names for numbered capture groups.
- `timestampColumn` (`string`): Key of the timestamp column.
- `levelColumn` (`string`): Key of the severity column (`ERROR`, `WARN`, `INFO`, etc.).
- `multilinePattern` (`string`): Regular expression matching continuation lines (e.g. stack traces).
- `wordWrap` (`boolean`, default `false`): Default line wrapping preference.
- `encoding` (`"utf-8" | "utf-8-bom" | "shift-jis"`): Log file character encoding.

### 4.3 Style Cascade Order

Resolved from lowest to highest precedence:
1. `table` (defaults)
2. `columns[key].style`
3. `rules[]` (evaluated in order)
4. `ranges[]` (evaluated in order)
5. `rows[rowIndex].style`
6. `cells["<rowIndex>:<colKey>"]`

---

## 5. Reserved Future Extensions (Post-v0.1)

The following capabilities are planned for future revisions while remaining backward-compatible with v0.1 sidecar files:
- **Portable Bundle (`.metatext.zip`)**: Packaging the source data file, `.metatext.jsonc`, and optional snapshot history into a single portable archive.
- **Snapshot History Metadata**: Metadata referencing external/shadow Git snapshot repositories (e.g. `gix` object stores) for dataset + sidecar time travel.
