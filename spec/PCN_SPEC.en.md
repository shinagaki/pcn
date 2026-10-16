# PCN — Portable Curling Notation, specification v1.0 (draft)

**Format 1.0 | Document: draft, 7th revision | Last updated 2026-10-16 | Status: draft**

*This is the English edition of [PCN_SPEC.md](PCN_SPEC.md). The Japanese text is the reference; where the two differ, the Japanese edition prevails. Section numbers match.*

PCN is a text notation for recording a curling game so that it is **readable and writable by people, processable exactly by machines, and reproducible by a physics simulation**. It belongs to the same family of names as PGN (chess) and PDN (draughts).

Rules and statistical symbols follow the public material of World Curling / CURLIT (sources in §11). This specification is an unofficial document with no affiliation to World Curling or CURLIT.

- Extension: `.pcn`. Encoding: UTF-8. Line ending: LF (CRLF is accepted).
- MIME: `text/x-pcn`.
- The specification does not depend on any particular implementation. Layers ①–③ are a common language that lets any processor follow the same board; only layer ④ (`v=`) is engine-specific (§10).
- License: this specification is published under **CC BY 4.0** ([Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/)). Implement, quote and redistribute freely with attribution. Implementations based on it choose their own license.
- Latest version: <https://github.com/shinagaki/pcn> (spec, grammar, JSON schema, examples; the Japanese [PCN_SPEC.md](PCN_SPEC.md) is normative). The copies under <https://curlflux.creco.net/docs/> are mirrors of the same content.

## 1. Design principles

1. **Progressive detail (four layers)** — write only the layers you have, in the same file.

   | Layer | Content | Real sources |
   |---|---|---|
   | ① Linescore | Points per end, LSFE (hammer in end 1), LSD | World Curling results database, national association result pages |
   | ② Description | Thrower, shot type (Task), rotation (Handle), 0–4 rating (Points) | World Curling / CURLIT Shot by Shot (PDF, Game Centre) |
   | ③ Positions | Coordinates of every stone after each shot | Extracted from CURLIT PDF diagrams, coaching apps |
   | ④ Motion | Position, velocity and spin at release | Simulators (Digital Curling etc.), measuring equipment |

   With ④ a deterministic physics engine **replays the game exactly**. How a record without ④ is presented is up to the implementation: it may show ③ (positions) as the board, or generate throws that **fit** ③ or are **consistent with** ② and animate them. When a generated throw is written back, add `v=` together with `Engine`, and mark it `est` as an estimate (§4.2, §10).

   **Only layers up to ③ carry across implementations.** Layer ④ `v=` depends on the physics model: the same release stops in a different place in another engine. Therefore `v=` is always paired with an `Engine` tag, and **a player with a different engine ignores `v=` and rebuilds from ③ (or ②)** (§10). In short: ③ is the common language, ④ is exact reproduction and bit-level verification within one engine. An `Ice` tag (hog-to-hog time, curl) lets another engine calibrate to similar behaviour.
2. **Match World Curling practice** — Task and Points use the definitions and one-letter codes of the World Curling / CURLIT statisticians' manual *Curling Statistics: How to Score*. Only Handle differs: where the manual writes In-turn / Out-turn (`I` / `O`), PCN writes `>` / `<` so the direction of curl is visible at a glance (mapping in §4.2).
3. **Line-oriented and appendable** — one shot per line. Works with `grep` and spreadsheets, and diffs are readable. Because the file reads top to bottom, appending a line per throw gives a live record (an end without a `Score` line is in progress).
4. **Contain existing records rather than replace them** — public records (linescores, World Curling / CURLIT Shot by Shot, coaching-app CSV, simulator logs) each fit one of the layers ①–④. Curling has no public notation comparable to PGN or PDN, so being a container for existing material is a priority.
5. **Implicit values only where derivable** — throwing order (first/second) and thrower position follow from the rules and may be omitted; only exceptions (a skip throwing lead stones etc.) are written explicitly.
6. **World Curling’s rules by default; differences are declared** — the default rules are World Curling’s *Rules of Curling*. Variations used by professional tours and leagues (a limit on blank ends, shootouts, other ways of scoring, and so on) are declared with the `Rules` tag (§6.2, §6.3). Rules are used to derive omitted values and to check the record; **what the record says (`Score`, `hammer=`, `@`) always wins**. A reader that does not know a rule can still read the record as written.

## 2. Overall structure

```
[PCN "1.0"]
[Event "LGT World Men's Curling Championship 2026"]
[Site "Ogden, UT, USA"]
[Venue "Weber County Ice Sheet"]
[Date "2026.04.02"]
[Stage "Round Robin Session 18"]
[Sheet "A"]
[Red "JPN - Japan"]
[Yellow "POL - Poland"]
[RedPlayers "KOIZUMI S; USUI S; YAMAGUCHI T; YANAGISAWA R"]
[YellowPlayers "LOBAZA B; CIEMINSKI M; DOMIN K; STYCH K"]
[Ends "10"]
[LSFE "Red"]
[Result "5-0"]
[Termination "Concede"]

End 1 hammer=R
1 Y F>4
2 R D>4
3 Y D>2
4 R D<4
5 Y D<2
6 R T<3
7 Y S<2
8 R H>3
9 Y H>4
10 R T<2
11 Y D<3
12 R H<4
13 Y P>1
14 R H>3
15 Y S<2
16 R D<4
Score 3-0

End 2 hammer=Y
1 R F>4
...
Score 2-0
```

A file consists of a **header (tags)** and a **body (a sequence of ends)**. Blank lines are free; `;` starts a comment that runs to the end of the line.

## 3. Header (tags)

