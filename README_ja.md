# MetaText 仕様 (`.metatext.jsonc`)

[English (README.md)](./README.md) | [詳細仕様 (SPEC.md)](./SPEC.md) | [Web ページ](https://mahorobalab.github.io/metatext-spec/)

**MetaText** は、`CSV`・`TSV`・`Log`・`JSONL` などのプレーンテキストファイルに対して、データ本体を変更せずに見た目・パース設定・フィルタプリセット・行マーカー・条件付きハイライトなどを外付けで保存するためのオープンな JSONC サイドカー仕様（`<filename>.metatext.jsonc`）です。

## スキーマ・サンプル

- JSON Schema (v1): [`schemas/v1/metatext.schema.json`](./schemas/v1/metatext.schema.json)
- 公式配信 URL: `https://mahorobalab.github.io/metatext-spec/schemas/v1/metatext.schema.json`
- サンプル:
  - [`examples/spec_setting.csv.metatext.jsonc`](./examples/spec_setting.csv.metatext.jsonc)（ヘッダーなし CSV＋スタイル辞書＋条件付き書式）
  - [`examples/metrics.tsv.metatext.jsonc`](./examples/metrics.tsv.metatext.jsonc)（TSV＋列設定＋ビューソート）
  - [`examples/events.jsonl.metatext.jsonc`](./examples/events.jsonl.metatext.jsonc)（読み取り専用 JSONL 自動列展開＋レイテンシ抽出＋レベル別色分け）
  - [`examples/app.log.metatext.jsonc`](./examples/app.log.metatext.jsonc)（読み取り専用ログパース＋レベル別色分け＋フィルタプリセット＋マーカー）

## 基本的な使い方

対象ファイルと同じディレクトリに `<ファイル名>.metatext.jsonc` を配置します：

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

## リファレンス実装

- [MetaText Editor (`mahorobalab/metatext-editor`)](https://github.com/mahorobalab/metatext-editor)

## ライセンス

MIT License — 詳細は [`LICENSE`](./LICENSE) を参照してください。
