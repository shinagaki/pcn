# PCN — Portable Curling Notation, specification v1.0 (draft)

**Format 1.0 | Document: draft, 5th revision | Last updated 2026-09-05 | Status: draft**

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
| `RedReserve` / `YellowReserve` | | Alternates, `;`-separated |
| `RedSub` / `YellowSub` | | In-game substitutions (optional), e.g. `"E6 TANAKA for SUZUKI Y"`, `;`-separated. Display metadata only; the actual thrower after a substitution is given per shot with `p=` |
| `Ends` | | Scheduled ends. World Curling championships `10`, mixed doubles `8` (R17(c)). Default `10`; write it explicitly for 8-end games |
| `Stones` | | Stones per team per end (default `8`, mixed doubles `5`) |
| `Format` | | `Team` (default) / `MixedDoubles` / `Wheelchair`. Mixed doubles: `Stones` defaults to 5, throwing order 1-2-2-2-1 (R17(d)), pre-positioned stones written in each end's `Setup` (R17(f)). `Wheelchair` plays like the four-player game (no sweeping) and is treated the same way here; it affects rule defaults (§6.1) |
| `FGZ` | | Number of protected stones `0` / `3` / `4` / `5`. Default from `Date` and `Format` (§6.1). Mixed-doubles `3` is a different rule from four-player FGZ (§6.1) |
| `NoTick` | | `true` / `false`. Opponent FGZ stones on the centre line may not be moved off the line (from 2022-23). **Does not apply to mixed doubles or wheelchair** (R7). Default from `Date` and `Format` |
| `ThinkingTime` | | Thinking time (`"38:00"` for 10 ends, `"30:00"` for 8, `"22:00"` mixed doubles) |
| `SheetWidth` | | Sheet width in metres (default `4.75` = World Curling maximum; existing facilities may be as narrow as `4.42` (R1); narrow club sheets e.g. `4.28` = 14 ft 2 in) |
| `Ice` | | Ice conditions `"hh=13.8 curl=1.2"`. `hh` = hog-to-hog time in seconds of a draw stopping on the tee, `curl` = its curl in metres. Players calibrate friction and curl to match. Engine-specific multipliers `friction=` `curlx=` may be added |
| `LSFE` | | Team with the hammer (last stone) in end 1: `Red` / `Yellow` |
| `LSD` | | Last stone draw `"Red 286.3; Yellow 199.6"` (cm; outside the house = 199.6) |
| `Result` | | Final score `"Red-Yellow"` (e.g. `"5-0"`). In progress / unknown: `"*"` |
| `Termination` | | How the game closed. `Normal` (all scheduled ends played; default) / `Concede` / `Stopped` (cut short before the scheduled ends) / `Forfeit` (no-show, disqualification, out of time). **The reason is not part of the value; write it as a comment on the final end's line** (§4.6) |
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
- `hammer=` may be omitted; it is then derived from the previous end (the team that did not score gets the hammer; a blank keeps it) and from `LSFE`. If they conflict, the explicit value wins with a warning.
- Other attributes: `powerplay=R|Y` (mixed doubles power play), `clockR=MM:SS` / `clockY=MM:SS` (thinking time left at the end of the end).
- `Score` is the score of that end. An end not played to completion (concession etc.) is written `Score X` (neither team threw) or `Score 2-X`. Omitting `Score` means the end is not yet decided (in progress).
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
| `p=` | `1`–`4` | Thrower: index into the `Players` tag (1-based). Default is the regular order (two stones each) |
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

Example: `5 R T<0 x=fgz ; opponent centre guard removed → restored`, followed by the post-ruling `@`.

`x=hog` covers two cases that the rules treat differently: a violation (late release) and a missed shot (a stone that did not reach the hog line and
was removed). In both the only outcome is that the shooter is removed, and Shot by Shot records do not separate them, so PCN uses one code. Write the
difference in a `; comment` when it matters (e.g. `1 R D>0 x=hog ; did not reach the hog line`). A player may show the shooter stopping short of the
hog line and being removed (without the code, a shooter that vanished without changing the board cannot be told apart from a through).

**Game-level termination and forfeits** (`Termination` tag):

The value only says **how the game closed**, in four ways. **The reason is not in the value; it goes in a comment on the final end's line.** Reasons are endless, so no codes are added for them.

| Value | Meaning | Winner |
|---|---|---|
| `Normal` | All scheduled ends played (default; may be omitted) | Total score |
| `Concede` | One team conceded | The other team |
| `Stopped` | Cut short before the scheduled ends | **Score at the stop; a tie is a draw** |
| `Forfeit` | Decided by something other than play (no-show, disqualification, out of time) | The `Result` tag |

- With `Concede` / `Stopped` / `Forfeit`, unplayed ends are written `Score X` (neither team threw) or `Score 2-X`. `Result` carries the deciding score.
- **The score of a conceded end** follows World Curling's rule R12(h). If both teams still have stones to deliver, write `X`. If only one team has delivered
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
- Blank end: `Score 0-0` (hammer kept).
- Extra ends: continue `End n` beyond the scheduled count (`Ends` is the schedule; actual ends may exceed it).
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
2. **Hammer**: `LSFE` → after each `Score`, the team that did not score. A blank (0-0) keeps it. Ends with `X` do not change it.
3. **Thrower**: `Stones` divided by the number of players, in order (two each in the four-player game). `p=` overrides.
4. **Percentage**: `Points × 25`. Team / player percentage = `Σpoints ÷ (4 × rated shots)`. `-` and `X` are excluded from the denominator.
5. **Final score**: sum of `Score` lines. If it conflicts with `Result`, `Result` wins with a warning.