One `[Key "Value"]` per line. Inside a value, `"` is written `\"` and `\` is written `\\`. Order is free, but `PCN` first is recommended. Unknown tags are preserved and ignored.

| Tag | Required | Meaning |
|---|---|---|
| `PCN` | yes | Specification version. `"1.0"` |
| `Red` / `Yellow` | yes | Team names. Teams are identified by stone colour (World Curling practice). If the actual colours differ, still use Red/Yellow nominally and give the display colour in `RedColor` etc. |
| `Event` | | Event name |
| `Site` | | Location (city, country), e.g. `"Ogden, UT, USA"`. The building goes in `Venue` |
| `Venue` | | Venue (the building), e.g. `"Weber County Ice Sheet"`. If a sponsor renames the building, write the name in use at the time of the game |
| `Date` | | `YYYY.MM.DD` (unknown parts as `??`) |
| `Time` | | Start time `HH:MM` in **venue local time** (World Curling records are local). Infer the time zone from `Site`. Add `TZ` (e.g. `"+09:00"`) only if you need UTC |
| `Stage` | | Round / session (e.g. `"Round Robin Session 18"`, `"Final"`) |
| `Sheet` | | Sheet name |
| `RedPlayers` / `YellowPlayers` | | In throwing order, `;`-separated (lead; second; third; fourth). **By default the third player is the vice-skip and the fourth (last) is the skip.** A function is written after the name only when it differs from that default — `(S)` skip / `(V)` vice-skip (e.g. a skip who throws third). With three players write three names (the first two throw three stones each, the third two — World Curling R3(c)(i)). Mixed doubles: two names |
| `RedShort` / `YellowShort` | | Abbreviation (`JPN` etc., 3–4 letters) |
| `RedColor` / `YellowColor` | | Display colour (`#rrggbb`) |
| `RedCoach` / `YellowCoach` | | Coaches, `;`-separated |
| `RedReserve` / `YellowReserve` | | Alternates, `;`-separated. `p=` and `orderR=` / `orderY=` (§4.1) refer to them with the numbers that follow `Players` (with four `Players`, the first alternate is `5`) |
| `RedSub` / `YellowSub` | | In-game substitutions (optional), e.g. `"E6 TANAKA for SUZUKI Y"`, `;`-separated. Display metadata only; who throws after a substitution is given by the `End` attributes `orderR=` / `orderY=` (§4.1) or per shot with `p=` |
| `Ends` | | Scheduled ends. World Curling championships `10`, mixed doubles `8` (R17(c)). Default `10`; write it explicitly for 8-end games |
| `Stones` | | Stones per team per end (default `8`, mixed doubles `5`) |
| `Format` | | `Team` (default) / `MixedDoubles` / `Wheelchair` / `WheelchairMixedDoubles` / `Mixed`. Mixed doubles: `Stones` defaults to 5, throwing order 1-2-2-2-1 (R17(d)), pre-positioned stones written in each end's `Setup` (R17(f)). `Wheelchair` plays like the four-player game (no sweeping) and is treated the same way here. `WheelchairMixedDoubles` is wheelchair mixed doubles and plays like mixed doubles. `Mixed` is four-player mixed curling with two men and two women (R16) and plays like `Team`. Every value affects rule defaults (§6.1) |
| `FGZ` | | Number of stones during which the protection applies (an integer `0`–`15`; `5` protects through the 5th stone, and stones may be removed from the 6th). World Curling uses `5` for four-player games and `3` for mixed doubles. Default from `Date` and `Format` (§6.1). Mixed-doubles `3` protects different stones from four-player FGZ (§6.1). Events that protect stones differently write it with `fgzmove` / `fgzsides` / `fgzzone` in `Rules` (§6.2) |
| `NoTick` | | `true` / `false`. Opponent FGZ stones on the centre line may not be moved off the line (from 2022-23). **Does not apply to mixed doubles or wheelchair** (R7). Default from `Date` and `Format`. Events that apply it only in some ends write `notickends` in `Rules` (§6.2) |
| `Rules` | | Rules that differ from the defaults (World Curling’s rules), `"key=value key=value …"`: the hammer after a blank end, how a tie is decided, how the game is scored, how FGZ stones are protected, what happens when a team runs out of time, and so on. Keys and defaults in §6.2; how particular events are written in §6.3 |
| `ThinkingTime` | | Thinking time (`"38:00"` for 10 ends, `"30:00"` for 8, `"22:00"` mixed doubles). Time given per end is written `"4:00/end"` |
| `SheetWidth` | | Sheet width in metres (default `4.75` = World Curling maximum; existing facilities may be as narrow as `4.42` (R1); narrow club sheets e.g. `4.28` = 14 ft 2 in) |
| `Ice` | | Ice conditions `"hh=13.8 curl=1.2 turns=5"`. `hh` = hog-to-hog time in seconds of a draw stopping on the tee, `curl` = how far a draw to the tee, released straight along the centre line as its aim, **ends up to the side of the aim line when it stops**, in metres (the total curl from release to rest, measured to the centre of the stopped stone), `turns` = the standard number of turns (how many times a stone rotates from release until it stops; 5 if omitted). `curl` is the value for a draw thrown with `turns` turns (some engines curl differently with the number of turns). Video often measures only the part after the far (house-end) hog line; that value is smaller than `curl` because it leaves out the curl before it, so convert it rather than writing it as is. Players calibrate friction and curl to match `hh` and `curl`, and use `turns` for the rotation of deliveries whose rotation is not given (those a reproducer fills in). Engine-specific multipliers `friction=` `curlx=` may be added |
| `LSFE` | | Team with the hammer (last stone) in end 1: `Red` / `Yellow` |
| `LSD` | | Last stone draw `"Red 286.3; Yellow 199.6"` (cm; outside the house = 199.6) |
| `Result` | | Final score `"Red-Yellow"` (e.g. `"5-0"`). In progress / unknown: `"*"`. A game decided by a shootout keeps the tied score (e.g. `"4-4"`; the winner is derived from `Shootout`) |
| `Termination` | | How the game closed. `Normal` (all scheduled ends played; default) / `Concede` / `Stopped` (cut short before the scheduled ends) / `Forfeit` (no-show, disqualification, out of time) / `Shootout` (tied, then decided by a shootout). **The reason is not part of the value; write it as a comment on the final end's line** (§4.6) |
| `Shootout` | | Result of a shootout (a draw to the button that decides the winner), `"R 23.5; Y 41.0"`: `<side> <distance in cm>` in throwing order, `;`-separated. Distance from the tee to the centre of the stone, measured as for `LSD`; a stone covering the pin so that it cannot be measured is `0`, outside the house is `199.6`. The thrower may follow in parentheses (`R 23.5 (FUJISAWA S)`). If a round is tied and thrown again, keep listing (§4.6) |
| `Timeout` | | Time-outs (optional), `;`-separated "team, end, before which of that team's stones" (counted per team, 1–8 or 1–5 in mixed doubles, not the shot number within the end; the same count as "CHN stone 8" in Results Books. Extra ends use their end number, or `EE`), e.g. `"Y E6 before 5; R E11 before 3"`. Several are possible (one per team per end plus one each in extra ends). A `; time-out` comment on the shot line is also fine (§4.6) |
| `Engine` | | Identifier of the deterministic engine that replays layer ④, as `name/version` (e.g. `"myengine/1"`). Required when `v=` is written |
| `Source` | | Source (URL etc.) |
| `Annotator` | | Recorder |
| `Curation` | | Finishing stage of the data (optional), one ordered ladder `generated` < `adjusted` < `verified` < `reenacted`. `generated` = motion generated automatically from positions (including corrections to the source record) / `adjusted` = positions or throws adjusted by hand after generation (for any reason: comparing with diagrams, fixing physical contradictions, partial video checks) / `verified` = every outcome-relevant throw checked and adjusted against video / `reenacted` = fully reproduced with `Anim` keyframes (video required). Absent = `generated`. Higher stages include lower ones. Regenerating a file at `adjusted` or above would destroy hand work, so a regenerating processor should refuse (or at least warn) |

