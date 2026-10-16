# Changelog

The `[PCN "1.0"]` value in files names the format family and does not change while the document is a draft. The document is versioned by revision.

| Document | Date | Notes |
|---|---|---|
| Draft, 7th revision | 2026-10-16 | Current. Rule variations: the `Rules` tag (`"key=value …"`; keys `blank`, `tie`, `scoring`, `win`, `skins`, `pin`, `fgzmove`, `fgzsides`, `fgzzone`, `notickends`, `timeout`, all defaulting to World Curling's rules; the record always wins and unknown keys are kept and ignored) and §6.3 on writing the Grand Slam of Curling, Rock League and skins games. Shootouts: `Termination "Shootout"` and the `Shootout` tag (`"R 23.5; Y 41.0"`). `x=time` for a stone not thrown after running out of time. `Format` adds `WheelchairMixedDoubles` and `Mixed`; `FGZ` is an integer 0–15; `ThinkingTime` may be per end (`"4:00/end"`). The hammer after a blank end in mixed doubles is written out (R17(g)III). The rules PDF link points to its new location. Existing files keep their meaning. Examples 08 and 09 |
| Draft, 6th revision | 2026-09-29 | (2026-10-08: rule citations updated to the 1 September 2026 edition of *The Rules of Curling*; the score of a conceded end is now R12(i). No change to the format.) `End` attributes `hh=` / `curl=` (ice from that end on) and `orderR=` / `orderY=` (delivery rotation from that end on, for substitutions and the mixed doubles order); `p=` can refer to alternates, numbered after `Players`; `curl` in the `Ice` tag is now defined as how far a tee draw aimed down the centre line ends up off the aim line (previously: sideways movement from the near hog line), and `turns=` (standard turns per delivery, default 5) is added |
| Draft, 5th revision | 2026-09-04 | Published in this repository (previously part of the CurlFlux repository) |
| Draft, 4th revision | 2026-09-01 | |
| Draft, 3rd revision | 2026-08-25 | |
| Draft, 2nd revision | 2026-08-23 | |
| Draft, 1st revision | 2026-08 | |

Format 1.0 will be fixed when the content settles. Until then, changes are recorded here and in §12 of the specification.