### 6.1 Default rules by era and format

Without `FGZ` / `NoTick` tags, the rules of the time are inferred from `Date` and `Format`, using **World Curling (international) adoption dates**. National and club events that differed should tag explicitly.

**Four-player and wheelchair** (`Format` `Team` / `Wheelchair`)

| Period | `FGZ` | `NoTick` | Basis |
|---|---|---|---|
| to June 1993 | 0 | false | Before FGZ (prototype: 1990-91 Moncton Rule, trialled at the 1992 Olympics) |
| from July 1993 | 4 | false | WCF introduced the four-rock FGZ in international play from 1993-94 |
| from July 2018 | 5 | false | Decided at the September 2017 WCF Congress, five-rock from 2018-19 |
| from July 2022 | 5 | true | No-tick applied from 2022-23 (not wheelchair) |

Canadian national championships alone used **three rocks** from 1993-94 to 2001-02 (unified to four in 2002-03). Since that differs from the international basis it is not a date default; write `FGZ "3"` explicitly.

**Mixed doubles** (`Format` `MixedDoubles`) uses the following regardless of era.

| `FGZ` | `NoTick` | Basis |
|---|---|---|
| 3 | false | R17(e) (protection for the first three stones) / R7 (no-tick does not apply) |

#### What is protected differs by format (important)

The same `FGZ` tag protects different things in the four-player game and in mixed doubles.

- **Four-player and wheelchair (R6)**: before the 6th stone of the end, putting an **opponent's** stone that is in the FGZ (between the tee line and the hog line of the playing end, house excluded) **out of play** is a violation. Own stones may be removed, and stones in the house are not covered. The ruling is **mandatory**: remove the shooter, return the moved stones.
- **Mixed doubles (R17(e))**: before the 4th stone, putting **any already thrown or pre-positioned stone, own or opponent**, **out of play** is a violation. Location is not limited to the FGZ. Same ruling as the four-player game.
- **No-tick (R7)**: before the 6th stone, moving an **opponent's** FGZ stone that touches the centre line off the line or out of the FGZ is a violation. The ruling is **the non-offending team's choice** (① remove the shooter and restore, ② leave). A stone put fully out of play falls under R6(b). **Not applied to mixed doubles or wheelchair.**

PCN records the throw as it was; the player judges the outcome. A player may implement ① (restore) on a violation. When replaying a recorded game, use the recorded `@` boards as they are, without applying the judgement (the ruling of the day is already reflected; re-judging with estimated physics causes false positives). Violations preserved in a record can be marked with `x=fgz` / `x=notick` (§4.6).

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

## 11. Sources

Primary material this specification follows. The specification is unofficial and not affiliated with World Curling or CURLIT; where interpretations differ, the originals prevail.

| Content | Source |
|---|---|
| Rules of play (score of a conceded end = R12(h), FGZ = R6, no-tick = R7, mixed doubles = R17, wheelchair = R14) | *The Rules of Curling and Rules of Competition*, World Curling, July 2024 ([list](https://worldcurling.org/rules/) / [PDF](https://worldcurling.org/wp-content/uploads/2024/08/Rules-2024.pdf)) |
| Definitions of Task, Handle and Points | *Curling Statistics: How to Score*, © World Curling and CURLIT Ltd., 2025 ([PDF](https://curlit.com/powerpoint/StatsTraining_2025_2026.pdf)) |
| Source data for layers ② and ③ (Shot by Shot, diagrams) | World Curling Live Scores (`livescores.worldcurling.org`) / CURLIT Results Books (`curlit.com`) |
| History of the FGZ and national adoption dates | Curling Canada "History of Curling"; World Curling announcements (five-rock decision at the September 2017 Congress, 2022 no-tick trial) |

Rule excerpts (July 2024 edition, paraphrased):

- **R6(b)** — before the 6th stone of an end, if a delivered stone directly or indirectly moves an opponent's FGZ stone out of play, the delivered stone is removed and the moved stones are returned.
- **R7** — before the 6th stone, if an opponent's FGZ stone touching the centre line is moved off the line or out of the FGZ, the non-offending team chooses between "remove and restore" and "leave". **Not applied to wheelchair or mixed doubles.**
- **R17(e)** — before the 4th stone, if a delivered stone directly or indirectly moves **any delivered or pre-positioned stone** out of play, the delivered stone is removed and the moved stones are returned.
- **R17(c)(d)** — eight ends per game, five stones per team per end. The player who throws the first stone also throws the last; the other throws stones 2–4.

## 12. Change log

The `[PCN "1.0"]` value in files names the format family and is not revised while the document is a draft. The document is managed by draft revision number; format 1.0 is frozen when the content is settled (incompatible changes afterwards become 2.0).

| Document | Date |
|---|---|
| Draft, 5th revision (English edition added) | 2026-09-05 |
| Draft, 4th revision | 2026-09-01 |
| Draft, 3rd revision | 2026-08-25 |
| Draft, 2nd revision | 2026-08-23 |
| Draft, 1st revision | 2026-08 |