## 4. Body

### 4.1 Ends

```
End <n> [hammer=R|Y] [key=value ...]
  ...shot lines...
Score <red>-<yellow>
```

- `n` starts at 1. Extra ends continue the numbering (11, 12 …); showing `EE` is up to the player.
- `hammer=` may be omitted; it is then derived from the previous end (the team that did not score gets the hammer; after a blank end it follows `blank` in `Rules`, by default kept in four-player games and passed to the other team in mixed doubles) and from `LSFE`. If they conflict, the explicit value wins with a warning. In games where the hammer moves under non-default rules (§6.3), writing `hammer=` on every end is recommended, for readers that do not know `Rules`.
- Other attributes: `powerplay=R|Y` (mixed doubles power play), `clockR=MM:SS` / `clockY=MM:SS` (thinking time left at the end of the end).
- **Change of ice** `hh=` / `curl=`: ice conditions **from that end on**, with the same meaning as the `Ice` tag (§3): `hh` = hog-to-hog time of a draw to the tee (s), `curl` = how far a draw to the tee aimed down the centre line ends up off the aim line when it stops (m). If only one is written, the other keeps its previous value. The standard number of turns `turns` is one per game and is written only in the `Ice` tag. The engine-specific multipliers `friction=` `curlx=` may be added.
  - Ice changes during a game (heavier early while the pebble is fresh, faster after scraping or as the temperature changes, and so on). Scraping happens between ends, and measurements are taken end by end, so this is an end attribute.
  - Example (faster from end 6): `End 6 hammer=R hh=14.3 curl=1.40`
- **Change of delivery rotation** `orderR=` / `orderY=`: the delivery rotation **from that end on**, as comma-separated player numbers. Players are numbered from `1` in `Players` order, and alternates continue the numbering (the first name in `RedReserve` / `YellowReserve` is the number of `Players` plus one).
  - One number per player: each throws `Stones` divided by the number of players, in order; when it does not divide evenly, earlier players throw more (four players: two each, `orderY=5,2,3,4` = the first alternate throws lead; three players: 3, 3 and 2, `orderY=1,2,4`).
  - Mixed doubles takes two numbers: the first throws stones 1 and 5, the second stones 2 to 4 (R17(d)). The team may change who throws stones 1 and 5 from end to end, so write it on each end where it changes, including an end where it changes back.
  - As many numbers as `Stones`: the thrower of each stone (when a team drops to three players and the first two throw three each, `orderY=1,1,1,2,2,2,4,4`).
  - Under the rules, the delivery rotation and the skip and vice-skip positions may change **only when an alternate is brought in, and only at the start of an end**; the replaced player may not return to the game (WCF R3). That is why this is an end attribute. A single stone thrown by someone else is written with `p=` on that shot line.
  - Example (from end 5, the alternate replaces the first player and throws lead):
    ```
    [YellowPlayers "SCHEUERL J; MESSENZEHL F; KAPP B; MUSKATEWITZ M"]
    [YellowReserve "TREVISIOL M"]
    ...
    End 5 hammer=R orderY=5,2,3,4
    ```
- `Score` is the score of that end. An end not played to completion (concession etc.) is written `Score X` (neither team threw) or `Score 2-X`. Omitting `Score` means the end is not yet decided (in progress).
  - `Score` is the points counted in that end as they stand. In events that add points for a stone covering the pin (`pin` in `Rules`), it includes the bonus. In games decided by ends won or by skins (`scoring` in `Rules`), `Score` is still the points counted in the end, and the game result or the skins are derived from it (§6.2).
