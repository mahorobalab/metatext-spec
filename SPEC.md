# MetaText Sidecar Specification (`.metatext.jsonc`) v1

> Status: v1.0 (Canonical Schema URL: `https://mahorobalab.github.io/metatext-spec/schemas/v1/metatext.schema.json`)

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

## 3. JSONC Preservation, Versioning & Interoperability Rules

1. Sidecar files use **JSON with Comments (JSONC)** (line comments `//`, block comments `/* */`, and trailing commas allowed).
2. Editors modifying `.metatext.jsonc` **MUST** perform targeted/AST edits (such as Microsoft `jsonc-parser` `modify`/`applyEdits`) rather than re-serializing the entire object, so existing comments and formatting remain intact.
3. Newly generated `.metatext.jsonc` files **SHOULD** include `"$schema": "https://mahorobalab.github.io/metatext-spec/schemas/v1/metatext.schema.json"` and `"version": "1.0"` at the top level for self-documentation and editor auto-completion.
4. Unknown properties inside `extensions` or root-level `x-*` keys **MUST NOT** be deleted when another tool saves unrelated properties.
5. **Versioning Policy**: The `v1` schema URL (`.../schemas/v1/metatext.schema.json`) receives only backward-compatible minor additions (`1.x`). Legacy draft files with `"version": "0.1"` remain valid under `v1`.

---

## 4. Schema Structure

JSON Schema (draft-07): [`schemas/v1/metatext.schema.json`](schemas/v1/metatext.schema.json)

### 4.1 Core Top-Level Properties

| Property | Type | Description |
| --- | --- | --- |
| `$schema` | `string` (URI) | Schema URL (`https://mahorobalab.github.io/metatext-spec/schemas/v1/metatext.schema.json`) |
| `version` | `"1.0"` (or `1.x`, legacy `"0.1"`) | Schema version |
| `format` | `"csv" \| "tsv" \| "log" \| "jsonl" \| "text"` | Explicit target format (auto-detected from extension if omitted) |
| `readOnly` | `boolean` | Open target in read-only mode by default (recommended `true` for log files) |
| `csv` | `object` | Dialect configuration for CSV/TSV files |
| `jsonl` | `object` | Flattening and encoding configuration for JSON Lines (`.jsonl` / `.ndjson`) files |
| `log` | `object` | Parsing and display configuration for log/text files |
| `table` | `object` | Table-wide layout (`fontFamily`, `fontSize`, `defaultRowHeight`, `headerHeight`, `freezeColumns`, `zebraStriping`) |
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

### 4.3 `CellStyle` & `ColumnConfig`

#### `CellStyle`
- `bold`, `italic` (`boolean`)
- `color`, `backgroundColor` (`string`, CSS color)
- `fontFamily` (`string`), `fontSize` (`number`, `8..48`)
- `wrap` (`boolean`): Wrap long text inside this cell or range.
- `align` (`"left" | "center" | "right"`): Horizontal text alignment.
- `numberFormat` (`string`): Excel-compatible number format string (ECMA-376 / `numfmt`, e.g. `#,##0`, `¥#,##0`, `0.0%`).

#### `ColumnConfig`
- `label` (`string`), `width` (`number`, `24..2000`), `visible` (`boolean`, default `true`)
- `align` (`"left" | "center" | "right"`), `wrap` (`boolean`, default `false`), `readonly` (`boolean`, default `false`)
- `type` (`"text" | "number" | "boolean" | "enum" | "date"`, default `"text"`), `options` (`string[]` for `enum`)
- `numberFormat` (`string`), `style` (`StyleRef`: named style key or inline `CellStyle`)

### 4.4 Style Cascade Order

Resolved from lowest to highest precedence:
1. `table` (defaults)
2. `columns[key].style`
3. `rules[]` (evaluated in order)
4. `ranges[]` (evaluated in order)
5. `rows[rowIndex].style`
6. `cells["<rowIndex>:<colKey>"]`

---

## 5. Portable Bundle Format (`.metatext.zip`) v0.1

For single-file sharing, archiving, and cross-environment transfer (Desktop, VS Code, Web), a source file, its `.metatext.jsonc` sidecar, and an optional Shadow Git snapshot history can be packaged into a standard DEFLATE ZIP archive with the suffix **`.metatext.zip`**.

### 5.1 File Naming Convention

Preserve the original source file extension before `.metatext.zip`:

```text
<sourceFileName>.metatext.zip
```

Examples:
- `report.csv` → `report.csv.metatext.zip`
- `metrics.tsv` → `metrics.tsv.metatext.zip`
- `events.jsonl` → `events.jsonl.metatext.zip`
- `app.log` → `app.log.metatext.zip`

Tools **MUST NOT** treat arbitrary `.zip` files without the `.metatext.zip` suffix (or without a valid `manifest.json`) as MetaText bundles.

### 5.2 Archive Layout

```text
report.csv.metatext.zip
├── manifest.json                         # Required bundle manifest
├── data/
│   ├── report.csv                        # Required primary data file (encoding & line endings preserved)
│   └── report.csv.metatext.jsonc         # Required sidecar metadata file
└── history/                              # Optional Shadow Git bare repository (omitted by default)
    ├── HEAD
    ├── config
    ├── objects/...
    └── refs/...
```

### 5.3 `manifest.json` Schema

JSON Schema (draft-07): [`schemas/v0.1/metatext-bundle.schema.json`](schemas/v0.1/metatext-bundle.schema.json)

```json
{
  "bundleVersion": "0.1",
  "format": "csv",
  "sourceFile": "data/report.csv",
  "sidecarFile": "data/report.csv.metatext.jsonc",
  "includesHistory": false,
  "createdAt": "2026-11-26T05:00:00Z",
  "createdBy": "MetaText Editor v0.1.0"
}
```

### 5.4 Validation, Direct Editing & Security Rules

1. **Zip Slip Prevention**: Tools **MUST** reject any ZIP entry path or manifest path containing `..`, leading `/`, Windows drive prefixes (e.g. `C:`), or ASCII control characters.
2. **Zip Bomb Prevention**: Tools **MUST** enforce limits on entry count (default `10,000`), total uncompressed size (default `512 MiB`), and suspicious compression ratios (default `> 100x` for entries `>= 1 MiB`) before extracting.
3. **Privacy Default**: Export UIs **SHOULD** default `includesHistory` to `false` so local editing history is not unintentionally shared.
4. **Atomic Direct Save**: Tools supporting direct editing of `.metatext.zip` **SHOULD** write updated archives to a temporary file and atomically rename over the original bundle, preserving `createdAt`, `createdBy`, and embedded `history/` entries.

---

## 6. Reserved Future Extensions (Post-v1.0)

The following capabilities are planned for future minor revisions (`1.x`) while remaining backward-compatible with v1.0 sidecar files:
- **Snapshot History Metadata**: Metadata referencing external/shadow Git snapshot repositories (e.g. `gix` object stores) for dataset + sidecar time travel.
