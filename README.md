# fizz-persona

**Fizz** — AITuber system in [Almide](https://github.com/almide/almide) — の
persona オンディスク形式契約。

[openaituber](https://github.com/aiviecast/openaituber) の
`docs/almide-component-breakdown.md` §1 で定義した契約のうち、
**persona-format** を担う。ランタイムのワイヤ契約は
[fizz-protocol](https://github.com/aiviecast/fizz-protocol)。

kotodama の `personae/<name>/` 構成 (spec.json / system.md / voice.json /
world.json / transforms.json / triggers.json) と decode 互換。

## Install

```toml
# almide.toml
[dependencies]
fizz_persona = { git = "https://github.com/aiviecast/fizz-persona", tag = "v0.1.0" }
```

## On-disk layout

```
personae/<name>/
  spec.json         → identity.PersonaSpec   入口。他ファイル参照 + effect 制約
  system.md         → String                 renderer 指示書 (そのまま読む)
  voice.json        → voice.VoiceProfile     レジスタ / 長さ / 自称ルール
  world.json        → world.World            キャラ世界資産
  transforms.json   → transform.Transforms   回答変換パターン (有限集合)
  triggers.json     → trigger.Triggers       入力パターン → transform prior
  examples.tsv      → examples               few-shot 例文 (質問<TAB>回答)
```

## Modules

| module | contract |
|---|---|
| `fizz_persona` (mod) | 集約型 `Persona` と `validate()` (ファイル間整合検査) |
| `fizz_persona.identity` | `PersonaSpec` / `EffectPolicy` — spec.json |
| `fizz_persona.voice` | `VoiceProfile` / `Register` / `Lengths` — voice.json |
| `fizz_persona.world` | `World` / `WorldAsset` — world.json |
| `fizz_persona.transform` | `Transforms` / `Transform` — transforms.json |
| `fizz_persona.trigger` | `Triggers` / `TriggerRule` + `match_triggers()` — triggers.json |
| `fizz_persona.examples` | `FewShotExample` + TSV parse/format (形式の正本) |

## Design notes

- **形式だけを定める。** ファイル IO は persona-loader 部品の責務 —
  このパッケージは `fs` に依存しない (JSON `Value` を受けて decode するだけ)。
- **試行錯誤はリビルド不要。** persona はすべて手編集可能なオンディスク
  ファイル。形式が変わるときだけこのリポジトリのバージョンが上がる。
- **effect 制約は宣言が正。** `EffectPolicy.allowed` は kotodama 系
  ランタイムが lower-time に静的検査する前提のリスト。
- `validate()` は triggers の suggest が transforms に実在することを検査する。

## Usage

```almide
import fizz_persona.identity

// loader 部品が読んだ spec.json テキストを decode する例
// (コーデックは手書き — almide の deriving Codec は variant payload
//  decode 未実装のため使っていない)
fn parse_spec(text: String) -> Result[identity.PersonaSpec, String] =
  identity.spec_from_json(text)
```

## Tests

```bash
almide test
```