- **Scenario attributes**: a record that starts mid-game (a position "to think about the next shot") adds the following to its starting end. They express shot counts and cumulative score **without shot entries**, and are not needed in a full record.
  - `thrown=<R>-<Y>` … stones already thrown by first (R) and second (Y) at the start of the end (e.g. `thrown=7-6` means the next shot is the hammer team's 7th)
  - `score=<R>-<Y>` … cumulative score **at the start** of the end (distinct from the `Score` line = result of the end)
  - When an End has either, the reader starts counting from that end number, score, hammer and shot count. Unknown `key=value` attributes are ignored (like unknown leading words in §4.5).

### 4.2 Shot lines

```
<number> [R|Y] <shot> [key=value ...] [; comment]
```

| Element | Form | Meaning |
|---|---|---|
| number | `1`–`16` (mixed doubles `1`–`10`) | Sequence number within the end, ascending from 1 |
| side | `R` / `Y` | If omitted, **the non-hammer team throws first**, then alternate. Write it only when the order was irregular (e.g. a throwing-order violation) |
| shot | `<Task><Handle><Points><annotation>` | See below. `-` and `X` stand alone as Task |
| `p=` | `1`– | Thrower: index into the `Players` tag (1-based). Alternates continue the numbering after `Players` (§3, `RedReserve`). Default is the end's delivery rotation (`orderR=` / `orderY=`; otherwise the regular order, §6 rule 3) |
| `t=` | seconds | Hog-to-hog time (hog line to hog line, 21.945 m): how fast the stone travels on the ice, which describes the ice, e.g. `t=12.7` |
| `i=` | seconds | Split time (delivery-end back line to hog line, 8.230 m): measured during the delivery, it gives the thrower's release weight, e.g. `i=3.7` |
| `clock=` | `MM:SS` | Thinking time left after the shot |
| `v=` | `x,y,vx,vy,w` | ④ motion (§5.2) |
| `est` | flag | `v=` is not a measurement but an estimate generated from ② / ③ |

#### Task — the World Curling / CURLIT statisticians' codes

| Code | Name | Class |
|---|---|---|
| `D` | Draw | slow |
| `F` | Front (placed short of the house; includes centre / corner guards) | slow |
| `G` | Guard (protecting a specific stone) | slow |
| `R` | Raise / Tap Back | slow |
| `W` | Wick / Split / Soft-peeling | slow |
| `Z` | Freeze | slow |
| `T` | Take-out | fast |
| `H` | Hit and Roll | fast |
| `C` | Clearing (peel) | fast |
| `S` | Double Take-out | fast |
| `P` | Promotion Take-out (run-back) | fast |
| `-` | Through (intentional; no Handle or Points) | — |
| `X` | Not considered (burned stone etc.; no Points) | — |
| `?` | Unspecified (task not assigned; Handle and Points may be given) | — |

Note: in the statisticians' manual `-` (Through) is a Task whose Points are automatically `X`, while `X` (Not considered) is really a **Points value**. PCN keeps both as Task codes for a uniform syntax (`-` = no Handle/Points, `X` = no Points).

Note: `?` (Unspecified) means the Task has not been decided. Use it for a throw that has a position (`@`) but no confirmed type, so that PCN's requirement for a Task does not force a guess. A reproducer fits the throw without a task hint and may fill the type in from the resulting board (the position is authoritative). A throw with an explicit type keeps it and never becomes `?`. The annotation `?` (dubious execution) follows the Task and is distinguished by position.

#### Handle (rotation)

| Code | Meaning |
|---|---|
| `>` | Clockwise (seen from above). In-turn for a right-hander. The stone curls **right** |
| `<` | Counter-clockwise. Out-turn for a right-hander. The stone curls **left** |

Handedness is not recorded, only the rotation of the stone (as World Curling does). Omit if unknown.

#### Points (rating)

`0`–`4` (World Curling 0/25/50/75/100%). Omit if unknown. Historic 5- or 6-point scales are not represented; round to the 4-point scale on import (4 and above become `4`). Rating follows the World Curling / CURLIT statisticians' guidelines (e.g. Take-out 4 = opponent removed and shooter stays, 3 = stays but rolls, 2 = both out, 0 = opponent stays).

World Curling statistics record hog-line and FGZ violations as "Player's Fault" separately from the Task. PCN **writes the intended Task and gives the reason in an `x=` field** (§4.6).

#### Annotations

`!` (good), `!!` (brilliant), `?` (dubious), `??` (bad), `!?`, `?!` may follow Points. They are subjective and independent of the rating.

### 4.3 Board lines (layer ③)

Immediately after a shot line or a `Setup` line, a line starting with `@` gives the coordinates of every stone **after** that shot.

```
3 Y D>2
@ Y0.05,3.42 R-0.21,0.88 Y0.33,-0.12
```

- Items are `<R|Y><x>,<y>` separated by spaces. Coordinates per §5.1. Order is free.
- Only stones in play are written (removed stones are omitted).
- If no stones remain, the line is just `@`.
- A trailing `*` marks the **shooter's final position when it stayed in play** (drawn with a thick black rim in World Curling diagrams). At most one per position. When fitting (③→②) cannot tell which stone was thrown, this mark makes the fitter keep the shooter in play at that spot. Do not mark a position where the shooter went out. Example: `@ R0.10,0.30* Y0.08,2.69`
- `@` means "the board as settled **before the next shot**". Stones repositioned by an umpire, and corrections made from video or photos, belong in this board. `Setup` is never written in the middle of an end (only at the start; §4.4).

### 4.4 Starting from a position (`Setup`)

To start from an arbitrary position (mixed-doubles pre-positioned stones, practice problems, tactical study), write it right after `End`.

```
End 1 hammer=Y
Setup
@ R0.00,3.50 Y0.00,0.30
1 R D<4
```

### 4.5 Engine extension lines

- `Anim <stone> [dur=<s>] <t>:<x>,<y> [<t>:<x>,<y> …]` — **hand-authored animation (reserved, provisional)**. Describes stone motion that physics cannot reproduce (an unnatural path transcribed from video) as keyframes per stone. One line per hand-animated stone per throw.
  - **Stone id** `<stone>` = `<R|Y><number>`. `0` is **the stone thrown in this shot**; `1` and up count existing stones of that colour **in the order of the previous board line (`@` / `Setup`)**. Each keyframe's `<x>,<y>` is in the §5.1 coordinates (m). The `t=0` keyframe must equal the stone's start position (the release point for `0`) as a check on identity.
  - **Time** `<t>` is normalised 0–1 (0 = start of throw, 1 = stop). Real duration is `dur=<s>` (seconds from 0 to 1; implementation default if omitted). Times ascend, first `0`, last `1`.
  - **Interpolation** between keyframes is **centripetal Catmull-Rom (α = 0.5)**; end tangents use duplicated end keyframes. The method is fixed so every implementation draws the same curve.
  - **Stones without `Anim`** in that throw are **stationary** (no physics); the final board is the post-shot `@`. Physics and animation are not mixed within one throw.
  - **Stones leaving play**: a stone absent from the post-shot `@` is removed at `t=1` (letting the last keyframe exit the sheet looks natural).
  - A shot with `Anim` is replayed from it in preference to `v=` (④), and `@` is its **final board (t=1)**. Implementations that do not support it ignore the line and replay from `@` (③) / `v=` (④).
- Future extensions are identified by their leading word. Lines with an unknown leading word are preserved and ignored (an implementation without `Anim` can still replay from `@` / `v=`).

Example (only the shooter is animated, with a curve physics would not produce, stopping after 2.4 s):

```
9 R S>3
Anim R0 dur=2.4 0:0.10,30.8 0.5:0.30,4.0 0.8:0.55,0.8 1:0.45,-0.1
@ Y-0.02,3.5 R0.45,-0.1
```

### 4.6 Irregularities and edge cases

Two principles: **outcomes are expressed by the `@` board** (which can represent any position) / **reasons are expressed by a `; comment` or an `x=` field**. This covers violations, special rulings and irregular progress. Unknown `key=value` pairs and comments round-trip unchanged (implementations must not drop keys they do not know).

**Violations** — add `x=<code>` and write the **board after the ruling** in `@`:

| Code | Meaning | Ruling (reflected in `@`) |
|---|---|---|
| `x=fgz` | Protected-stone violation (four-player: an opponent FGZ stone put out of play before the 6th stone; mixed doubles: any stone, own or opponent, put out of play before the 4th stone) | Moved stones returned, shooter removed. If the `FGZ` rule is in effect the player detects and restores it automatically, so `@` may be omitted |
| `x=notick` | No-tick violation (an opponent FGZ stone on the centre line moved off the line; not in mixed doubles or wheelchair) | The non-offending team chooses (restore / leave). Write the chosen result in `@` |
| `x=hog` | Hog-line violation (late release), and a stone that did not reach the hog line | Shooter removed (not written in `@`) |
| `x=burn` | Burned stone (touched while moving) | The opponent's choice (leave / restore / remove) reflected in `@`. Details in a `; comment` |
| `x=reposition` | Stones repositioned by an umpire | The repositioned board in `@` |
| `x=wrong` | Wrong stone or wrong order | Result of the ruling in `@`; details in a comment |
| `x=time` | A stone not thrown because the team ran out of thinking time (in events where running out of time does not lose the game but stops the team from throwing: `timeout=stop` in `Rules`, §6.2) | The board does not change. Write the task as `-` (e.g. `15 R - x=time`). The other team keeps throwing its remaining stones |

Example: `5 R T<0 x=fgz ; opponent centre guard removed → restored`, followed by the post-ruling `@`.

`x=hog` covers two cases that the rules treat differently: a violation (late release) and a missed shot (a stone that did not reach the hog line and
was removed). In both the only outcome is that the shooter is removed, and Shot by Shot records do not separate them, so PCN uses one code. Write the
difference in a `; comment` when it matters (e.g. `1 R D>0 x=hog ; did not reach the hog line`). A player may show the shooter stopping short of the
hog line and being removed (without the code, a shooter that vanished without changing the board cannot be told apart from a through).

**Game-level termination and forfeits** (`Termination` tag):

The value only says **how the game closed**, in five ways. **The reason is not in the value; it goes in a comment on the final end's line.** Reasons are endless, so no codes are added for them.

| Value | Meaning | Winner |
|---|---|---|
| `Normal` | All scheduled ends played (default; may be omitted) | Total score (or, when `scoring` in `Rules` is not `stones`, that way of scoring; §6.2) |
| `Concede` | One team conceded | The other team |
| `Stopped` | Cut short before the scheduled ends | **Score at the stop; a tie is a draw** |
| `Forfeit` | Decided by something other than play (no-show, disqualification, out of time) | The `Result` tag |
| `Shootout` | Ended tied and was decided by a shootout | The distances in the `Shootout` tag |

A **shootout** decides the winner with a draw to the button instead of extra ends (the Grand Slam of Curling, Rock League and others; §6.3). How a tied game is decided is declared with `tie` in `Rules` (§6.2).

- Write `Termination "Shootout"` and the `Shootout` tag. `Result` keeps the tied score (e.g. `"4-4"`), and the `Score` lines end with the scheduled ends, with no extra end.
- The `Shootout` tag lists `<side> <distance in cm>` in throwing order, `;`-separated (§3). Read one stone from each team as a round; the shorter distance wins. A round with equal distances (both teams covering the pin, for example) is decided by the next round.
- Example (yellow threw first, red won): `[Shootout "Y 41.0; R 23.5"]`. Both teams covered the pin and the second round decided it: `[Shootout "R 0; Y 0; R 12.0; Y 30.1"]`
- A way to write each shootout stone’s position and motion (layers ③ and ④) in the body is not defined yet (a future extension; under the first-word rule in §4.5, unknown lines are kept and ignored).

- With `Concede` / `Stopped` / `Forfeit`, unplayed ends are written `Score X` (neither team threw) or `Score 2-X`. `Result` carries the deciding score.
- **The score of a conceded end** follows World Curling's rule R12(i). If both teams still have stones to deliver, write `X`. If only one team has delivered
  all its stones and the team that still has stones to deliver has stone(s) counting, write those points (e.g. the hammer team counts one with its last stone
  still to come → `Score 0-1`). If the team that delivered all its stones is counting, or no stone is counting, write `X`. When the game has an official
  record, write the official score as it is (World Curling records include cases where the team that delivered all its stones was given its points).
- Example of a stop (an event time limit after end 7):

```
[Ends "8"]
[Result "5-4"]
[Termination "Stopped"]
…
End 7 ; game reached the event time limit (2.5 hours); last end
…
Score 1-0
```

- Any reason is acceptable (time limit, facility failure, weather, opponent absent …). Readers decide handling from `Termination` and show the reason as a comment.
- Running out of thinking time is `Forfeit`, not `Stopped`, because it decides a loser.
- Blank end: `Score 0-0`. The hammer for the next end follows `blank` in `Rules` (by default kept in four-player games and passed to the other team in mixed doubles).
- Extra ends: continue `End n` beyond the scheduled count (`Ends` is the schedule; actual ends may exceed it). Events that limit the number of extra ends, or decide a tie by a shootout or a draw instead, write it with `tie` in `Rules` (§6.2).
- Draw: a game that ended tied with `Termination "Stopped"` is a draw (`Result "4-4"`). There is no dedicated draw value, because how the game ended and the result are separate axes.

**In progress / undecided**: omitting `Score` makes the end undecided (a live record mid-end).

**Other**: measurements need nothing extra (the result shows in `@` / `Score`; comment if needed). Time-outs: `; time-out` on the shot line where they were taken, or listed in the `Timeout` tag (§3). Power play: `End … powerplay=R|Y`.

## 5. Coordinates and motion

### 5.1 Coordinate system

- Unit: **metres**. Two decimals (1 cm) standard, up to four if needed.
- Origin: **the tee (centre of the button) of the house in play**.
- `+x`: to the thrower's right. `+y`: toward the thrower (the hack). Guards have `y > 0`; the back of the house (back-line side) has `y < 0`.
- Reference dimensions (World Curling): 12 ft circle radius 1.829, 8 ft 1.219, 4 ft 0.610, button 0.152, hog line `y = 6.401`, back line `y = −1.829`, sheet width 4.75 (`|x| ≤ 2.375`). Stone radius 0.1455.
- Conversion from Digital Curling (origin at the hack, `y` toward the house): `x_pcn = x_dc`, `y_pcn = 38.405 − y_dc` (hack to tee 38.405 m).

### 5.2 Motion `v=x,y,vx,vy,w`

| Element | Unit | Meaning |
|---|---|---|
| `x,y` | m | Stone centre at the start of the record (release), in the §5.1 system |
| `vx,vy` | m/s | Velocity vector |
| `w` | rad/s | Angular velocity, **positive = counter-clockwise** seen from above (a `>` stone is negative) |

A shot with `v=` replayed by the deterministic engine named in `[Engine ...]` reaches the same board even if layer ③ is omitted. A different engine should ignore `v=` and rebuild from ③ or ②.

## 6. Derivation rules

1. **Throwing side**: from the `End` `hammer`, the first stone is thrown by the non-hammer team, then alternately. Lines with an explicit `R`/`Y` are followed; the next omitted line is thrown by "the team with fewer stones thrown".
2. **Hammer**: `LSFE` → after each `Score`, the team that did not score. After a blank (0-0) it follows `blank` in `Rules` (§6.2). By default the four-player game keeps it. In mixed doubles, the team that delivered the first stone of the blank end decides the placement of the positioned stones in the next end (R17(g)III) and normally uses that to take the hammer, so the hammer is treated as passing to the other team (a blank caused by an equal measure leaves the decision, and so the hammer, unchanged). Ends with `X` do not change it. A written `hammer=` always wins.
3. **Thrower**: in the four-player game, `Stones` divided by the number of `Players`, in order; when it does not divide evenly, earlier players throw more (two each for four players; 3, 3 and 2 for three). Mixed doubles: 1-2-2-2-1 (the first player throws stones 1 and 5). From an end with `orderR=` / `orderY=` on, that rotation (§4.1). `p=` overrides.
4. **Percentage**: `Points × 25`. Team / player percentage = `Σpoints ÷ (4 × rated shots)`. `-` and `X` are excluded from the denominator.
5. **Final score and winner**: sum of `Score` lines. The winner is decided by the total, or, when `scoring` in `Rules` is not `stones`, by that way of scoring (§6.2). A game with `Termination "Shootout"` is decided by the distances in the `Shootout` tag (§4.6). If it conflicts with `Result`, `Result` wins with a warning.

### 6.1 Default rules by era and format

Without `FGZ` / `NoTick` tags, the rules of the time are inferred from `Date` and `Format`, using **World Curling (international) adoption dates**. National, club and professional-tour events that differed should tag explicitly (professional examples in §6.3). The defaults of the `Rules` keys (§6.2) are World Curling’s rules regardless of era.

**Four-player, wheelchair and four-player mixed** (`Format` `Team` / `Wheelchair` / `Mixed`)

| Period | `FGZ` | `NoTick` | Basis |
|---|---|---|---|
| to June 1993 | 0 | false | Before FGZ (prototype: 1990-91 Moncton Rule, trialled at the 1992 Olympics) |
| from July 1993 | 4 | false | WCF introduced the four-rock FGZ in international play from 1993-94 |
| from July 2018 | 5 | false | Decided at the September 2017 WCF Congress, five-rock from 2018-19 |
| from July 2022 | 5 | true | No-tick applied from 2022-23 (not wheelchair) |

Canadian national championships alone used **three rocks** from 1993-94 to 2001-02 (unified to four in 2002-03). Since that differs from the international basis it is not a date default; write `FGZ "3"` explicitly.

**Mixed doubles and wheelchair mixed doubles** (`Format` `MixedDoubles` / `WheelchairMixedDoubles`) use the following regardless of era.

| `FGZ` | `NoTick` | Basis |
|---|---|---|
| 3 | false | R17(e) (protection for the first three stones) / R7 (no-tick does not apply) |

#### What is protected differs by format (important)

The same `FGZ` tag protects different things in the four-player game and in mixed doubles.

- **Four-player and wheelchair (R6)**: before the 6th stone of the end, putting an **opponent's** stone that is in the FGZ (between the tee line and the hog line of the playing end, house excluded) **out of play** is a violation. Own stones may be removed, and stones in the house are not covered. The ruling is **mandatory**: remove the shooter, return the moved stones.
- **Mixed doubles (R17(e))**: before the 4th stone, putting **any already thrown or pre-positioned stone, own or opponent**, **out of play** is a violation. Location is not limited to the FGZ. Same ruling as the four-player game.
- **No-tick (R7)**: before the 6th stone, moving an **opponent's** FGZ stone that touches the centre line off the line or out of the FGZ is a violation. The ruling is **the non-offending team's choice** (① remove the shooter and restore, ② leave). A stone put fully out of play falls under R6(b). **Not applied to mixed doubles or wheelchair.**

PCN records the throw as it was; the player judges the outcome. A player may implement ① (restore) on a violation. When replaying a recorded game, use the recorded `@` boards as they are, without applying the judgement (the ruling of the day is already reflected; re-judging with estimated physics causes false positives). Violations preserved in a record can be marked with `x=fgz` / `x=notick` (§4.6).

Events that change the protection itself (forbidding any movement within play, protecting a team's own stones as well, and so on) write it with `fgzmove` / `fgzsides` / `fgzzone` in `Rules` (§6.2).

### 6.2 Declaring rules (`Rules`)

A game played under rules that differ from World Curling's declares the differences with the `Rules` tag.

```
[Rules "blank=keep1 tie=shootout"]
```

- The value is a list of `key=value` pairs separated by spaces. Keys are lowercase ASCII and values contain no spaces. Order is free, and each key is written once.
- A key that is not written takes its default (World Curling's rules; see the table below). A game without a `Rules` tag uses every default.
- The number of ends and stones, the format, the FGZ count and no-tick are still written with the `Ends` / `Stones` / `Format` / `FGZ` / `NoTick` tags. `Rules` carries the other differences.

**How readers handle it**

1. **The record wins** — `Score`, `hammer=`, `@`, `Result`, `Termination` and `Shootout` take precedence over `Rules`. `Rules` is used to derive omitted values (the hammer, the winner), to check the record (the points counted from the board against `Score`), and for the player's own judgements (automatic detection of violations, for example).
2. **Unknown keys and values are kept and ignored** — reading does not stop; the user is told that there is a rule the reader does not know (a warning). When writing, unknown keys are kept as they are.
3. **Writers make the record explicit** — in a game where the hammer or the winner moves under non-default rules, write `hammer=` on every end, together with `Result` and `Termination`, so that a reader that does not know `Rules` still reads the record correctly.

**Keys**

| Key | Values | Default | Meaning |
|---|---|---|---|
| `blank` | `keep` / `switch` / `keep<N>` | `keep` for four-player games, `switch` for mixed doubles | The hammer after a blank end (0-0). `keep` = kept. `switch` = passes to the other team (in mixed doubles, the decision on the positioned stones passes; §6, rule 2). `keep<N>` = each team keeps the hammer on its first N blank ends of the game (extra ends included), and loses it from the N+1th |
| `tie` | `extra` / `extra<N>` / `shootout` / `draw`, joined with `+` | `extra` | How a game tied after the scheduled ends is decided, in the order the methods apply. `extra` = extra ends until decided. `extra<N>` = up to N extra ends. `shootout` = a shootout (§4.6). `draw` = the game ends as a draw. Example `extra1+shootout` = one extra end, then a shootout if still tied |
| `scoring` | `stones` / `ends` / `skins` | `stones` | How the winner is counted. `stones` = total score. `ends` = number of ends won (match play). `skins` = total value of the skins won. What it takes to "win" an end is `win` |
| `win` | integer, 1 or more | `1` | With `scoring` `ends` / `skins`: the points the team with the hammer needs to win the end (the skin). The team without the hammer wins it by scoring at all (a steal). An end where neither happens is a "push", won by nobody |
| `skins` | value of each end, `,`-separated | none | Skin values for `scoring=skins`. Ends beyond the list (extra ends) use the last value. A skin nobody wins carries over to the next end, and the team that next wins a skin also takes the carried value |
| `pin` | `<points>@<ends>` | none | In the given ends, a counting stone that covers the pin (the hole at the centre of the tee) counts as `<points>` (normally 1). `<ends>` is `last` (the final scheduled end), `all`, or end numbers, `,`-separated. Example `pin=2@last` |
| `fgzmove` | `out` / `any` | `out` | What is forbidden for a stone the FGZ protects. `out` = putting it out of play (R6, R17(e)). `any` = moving it at all (moving it within play is also a violation) |
| `fgzsides` | `opp` / `all` | `opp` for four-player games, `all` for mixed doubles | Which stones are protected. `opp` = the opponent's (R6). `all` = both teams' stones already delivered and the positioned stones (R17(e)) |
| `fgzzone` | `fgz` / `play` | `fgz` for four-player games, `play` for mixed doubles | Where a protected stone must be. `fgz` = the free guard zone (between the tee line and the hog line of the playing end, house excluded). `play` = anywhere in play (R17(e)) |
| `notickends` | `all` / `last` / end numbers, `,`-separated | `all` | The ends in which `NoTick` applies. `last` = the final scheduled end and extra ends |
| `timeout` | `forfeit` / `stop` | `forfeit` | When a team uses up its thinking time. `forfeit` = it loses the game (`Termination "Forfeit"`). `stop` = it may not deliver any more stones (stones not thrown are task `-` with `x=time`; §4.6), and the other team keeps throwing its remaining stones |

**Scoring and the winner** (`scoring`, `win`, `skins`, `pin`)

- Whatever the way of scoring, a `Score` line holds **the points counted in that end** (including any `pin` bonus; §4.1).
- With `scoring=ends` / `skins`, the winner is derived from `Score` and the hammer of each end. If the team with the hammer scores at least `win` points, it wins the end (the skin); if the team without the hammer scores, it wins the end (the skin).
- `Result` holds the result in the game's own way of scoring (`stones`: total score; `ends`: ends won; `skins`: total value of the skins won).

**Adding keys**

Rules keep changing from event to event, league to league and year to year. New rules are taken in by adding keys to this table. The meaning of an existing key is never changed (a changed meaning gets a new key). When you meet a rule the keys cannot express, write it in a `; comment` first and propose a new key.

### 6.3 Rule variations by event

Some professional tours and leagues play under rules that differ from World Curling's. With the keys of §6.2 they are written as follows.

- Values change by event, year and round. **Check them against the conditions of the game you are recording.** The tables show values confirmed as of October 2026.
- Keys equal to their defaults are left out.
- In a game where the hammer or the winner moves differently from the defaults, write `hammer=`, `Result` and `Termination`, as §6.2 ("Writers make the record explicit") says.

**Grand Slam of Curling (GSOC)**

| Period / event | How to write it | Notes |
|---|---|---|
| 2026-27 season (GSOC Invitational) | `[Ends "8"]` `[FGZ "5"]` `[NoTick "true"]` `[ThinkingTime "29:00"]` `[Rules "blank=keep1 tie=shootout"]` | Shootouts in the round robin, tiebreakers and quarterfinals; extra ends in the semifinals and final (leave `tie` out). Two 60-second timeouts |
| 2025-26 season | `[Ends "8"]` and `[Rules "blank=keep1 tie=shootout"]`, and so on | Shootouts began in the round robin at the Masters, were extended to the playoffs at the Tour Challenge and Tahoe, and gave way to extra ends again for tiebreakers and playoffs at the Canadian Open and the Players' Championship. The blank-end limit started at the Tour Challenge. Check `tie` for each event and round |
| about 2014–2018 | write `[FGZ "5"]` explicitly | The five-rock rule was used before World Curling adopted it. The date default (§6.1) would give 4 |
| 2019 Champions Cup | `[NoTick "true"]` `[Rules "notickends=last"]` | No-tick was trialled in the eighth end and extra ends only |
| Elite 10 (about 2015–2019; match play) | `[Ends "8"]` `[FGZ "5"]` `[NoTick "true"]` `[ThinkingTime "4:00/end"]` `[Rules "scoring=ends win=2 blank=switch tie=shootout"]` | Decided by ends won. The hammer team wins an end with two or more, the other team with a steal; otherwise it is a push and the hammer passes |

**Rock League (2026, its first season)**

| Game | How to write it | Notes |
|---|---|---|
| Four-player (men's, women's) | `[Ends "7"]` `[FGZ "5"]` `[ThinkingTime "21:00"]` `[Rules "fgzmove=any fgzsides=all blank=keep1 tie=shootout pin=2@last timeout=stop"]` | No stone of either team in the FGZ may be moved through the 5th stone (they may be moved from the 6th). In the final end, a stone covering the pin counts two. Running out of time does not lose the game but stops the team throwing. Three 60-second timeouts |
| Mixed doubles | `[Format "MixedDoubles"]` `[Ends "8"]` `[ThinkingTime "23:00"]` `[Rules "fgzmove=any fgzzone=fgz tie=shootout pin=2@last timeout=stop"]` | No stone in the FGZ may be moved through the 3rd stone (from the 4th they may; `FGZ` keeps its default `3`). A blank end loses the hammer (the default `switch`) |
| Four-player mixed (Mixed Fours) | `[Format "Mixed"]` with the same `Rules` as four-player games | Men and women throw alternately |

In a shootout, any player on the ice may throw; the team that would have had the hammer in an extra end chooses the order, and if it is still tied the same players throw again without sweeping. The 2027 rules had not been published as of October 2026.

**Skins** (skins games, such as at the Continental Cup)

Write `[Rules "scoring=skins win=2 skins=…"]`. Skin values and the handling of extra ends differ by event and year, so list each end's value in `skins` (e.g. `skins=1,1,2,2,3,3,4,4` — an illustration, not the values of a real event).

## 7. JSON representation (PCN-JSON)

For program-to-program exchange, a JSON form maps one-to-one to the text.

```json
{
  "pcn": "1.0",
  "tags": { "Event": "...", "Red": "...", "Yellow": "..." },
  "ends": [
    {
      "no": 1, "hammer": "R",
      "setup": null,
      "shots": [
        { "no": 1, "side": "Y", "task": "F", "handle": ">", "points": 4,
          "player": 1, "hogToHog": null, "board": [["Y", 0.05, 3.42]],
          "v": null, "est": false, "nag": "", "comment": "" }
      ],
      "score": { "red": 3, "yellow": 0 }
    }
  ]
}
```

- `handle` is `">"` / `"<"` / `null`; `points` is an integer or `null`; `task` is a string including `"-"` and `"X"`.
- An end may carry `powerplay` (`"R"` / `"Y"` / `null`) and `clockR` / `clockY` (strings).
- `board` is an array of `[side, x, y]`; `v` is `[x, y, vx, vy, w]`.
- `X` in `score` is `null` (e.g. `{ "red": 2, "yellow": null }`).
- `Rules` and `Shootout` go into `tags` as strings, like every other tag (parsing the value is up to the implementation). A shot's `x` may be `"time"`.

## 8. Grammar (EBNF)

```
file      = { line } ;
line      = tag | end | score | setup | shot | board | comment | empty ;
tag       = "[" key WS string "]" ;
end       = "End" WS int { WS kv } ;
score     = "Score" WS ( "X" | num "-" num ) ;          (* num is int or "X" *)
setup     = "Setup" ;
shot      = int [ WS side ] WS shotspec { WS ( kv | "est" ) } ;
board     = "@" { WS stone } ;
stone     = side coord "," coord [ "*" ] ;              (* trailing * = shooter that stayed in play *)
side      = "R" | "Y" ;
shotspec  = task [ handle ] [ points ] { nag } | "-" | "X" ;
task      = "D"|"F"|"G"|"R"|"W"|"Z"|"T"|"H"|"C"|"S"|"P"|"?" ;   (* "?" = unspecified *)
handle    = ">" | "<" ;
points    = "0"|"1"|"2"|"3"|"4" ;
nag       = "!" | "?" ;
kv        = key "=" value ;                                (* value contains no whitespace *)
comment   = ";" { any } ;

(* inside tag values *)
rules     = kv { WS kv } ;                                 (* [Rules "…"]; keys are lowercase ASCII. §6.2 *)
shootout  = sotry { ";" WS sotry } ;                       (* [Shootout "…"], in throwing order *)
sotry     = side WS number [ WS "(" name ")" ] ;          (* distance in cm; 0 = covering the pin, 199.6 = outside the house *)
```

A `; comment` may end any line. Case is significant.

## 9. Examples

### 9.1 Layer ② only (transcribed from World Curling Shot by Shot)

```
End 3 hammer=R
1 Y F>4
2 R D>4
3 Y T<4
```

For a complete example from tags to body see §2.

### 9.2 With layer ③ (coordinates taken from diagrams)

```
End 3 hammer=R
1 Y F>4
@ Y-0.02,3.61
2 R D>4
@ Y-0.02,3.61 R0.08,0.41
3 Y T<4
@ Y-0.02,3.61 Y0.31,-0.55
```

### 9.3 With layer ④ (a game recorded with a deterministic engine)

```
[Engine "myengine/1"]
...
1 Y F>4 t=13.1 v=0.0000,30.8000,0.0124,-2.3315,-0.8000
@ Y-0.02,3.61
```

## 10. Playback guidelines (for implementers)

1. `v=` present and `Engine` **matches yours** → replay the physics as is (exact reproduction).
2. `v=` present but `Engine` **differs** → do not use `v=`. A different physics model stops the same release elsewhere, so re-fit from `@` as in 3 (calibrate your engine to the `Ice` tag first if present).
3. `@` present → **fit** a throw from the difference between consecutive boards (stopping point for draws, contact outcome for hits). The resulting `v=` may be saved with `est`.
4. Layer ② only → generate a throw consistent with Task, Handle, Points and the board. Lower Points mean farther from the target. `-` passes without contact; `X` leaves the board unchanged.
5. Show generated or fitted throws as estimates in the UI (e.g. with `≈`).
6. **Do not apply rule-violation judgement when replaying a recorded game.** The ruling of the day is already in the record; re-judging with estimated physics causes false positives (§6.1).
7. When reading a live record, treat a final end without `Score` as in progress and expect lines to be appended.
8. Unknown keys or values in `Rules` do not stop reading. Use the record (`Score`, `hammer=`, `@`, `Result`, `Termination`, `Shootout`) as it is, and tell the user that there is a rule the reader does not know (§6.2).

## 11. Sources

Primary material this specification follows. The specification is unofficial and not affiliated with World Curling or CURLIT; where interpretations differ, the originals prevail.

| Content | Source |
|---|---|
| Rules of play (score of a conceded end = R12(i), FGZ = R6, no-tick = R7, mixed doubles = R17, wheelchair = R14) | *The Rules of Curling and Rules of Competition*, World Curling, 1 September 2026 ([list](https://worldcurling.org/rules/) / [PDF](https://worldcurling.org/wp-content/uploads/2026/10/Rules-2026_Clean.pdf)) |
| Definitions of Task, Handle and Points | *Curling Statistics: How to Score*, © World Curling and CURLIT Ltd., 2025 ([PDF](https://curlit.com/powerpoint/StatsTraining_2025_2026.pdf)) |
| Source data for layers ② and ③ (Shot by Shot, diagrams) | World Curling Live Scores (`livescores.worldcurling.org`) / CURLIT Results Books (`curlit.com`) |
| History of the FGZ and national adoption dates | Curling Canada "History of Curling"; World Curling announcements (five-rock decision at the September 2017 Congress, 2022 no-tick trial) |
| Grand Slam of Curling rule variations (§6.3) | The "Eight Ends" column on thegrandslamofcurling.com ([2026 GSOC Co-op Invitational](https://www.thegrandslamofcurling.com/news/eight-ends-everything-you-need-to-know-for-the-2026-gsoc-co-op-invitational), [stats from the rule changes](https://www.thegrandslamofcurling.com/news/eight-ends-intriguing-stats-from-gsocs-rule-changes)), [match play rules](https://thegrandslamofcurling.com/match-play-rules), TSN "Grand Slam of Curling continues to tweak shootout, extra end experiment" (December 2025) |
| Rock League rule variations (§6.3) | Rock League "[Format](https://rockleague.com/format)" (2026 rules), "[Eight Ends: The lowdown on Rock League's format and rules](https://rockleague.com/posts/eight-ends-the-lowdown-on-rock-leagues-format-and-rules)" |

Rule excerpts (1 September 2026 edition, paraphrased):

- **R6(b)** — before the 6th stone of an end, if a delivered stone directly or indirectly moves an opponent's FGZ stone out of play, the delivered stone is removed and the moved stones are returned.
- **R7** — before the 6th stone, if an opponent's FGZ stone touching the centre line is moved off the line or out of the FGZ, the non-offending team chooses between "remove and restore" and "leave". **Not applied to wheelchair or mixed doubles.**
- **R17(e)** — before the 4th stone, if a delivered stone directly or indirectly moves **any delivered or pre-positioned stone** out of play, the delivered stone is removed and the moved stones are returned.
- **R17(c)(d)** — eight ends per game, five stones per team per end. The player who throws the first stone also throws the last; the other throws stones 2–4.
- **R17(g)III** — if neither team scores in an end, the team that delivered the first stone in that end has the decision on placement in the next end. After a blank end caused by an equal measure, the decision does not change.

## 12. Change log

The `[PCN "1.0"]` value in files names the format family and is not revised while the document is a draft. The document is managed by draft revision number; format 1.0 is frozen when the content is settled (incompatible changes afterwards become 2.0).

| Document | Date |
|---|---|
| Draft, 7th revision | 2026-10-16 (the `Rules` tag for declaring rule variations [§6.2] and how particular events are written [§6.3: the Grand Slam of Curling, Rock League, skins]; shootouts [`Termination "Shootout"`, the `Shootout` tag]; `x=time` for stones not thrown after running out of time; `WheelchairMixedDoubles` and `Mixed` for `Format`; `FGZ` as an integer 0–15; per-end `ThinkingTime`; the hammer after a blank end in mixed doubles written out [R17(g)III]. Existing files keep their meaning) |
| Draft, 6th revision | 2026-09-29 |
| Draft, 5th revision (English edition added) | 2026-09-05 |
| Draft, 4th revision | 2026-09-01 |
| Draft, 3rd revision | 2026-08-25 |
| Draft, 2nd revision | 2026-08-23 |
| Draft, 1st revision | 2026-08 |
