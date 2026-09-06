# PCN — Portable Curling Notation

A text notation for recording a curling game, one line per shot. Readable and writable by people, processable exactly by machines, reproducible by a physics simulation.

```
[PCN "1.0"]
[Event "LGT World Men's Curling Championship 2026"]
[Date "2026.04.02"]
[Red "JPN - Japan"]
[Yellow "POL - Poland"]
[Ends "10"]
[LSFE "Red"]
[Result "5-0"]
[Termination "Concede"]

End 1 hammer=R
1 Y F>4
2 R D>4
3 Y D>2
4 R D<4
...
16 R D<4
Score 3-0
```

`13 Y P>1` = 13th stone of the end, yellow, a Promotion Take-out, clockwise rotation, rated 1 of 4.

- **Specification** — [English](spec/PCN_SPEC.en.md) · [日本語（正本 / normative）](spec/PCN_SPEC.md)
- **Grammar** — [EBNF](grammar/pcn.ebnf)
- **PCN-JSON** — [JSON Schema](schema/pcn-json.schema.json)
- **Examples** — [`examples/`](examples/) (one file per feature, plus one real game)
- **Reference implementation** — [CurlFlux](https://curlflux.creco.net/), a browser strategy board that replays PCN files with a deterministic physics engine. Its "Open PCN" button reads any file in this format.

## Why

World Curling publishes every stone of the Olympics and World Championships as "Shot by Shot", but as PDF, with stone positions as images. Simulators have their own log formats. Line scores live in databases. There was no common text format that all of these could flow into, the way PGN serves chess. PCN is designed to **contain existing records rather than replace them**.

## Four layers

| Layer | Content | Where it exists today |
|---|---|---|
| 1 | Line score, last-stone-in-first-end | Results databases |
| 2 | Thrower, shot type, rotation, 0–4 rating | World Curling / CURLIT Shot by Shot |
| 3 | Position of every stone after each shot | Results Book diagrams, Live Scores SVG, coaching apps |
| 4 | Release position, velocity, spin | Simulators, measuring devices |

Write only the layers you have. Layer 3 (positions) is the contract between implementations; layer 4 (velocities) is bound to one physics engine and is always paired with an `[Engine]` tag. A reader with a different engine ignores `v=` and refits the throw from the positions.

Shot types and ratings follow the World Curling / CURLIT statisticians' manual *Curling Statistics: How to Score*. Rotation is written `>` (clockwise, curls right) and `<` (counter-clockwise, curls left) instead of in-turn / out-turn, so the meaning does not depend on the thrower's handedness.

## Status

PCN 1.0 is a **draft** (5th revision, 2026-09-04). The `[PCN "1.0"]` value names the format family and will not change while the document is a draft; the document itself is versioned by revision (see [CHANGELOG](CHANGELOG.md)). The Japanese text is normative; where the English edition differs, the Japanese edition prevails.

This is an unofficial document with no affiliation to World Curling or CURLIT. Where its reading of the rules differs from *The Rules of Curling* (World Curling, July 2024), the original prevails.

## Examples

| File | Shows |
|---|---|
| [`01-linescore.pcn`](examples/01-linescore.pcn) | Layer 1 only: ends and scores, no shots |
| [`02-shot-by-shot.pcn`](examples/02-shot-by-shot.pcn) | Layer 2: task, rotation, rating for every stone |
| [`03-positions.pcn`](examples/03-positions.pcn) | Layer 3: `@` board lines with the `*` thrown-stone mark |
| [`04-mixed-doubles.pcn`](examples/04-mixed-doubles.pcn) | `Format`, `Setup` pre-positioned stones, `powerplay` |
| [`05-irregular.pcn`](examples/05-irregular.pcn) | A FGZ violation with `x=fgz`, a game stopped on the time limit, `Score X` |
| [`06-scenario.pcn`](examples/06-scenario.pcn) | A mid-end position cut out as a study (`thrown=`, `score=`) |
| [`07-anim.pcn`](examples/07-anim.pcn) | The reserved `Anim` keyframe extension |
| [`og2018-women-bronze-gbr-jpn.pcn`](examples/og2018-women-bronze-gbr-jpn.pcn) | A real game with all four layers: the PyeongChang 2018 bronze medal game, converted from the World Curling Shot by Shot PDF and adjusted against video (`Curation "verified"`). Replay it in [CurlFlux](https://curlflux.creco.net/?replay=og2018-women-bronze-gbr-jpn&lang=en). |

The synthetic examples use fictional scores and positions; they exist to show the syntax.

## License

- The specification, grammar, schema and this documentation: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Implement, quote and redistribute freely with attribution ("PCN — Portable Curling Notation, by Shintaro Inagaki").
- Any code in this repository: [MIT](LICENSE-CODE).
- The real-game example is derived from a World Curling / CURLIT Results Book (source URL in its `[Source]` tag). World Curling and CURLIT hold the rights to the underlying record; the file is included as a format example.

## Contributing

Open an issue for anything unclear, missing, or wrong in the spec. Reports of the form "I wrote a parser / converter and hit this" are the most useful. Discussion in English or Japanese.

---

## 日本語

PCN（Portable Curling Notation）は、カーリングの試合を1投1行のテキストで記録する記譜形式です。人が読み書きでき、機械が正確に処理でき、物理シミュレーションで再現できることを目指しています。

- 仕様（正本）: [spec/PCN_SPEC.md](spec/PCN_SPEC.md)　英語版: [spec/PCN_SPEC.en.md](spec/PCN_SPEC.en.md)
- 状態: v1.0 草案（第5版、2026-09-04）。World Curling・CURLIT とは関係のない非公式の文書です
- ライセンス: 仕様・文法・スキーマ・文書は CC BY 4.0、コードは MIT
- 参照実装: [CurlFlux](https://curlflux.creco.net/)（作戦ボード。「PCN を開く」でこの形式のファイルを再生できます）
- 解説記事: Zenn（準備中）

意見や報告は Issue へ。日本語で構いません。
