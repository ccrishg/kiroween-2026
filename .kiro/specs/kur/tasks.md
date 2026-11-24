# Implementation Plan: KUR

> ## ⚠️ Scope recut on Tuesday 24 November 2026
>
> The previous version of this plan spread 82 leaf tasks between 3 and 27 November. That schedule was written against a wrong date: almost all of it had already elapsed, the "recordable demo" milestone of 22 November had passed, and nothing was implemented. This version is rebuilt against the time that actually remains: **8 development days, 24 November to 1 December, two developers, roughly 16 person-days.** Feature freeze is **Tuesday 1 December, end of day**. 2 December is video and write-up, 3 December is submission, 4 December is buffer, and the hackathon deadline is 5 December 07:00 KST.
>
> **Cut to fit, from 82 leaf tasks to 30:**
> - **Property-based tests: 15 properties down to 5.** Kept: Property 1 (exact dice probability fold), Property 3 (one piece per square), Property 4 (seven pieces conserved), Property 8 (every match terminates), Property 11 (frequencies inside the audible band). Properties 2, 5, 6, 7, 9, 10, 12, 13, 14 and 15 are out. The five kept are the cheapest to write and the clearest to show on camera.
> - **Opponent_Policy:** the simple deterministic scoring version only. No difficulty setting, no search.
> - **Audio_Engine:** interval tuning plus reactions to `roll`, `capture`, `rosette` and `match_end`. The `move` and `retire` reactions and the register shift under threat (Requirements 11.5, 11.7, 11.8) are out.
> - **Out entirely:** the hardening group (import-boundary ESLint rule, bundle budget gate), accessibility beyond the mute control and reduced motion (Requirements 16.4 and 16.6), Prediction_Mode (Requirement 13), Match_Record and local storage (Requirement 20), and the Match_Record round-trip property.
> - **Kept against the cut line, deliberately:** the Provenance_Panel as static text, because it costs an hour and carries the Resurrection-category argument; the Match_Summary in its minimal form, because the data already arrives on the `match_end` event and the educational claim collapses without it; and the audio unlock after a user gesture (Requirement 19.2), because without it the demo is silent in a judge's browser.
>
> Nothing in `requirements.md` or `design.md` changed. Those documents still describe the full system; this plan describes the part of it that will exist on 1 December.

## Overview

This plan builds a playable vertical slice of the design: Requirements 1–8 (Game_Engine, Dice_Model, legal moves, rosettes, capture, retirement, The_Dead, Event_Stream), 9 in minimal form (the bundled Hymn_Notation document and its parser), 10 (Tuning_System and run-time synthesis), 11 in reduced form (audio reacting to `roll`, `capture`, `rosette` and `match_end`), 12 (Probability_Visualizer), 14 in minimal form (Match_Summary), 15 (Board_Renderer and art direction), 16.1–16.3 and 16.7–16.9 (mute, reduced motion, non-colour cues), 17 (public URL), 18 (Provenance_Panel) and 19.1–19.7 (degraded audio and surfaced rejections).

Two developers work in two parallel lanes that meet at three checkpoints. Every task carries its owner:

| Tag | Owner | Surface |
|---|---|---|
| `[M1]` | Member 1 | `src/engine/` — Game_Engine, Dice_Model, rules, Opponent_Policy, Match_Controller turn loop, property harness, Properties 1, 3, 4 and 8; plus the two read-only `match_end` views on 30 November |
| `[M2]` | Member 2 | `src/audio/`, `src/render/`, `src/app/` — Audio_Engine, Tuning_System, Notation_Parser, Board_Renderer, Probability_Visualizer, interface and art direction, Property 11 |
| `[ambos]` | Both | The event contract, the engine/presentation wiring, the checkpoints |

The ordering serves one hard date: **Sunday 29 November, a match must be playable end to end in the browser.** The video and the write-up are 40% of the score and are never cut, so the plan reaches a complete-but-ugly slice first and spends only Monday 30 November and Tuesday 1 December on the teaching layer, the art pass and the properties.

Task 1.2 is a blocking gate. Nothing else starts until the Event_Stream contract is agreed and merged, because it is what lets M2 build the whole presentation layer against `Fake_Event_Source` while M1 is still writing the engine.

## Tasks

- [ ] 1. Foundations and the shared contract — Tue 24 November
  - [ ] 1.1 `[ambos]` Scaffold the repository toolchain
    - Create `package.json`, `vite.config.ts`, `tsconfig.json` with `strict: true` and `noUncheckedIndexedAccess: true`, and `vitest.config.ts` with two projects: `node` for `src/engine` and `src/audio`, `jsdom` for `src/render` and `src/app`
    - Install Vitest, `fast-check` and `@fast-check/vitest`; set the default `numRuns` to 100
    - Create the directory tree of the design: `src/engine`, `src/audio`, `src/render`, `src/app`, `src/data`, `src/devtools`, `tests/engine`, `tests/audio`, `tests/render`
    - Toolchain configuration only, no product code
    - _Requirements: 17.1_

  - [ ] 1.2 `[ambos]` **BLOCKING GATE — write the Event_Stream contract**
    - Write `src/engine/events.ts` exactly as specified in the design: `PlayerId`, `PieceId`, `RollValue`, `PathPos`, `Rational`, `EventKind`, `EventBase` carrying `kind`, `seq` and `player`
    - Define the six event interfaces with their full payloads: `RollEvent` (`rollValue`, `probability`, `extraRoll`, `rollCount`, `legalMoves`), `MoveEvent` (`pieceId`, `from`, `to`, `traversed`, `exposedOnShared`), `CaptureEvent` (`pathPos`, `capturedPlayer`, `capturedPieceId`, `handCountAfter`), `RosetteEvent` (`pathPos`, `grantedTo`), `RetireEvent` (`pieceId`, `from`, `retiredCountAfter`), `MatchEndEvent` (`winner`, `reason`, `rollCount`, `rollTally`, `captureCount`)
    - Define `LegalMoveSummary`, the `GameEvent` union, `EventConsumer` and `Unsubscribe`, and the `deepFreeze` helper the engine applies to every event before delivery
    - Keep the full payload set even for the events the reduced Audio_Engine ignores: the renderer and the Match_Summary still need them, and reopening this file later breaks both lanes at once
    - **No task below may start until this file is merged and both developers have signed off on it**
    - _Requirements: 8.1, 8.2, 8.3, 8.7, 8.8, 8.9_

  - [ ] 1.3 `[M2]` Write the Fake_Event_Source
    - Write `src/devtools/fakeEventSource.ts`: a scripted generator that replays a hand-written `GameEvent[]` with correct `seq` numbering, and a random-walk generator that produces plausible event sequences of a configurable length
    - Expose the same `subscribe(consumer)` shape as the engine so that swapping sources is a one-line change
    - This is what unblocks every M2 task from day one and doubles as the fixture for the render and audio tests
    - _Requirements: 8.1, 8.2, 8.5_

  - [ ] 1.4 `[M2]` Configure the public deployment pipeline
    - Write `.github/workflows/deploy.yml`: build with the Vite production build and publish `dist` to GitHub Pages on every push to the default branch
    - Set the Vite `base` path for the Pages URL and commit a minimal `index.html` shell so the first deploy produces a reachable page
    - Target: a public URL serving the skeleton by **the end of Tuesday 24 November**. Deployment is a submission rule, not a feature, and a Pages base-path problem discovered on 1 December would be a new problem at the worst moment
    - _Requirements: 17.1, 17.2, 17.3, 17.5_

- [ ] 2. `[ambos]` Checkpoint — contract frozen and public URL live, end of Tue 24 November
  - The public URL serves the skeleton, `events.ts` is merged and signed off, and `Fake_Event_Source` emits events both lanes can consume
  - Ensure all tests pass, ask the user if questions arise

- [ ] 3. `[M1]` Engine lane — Wed 25 to Fri 27 November
  - [ ] 3.1 Write the exact arithmetic, the board model and the initial state
    - Write `src/engine/rational.ts`: construction with `gcd` reduction, addition, multiplication and equality over integer pairs, with no float path
    - Write `src/engine/types.ts`: `PieceLocation`, `MatchState`, `Move`, `RuleId`, `Rejection` and `Result<T, E>`
    - Write `src/engine/board.ts`: square ids 0–19 over the 3×8 grid, `SHARED_SQUARES`, `ROSETTE_SQUARES`, `PATHS` for both players, `PATH_LENGTH`, `PIECES_PER_PLAYER`, the derived `occupancy(state)` map recomputed rather than stored, and the initial-state factory with fourteen pieces in Hand, `toMove: 'human'`, `pendingRoll: null`, counters at zero and `phase: 'awaiting_roll'`
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7, 2.2, 2.3_

  - [ ] 3.2 Implement the Dice_Model
    - Write `src/engine/dice.ts`: `mulberry32` seeded by an FNV-1a hash of the seed string; one roll draws four independent values and counts those below 0.5, simulating four two-outcome dice rather than sampling a precomputed distribution
    - `probabilityOf`, `probabilities` and `totalProbability` returning `Rational` values 1/16, 4/16, 6/16, 4/16 and 1/16, folded with exact rational addition to 1/1 with no decimal path
    - With no seed supplied, generate one from `crypto.getRandomValues` with a `Math.random` fallback and expose it on `seed`; reject a non-empty-string seed with a `Result` error carrying rule `'bad_seed'` and produce no `DiceModel`
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.7, 2.8_

  - [ ] 3.3 Implement the rules: legal moves, application and the single rejection point
    - Write `legalMoves(state, rollValue)` in `src/engine/rules.ts` returning, in fixed order: entry from Hand onto the path position equal to the Roll_Value, advance for each board piece where `p + rollValue <= 14`, and retirement where `p + rollValue === 15` exactly; the empty array for a Roll_Value of 0, and no move for `p + rollValue > 15`. Canonicalise entry from Hand to the lowest-id piece in Hand (Decision D3)
    - Exclude a destination holding a piece of the mover and a Rosette_Square holding an opponent piece; set `capture` on a move whose destination is a non-rosette Shared_Square holding an opponent piece, and `rosette` on a move ending on a Rosette_Square
    - Write `applyMove(state, move)` returning a new `MatchState`: advance by exactly the Roll_Value, consume `pendingRoll`, leave every other piece of the mover untouched; on a capture return the opposing piece to its Hand and discard its path progress, removing it from the square before placing the mover so no intermediate state holds two pieces on one square; on a retirement move the piece to `retired` and increment the retired count by exactly 1
    - Write `validateRequest(state, request)` returning `Result<Move>`, the only place the engine rejects: `not_your_turn`, `no_pending_roll`, `match_ended`, `piece_not_available`, `piece_retired`, `own_piece_on_destination`, `rosette_is_sheltered`, `beyond_path_end`, `move_not_legal`, each carrying a short human-readable `message` naming the violated rule. Never throw for a rule violation and never mutate state on a rejection
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.7, 3.8, 3.9, 3.10, 4.5, 5.1, 5.2, 5.3, 5.4, 5.5, 6.1, 6.2, 6.6, 6.7_

  - [ ] 3.4 Implement the Game_Engine: phases, emission, extra rolls and termination
    - Write `createEngine` in `src/engine/engine.ts` with phases `awaiting_roll`, `awaiting_move` and `ended`, plus `getState`, `getLegalMoves` and `reset`
    - `roll()` emits the `roll` event with its exact `probability`, `extraRoll`, `rollCount` and `legalMoves` summary, and resolves the degenerate turns itself: a Roll_Value of 0 or an empty legal set passes the turn and returns to `awaiting_roll` with no `move` event
    - `submitMove` applies the validated move, emits `move` then `capture`, `rosette` or `retire` as the move requires, and returns `MoveOutcome` with `extraRoll` and `matchEnded`. A move ending on a Rosette_Square keeps the mover to move and grants exactly one further roll regardless of how many rosettes that player occupies; any other ending, including a capture and a retirement, passes the turn. A retirement emits `retire` only and never a `move` event (Decision D4)
    - `subscribe` returning `Unsubscribe`; `seq` numbered consecutively from 1 with no gap and no repeat; every event deep-frozen before delivery and delivered synchronously in registration order over a snapshot of the consumer list, completing for all consumers before the next event is emitted; at least 8 consumers supported
    - Emit exactly one `match_end` when a retirement raises a player's retired count to 7, before accepting any further roll, and reject every later move request with `'match_ended'` and state unchanged; enforce the 2000-roll cap with `reason: 'roll_cap'`; carry `winner`, `rollCount`, `rollTally` and `captureCount` on the event
    - _Requirements: 3.1, 3.6, 4.1, 4.2, 4.3, 4.4, 5.6, 6.2, 6.3, 6.4, 6.5, 8.1, 8.2, 8.3, 8.6, 8.7, 8.9_

  - [ ] 3.5 Implement the simple Opponent_Policy
    - Write `src/engine/opponent.ts`: a single weighted scoring function over the at most seven legal moves, not a search and with no strength parameter. Weights in descending order: capture on a Shared_Square, landing on the central rosette at path position 8, landing on any other rosette, retiring a piece, entering from Hand, advancing the piece closest to retirement, and a penalty for ending on a non-rosette Shared_Square within four squares ahead of an enemy piece
    - Ties break on `(pieceId, to)`; return `null` exactly when the legal set is empty; select only from the set it was given; stay a pure function of `(state, rollValue, legal)`
    - The difficulty setting of Requirements 7.5 and the strength asymmetry are out of scope: one policy, deterministic, same every match
    - _Requirements: 7.1, 7.3, 7.6, 7.8_

- [ ] 4. `[M2]` Presentation lane — Wed 25 to Fri 27 November
  - [ ] 4.1 Declare the visual system and draw the board
    - Write `src/render/theme.ts`: at most 6 colours and at most 2 typefaces derived from clay, cuneiform forms and lamplight, exported as named constants; the layout rectangles for a 1280×720 viewport, board 640×240 as a 3×8 grid of 72 px squares with 8 px gutters, hand and retired trays flanking it, Probability_Visualizer in a 1280×240 band beneath
    - Write `src/render/boardRenderer.ts`: the 20 squares on that layout, the 5 rosettes with their marking and a fill at 3:1 contrast or better against the other squares, the hand and retired trays, and the pieces — human pieces as discs and The_Dead's as squares, so ownership survives greyscale
    - Implement `render(event)` as an exhaustive `switch` over the `GameEvent` union with an explicit default branch, deriving every drawn value from the event stream alone and reading no `MatchState`; show the resulting state within 700 ms, and on a `capture` show the piece leaving its square and the owning hand count rising within 600 ms; on an unrenderable event retain the last successful frame, show a staleness indicator and keep accepting events
    - If `canvas.getContext('2d')` returns null, show a message naming the missing capability and the supported desktop browsers instead of a blank surface
    - _Requirements: 4.6, 8.5, 15.1, 15.4, 15.5, 15.6, 15.7, 15.8, 15.9, 16.5, 17.9_

  - [ ] 4.2 Implement the Event_Queue, the motion paths and the destination affordance
    - Write `src/render/eventQueue.ts`: `push`, `pending` and `drained()`, awaiting each `render` before starting the next event so events reach the screen in stream order, one at a time
    - Interpolate the moved piece along `traversed` over 300 ms at `requestAnimationFrame` cadence, resolving `render` only when the event is on screen; `setReducedMotion(true)`, and an operating-system reduced-motion preference, place the piece in a single 16 ms frame instead, with no flashing and no looping motion. Add the session-scoped reduced-motion control that forces the single-frame path regardless of the operating-system preference
    - `setLegalDestinations(summaries)` drawing each legal destination with a lighter fill plus an inset ring and a shape cue, not a colour-only cue, at 3:1 contrast or better against a non-destination square, and no treatment at all when the human's legal set is empty; `onSquareClick(handler)` reporting a `PathPos` or `'hand'`
    - _Requirements: 15.2, 15.3, 15.7, 16.3, 16.8, 16.9_

  - [ ] 4.3 Write the hymn notation document, the minimal Notation_Parser and the Tuning_System
    - Write `src/data/hymn-h6.kur` in the line format of the design: a `#` comment header carrying the citation by author, title and year and the licensing position of the transcription, `vocab <name> <num>/<den>` lines declaring the complete accepted vocabulary, and `seq <name>...` lines concatenating into the Interval_Sequence. Reproduce nothing from the cited publication beyond the interval names and the citation itself. Configure the Vite `?raw` import
    - Write `parseHymnNotation` in `src/audio/notation.ts` returning `Result<IntervalSequence, NotationError>`: 1 to 500 intervals, members in 1 to 1024, the first offending symbol reported with its 1-based line and column, codes `unknown_symbol`, `bad_ratio`, `bound_violated` and `malformed_line`, document order preserved and no Interval_Sequence returned on any error. The Notation_Printer and its round-trip property are out of scope, so this parser reads the one bundled document and nothing else
    - If the bundled document fails to parse at startup, start the match with audio disabled and show a message naming the offending symbol with its line and column
    - Write `src/audio/tuning.ts`: `frequencyOf(degree)` computing `tonicHz * num / den` to within 0.01 Hz of the exact rational result, then folding into 110–1760 Hz by doubling or halving at most ten times; `setTonic` accepting 110 to 440 Hz only and otherwise retaining the previous tonic and returning an out-of-range error, default 220 Hz; a non-positive-integer ratio member yielding an error naming the degree and no frequency
    - _Requirements: 9.1, 9.2, 9.5, 9.6, 9.7, 9.8, 10.1, 10.4, 10.5, 10.7, 10.9, 19.4_

  - [ ] 4.4 Implement the reduced Audio_Engine
    - Write `src/audio/audioEngine.ts`: a fixed pool of eight voices, each `OscillatorNode` (triangle) → `GainNode` (10 ms attack, exponential decay) → shared `masterGain` → destination. Per-voice peak gain 0.1 and `masterGain` 0.9, so the summed peak cannot exceed 0.72 and Requirement 10.6 holds by arithmetic rather than by a limiter. A ninth request stops the longest-sounding voice first; schedule every tone at `context.currentTime + 0.01`; generate every tone through synthesis nodes created at run time and decode no audio file
    - Write `src/audio/degreeMap.ts`: five distinct `ROLL_DEGREES` indexed by Roll_Value, plus `CAPTURE_DEGREE` and `SHELTER_DEGREE` distinct from each other
    - Write `handle(event)` as the only Event_Stream entry point, reacting to exactly four event kinds: `roll`, `capture` and `rosette` with durations of 200–1500 ms, and `match_end` with the final four degrees of the Interval_Sequence in recorded order, all tones stopped within 2000 ms of it. Ignore `move` and `retire` entirely, and apply no register shift under threat — Requirements 11.5, 11.7 and 11.8 are out of scope
    - Add `setMuted` and the one interface control that drives it: operable by keyboard alone, indicating its state by text or shape and not by colour alone, retained for the rest of the browser session, stopping every sounding tone within 100 ms, and leaving `MatchState` and the Event_Stream identical to an unmuted run
    - _Requirements: 10.2, 10.3, 10.6, 10.8, 11.1, 11.2, 11.3, 11.4, 11.6, 11.9, 16.1, 16.2, 16.7_

- [ ] 5. `[ambos]` Match_Controller and the end-to-end wiring — Sat 28 to Sun 29 November
  - [ ] 5.1 `[M1]` Implement the Match_Controller turn loop
    - Write `src/app/MatchController.ts`, the only component that knows about time: it owns the engine, the policy and the presentation clock, and exposes the current phase, whether Roll is enabled, the pending Roll_Value, the human's legal destinations, the last rejection message and the audio status
    - On a human roll, enable destinations; on a destination pick, submit the move and on rejection keep `pendingRoll` so another legal move can follow
    - After any turn end, await `queue.drained()`; for The_Dead, roll, call `selectMove` behind the 500 ms fallback that takes `legal[0]`, submit, await `drained()`, then pause between 400 and 1200 ms with all input disabled, repeating while it keeps the turn
    - _Requirements: 7.2, 7.4, 7.7_

  - [ ] 5.2 `[M2]` Build the application shell
    - Write `src/app/App.tsx` and `src/app/components/`: the board canvas, the visualiser canvas, the Roll control, the readouts for the current Roll_Value and the player to move at 32 px or more, the mute and reduced-motion controls, and the rejection message surface
    - Boot straight into a set-up match where the Roll control is the only live affordance and no configuration input is required
    - Keep a dev-only toggle between `Fake_Event_Source` and the real engine
    - _Requirements: 1.9, 15.4, 15.6_

  - [ ] 5.3 `[ambos]` Wire the real engine to its consumers and surface rejections
    - Subscribe the Event_Queue and the Audio_Engine to the engine's `subscribe`, with the Audio_Engine attached directly rather than through the queue because it must react within 100 ms and does not animate; leave the visualiser subscription point ready for task 7.1
    - On a rejected human move request, leave the drawn board unchanged, retain the pending Roll_Value so another legal move can be submitted, and show the `Rejection.message` verbatim within 200 ms
    - Take the fake source out of the production path and verify a complete human-versus-The_Dead match runs from the first roll to `match_end` on the public URL
    - _Requirements: 8.5, 8.6, 11.1, 19.5_

- [ ] 6. `[ambos]` Checkpoint — playable end to end in the browser, Sun 29 November (HARD MILESTONE)
  - A full match can be played on the public URL against The_Dead with audio responding: ugly but complete, and recordable if everything after this slips
  - If this checkpoint is missed, cut again immediately. The video and the write-up are never cut
  - Ensure all tests pass, ask the user if questions arise

- [ ] 7. The teaching layer, the provenance and the art pass — Mon 30 November
  - [ ] 7.1 `[M2]` Implement the Probability_Visualizer
    - Write `src/render/probabilityVisualizer.ts`: five groups of two bars, expected against observed, on one shared axis from 0 to `max(6/16, maxObserved)` so lengths are directly comparable; expected bars outlined with a hatch pattern and observed bars solid so the distinction survives greyscale; labels at 18 px; the total roll count shown as an integer; the five expected probabilities displayed as the exact values 1/16, 4/16, 6/16, 4/16 and 1/16 at all times, and the empty-match state with a statement that no roll has been recorded
    - `handle(event)` increments the observed count of the Roll_Value and redraws within 16 ms as a single `fillRect` pass over ten bars and fifteen labels, displaying both the observed count and the observed relative frequency; `reset()` zeroes every count and the total before the first `roll` event of a new match; a `roll` event carrying a value outside 0 to 4 leaves every counter untouched, shows a "1 roll not counted" notice and keeps accepting events
    - Subscribe it directly to the engine at the point task 5.3 left ready
    - _Requirements: 12.1, 12.2, 12.3, 12.4, 12.5, 12.6, 12.7, 12.8, 12.9, 16.9_

  - [ ] 7.2 `[M1]` Implement the minimal Match_Summary
    - A read-only view over data that already exists: the `match_end` event's `rollTally`, `captureCount` and `winner`, and the visualiser's observed counts. For the five Roll_Values in ascending order, show the observed count, the observed relative frequency and the expected probability as percentages to one decimal place, and the signed difference in percentage points to one decimal place, within 1000 ms of `match_end`
    - Show the total roll count labelled as the sample size with the statement that one match is a single small sample in which deviation is normal, make no claim about statistical significance, and display 0 for a count whose events did not occur
    - The new-match control resets every observed count and the capture count to 0. The prediction-accuracy rows of Requirements 14.3 and 14.4 are out of scope because Prediction_Mode is cut
    - _Requirements: 14.1, 14.2, 14.5, 14.6, 14.7_

  - [ ] 7.3 `[M1]` Implement the Provenance_Panel as static text
    - A React component plus a `src/data/provenance.ts` constant: the Finkel_Ruleset citation, the hymn transcription citation matching the one in `hymn-h6.kur`, the statement that every tone is synthesised at run time, the five items each labelled "attested by the artefact" or "supplied by modern reconstruction", the redistribution and licence statements, a control reachable before, during and after a match, and the fallback that names an item whose source is not recorded while still showing the rest
    - Static text, no engine dependency, roughly one hour, and it is what makes the Resurrection framing an argument rather than a claim
    - _Requirements: 18.1, 18.2, 18.3, 18.4, 18.5, 18.6, 18.7, 18.8_

  - [ ] 7.4 `[M2]` Apply the art direction and the typography
    - Render the board, pieces and trays in the declared visual system of clay, cuneiform forms and lamplight, with no decorative imagery outside it; keep each square at 56×56 px or more and each piece at 32 px or more across on a 1280×720 viewport, and keep the piece shapes distinguishable in greyscale
    - At most two typefaces; all text at 16 px or more and 4.5:1 contrast or better; the current Roll_Value and the player to move at 32 px or more; the focus indicator at 3:1 contrast or better against its background
    - _Requirements: 12.9, 15.4, 15.5, 15.6, 16.5_

- [ ] 8. Properties, audio unlock and the freeze — Tue 1 December
  - [ ] 8.1 `[M1]` Build the property harness
    - Write `tests/engine/harness.ts`: `arbSeed` (non-empty printable string) and `selfPlay(seed)` playing a full match with both sides driven by the Opponent_Policy and returning the trace of every emitted event with the `MatchState` after each
    - Reachable states come from the trace rather than from a hand-rolled state generator, so every generated state is legally reachable
    - The harness runs in the Vitest `node` project with no `document` and no `window`, which is what makes the DOM-free engine claim of Requirement 8.4 an executed fact rather than an assertion
    - Each property test below lives in its own file under `tests/`, named after its property number, and carries the tag comment `// Feature: kur, Property N: <title>`
    - _Requirements: 7.1, 7.8, 8.4_

  - [ ]* 8.2 `[M1]` Write the property test for the probability fold
    - **Property 1: The dice probabilities sum to exactly one**
    - **Validates: Requirements 2.2, 2.3**
    - Generator: `fc.shuffledSubarray` over the five exposed `Rational` values taking the full array, plus `fc.array(arbRational)` to exercise the fold itself. `numRuns: 1000`; this one is cheap and it is the clearest property to show on camera

  - [ ]* 8.3 `[M1]` Write the property test for square occupancy
    - **Property 3: At most one piece stands on any square**
    - **Validates: Requirements 1.8, 5.4**
    - Generator: `arbSeed` through `selfPlay`. Map every board location of both players through `PATHS` and assert no repeated square id at every state in the trace. `numRuns: 1000`

  - [ ]* 8.4 `[M1]` Write the property test for piece conservation
    - **Property 4: Each player always owns exactly seven pieces**
    - **Validates: Requirements 1.7, 5.2, 5.4, 6.5**
    - Generator: the same `selfPlay` trace. Assert `hand + board + retired === 7` per player at every state, and that the retired count never decreases between consecutive states. `numRuns: 1000`

  - [ ]* 8.5 `[M1]` Write the property test for termination
    - **Property 8: Every valid match ends, within 2000 rolls**
    - **Validates: Requirements 6.3, 6.4**
    - Generator: `arbSeed` through `selfPlay`. Assert exactly one `match_end` in the trace, `rollCount <= 2000`, the named winner holding 7 retired pieces when `reason === 'all_retired'` and at least as many as the loser when `reason === 'roll_cap'`, and that requests after the end are rejected with `'match_ended'`

  - [ ]* 8.6 `[M2]` Write the property test for the audible band
    - **Property 11: Every synthesised frequency lands inside the audible band**
    - **Validates: Requirements 10.1, 10.4, 10.5**
    - Generator: `fc.double({ min: 110, max: 440 })` × `fc.record({ num: fc.integer({min:1,max:1024}), den: fc.integer({min:1,max:1024}) })`. Assert `110 <= f && f <= 1760` and that `Math.log2(f / exact)` is within 1e-9 of an integer of absolute value at most 10

  - [ ] 8.7 `[M2]` Implement the audio unlock gate and the degraded audio paths
    - If `AudioContext` is undefined at load, start the match with `status: 'disabled'` and show a notice within 1000 ms of the interface becoming playable
    - If the context is `suspended` after construction, render an `AudioGate` control that stays visible until activated or the match ends, calling `resume()` from inside the gesture handler and discarding events received while suspended rather than queueing them. **Without this the demo is silent in a judge's browser, so it is not optional and not deferred**
    - While audio is disabled, keep accepting rolls and move requests and keep updating the Board_Renderer and the Probability_Visualizer on every event
    - Wrap voice scheduling in try/catch with a per-match error counter: stop the affected tone within 100 ms, record the error and keep accepting events; three consecutive errors in one match disable the Audio_Engine for the rest of it with a notice
    - _Requirements: 19.1, 19.2, 19.3, 19.6, 19.7, 19.8_

- [ ] 9. `[ambos]` Checkpoint — feature freeze, end of Tue 1 December
  - From here only bug fixes. What is not in, does not go in. 2 December is the video and the write-up, 3 December is the submission, 4 December is buffer
  - Ensure all tests pass, ask the user if questions arise

## Notes

- **The recut is the headline, and it belongs in the write-up.** This plan was rebuilt on 24 November after the previous one was found to be scheduled against a date that had already passed. The interesting part is not that the schedule was wrong but what the constraint forced: five properties instead of fifteen, one opponent policy instead of three difficulty levels, an Audio_Engine that answers four events instead of six, and a Provenance_Panel kept over a Prediction_Mode because an hour of static text carries the category argument and a day of interaction design does not.
- **Owners.** `[M1]` is the engine lane plus Properties 1, 3, 4 and 8, and on 30 November the two read-only `match_end` views, because the engine lane is free that day and both views are pure presentation over data the engine already emits. `[M2]` is the audio, render and interface lane plus Property 11. `[ambos]` marks only what genuinely needs agreement: the event contract, the engine wiring and the checkpoints.
- **Task 1.2 is a blocking gate.** `src/engine/events.ts` is written and signed off before any other implementation task begins. From that moment M2 works against `src/devtools/fakeEventSource.ts` and never reads `MatchState`, so the two lanes do not wait on each other until task 5.
- **Deployment is on day one.** Task 1.4 puts the project on a public URL on 24 November. It is a submission requirement and it is the one thing that cannot be fixed at the last minute.
- **The 29 November milestone drives the order.** A playable slice (task 6) comes before the perfection of any layer. The teaching layer, the art pass and the properties all land in the two days after it, in that order, so that each of them can be dropped individually without losing the demo.
- **Tasks marked `*` are optional and can be skipped for a faster MVP.** In this plan they are only the five property tests. They are a scored criterion of the hackathon, so skipping them costs points even though the application still runs: treat them as the last thing to cut, not the first.
- **One property, one test.** Each of Properties 1, 3, 4, 8 and 11 is implemented by exactly one property-based test with Vitest and `fast-check` through the `@fast-check/vitest` binding, in its own file, tagged `// Feature: kur, Property N: <title>`. Minimum `numRuns` is 100, raised to 1000 for Properties 1, 3 and 4, which are cheap. Property-based testing is not implemented from scratch. The numbering follows `design.md`, so the gaps at 2, 5, 6, 7, 9, 10 and 12 to 15 are the cut properties and not a renumbering error.
- **Degraded paths are folded into their owning tasks, not given their own.** The missing-Canvas message sits in task 4.1, the notation parse failure in task 4.3, and the audio error counter in task 8.7. Each is a few lines, and at 16 person-days a separate task for each would cost more in coordination than in code.
- **No standalone unit-test tasks.** The previous plan carried eleven of them. At this scale the five properties plus the assertions written inline while implementing each task are the whole automated suite. The engine property harness also covers the DOM-free claim of Requirement 8.4 for free.
- **Half a day to a day each.** Thirty leaf tasks across roughly sixteen person-days of two-developer capacity between 24 November and 1 December. That is tight but it is arithmetic rather than hope, which is what the previous version was not.
- **Not in this list, because they are not coding tasks:** the manual browser matrix on the current and previous stable Chrome, Firefox, Safari and Edge at 1280×720, the projector legibility check, all of which belong in `docs/submission-checklist.md` before the 1 December freeze; and the demo video, the Kiro write-up and the Devpost submission itself.
- **Out of scope in this plan, recorded here so the write-up can say so plainly:** Prediction_Mode (Requirement 13), the prediction rows of the Match_Summary (14.3, 14.4), the screen-reader live region and the full keyboard board path (16.4, 16.6), Match_Record and local storage (Requirement 20), the Notation_Printer and its round trip (9.3, 9.4), the `move` and `retire` audio reactions and the register shift under threat (11.5, 11.7, 11.8), the opponent difficulty setting (7.5), the import-boundary ESLint rule, the bundle budget gate and ten of the fifteen design properties.

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1"] },
    { "id": 1, "tasks": ["1.2"] },
    { "id": 2, "tasks": ["1.3", "1.4"] },
    { "id": 3, "tasks": ["2"] },
    { "id": 4, "tasks": ["3.1", "4.1"] },
    { "id": 5, "tasks": ["3.2", "4.2"] },
    { "id": 6, "tasks": ["3.3", "4.3"] },
    { "id": 7, "tasks": ["3.4", "4.4"] },
    { "id": 8, "tasks": ["3.5"] },
    { "id": 9, "tasks": ["5.1", "5.2"] },
    { "id": 10, "tasks": ["5.3"] },
    { "id": 11, "tasks": ["6"] },
    { "id": 12, "tasks": ["7.1", "7.2"] },
    { "id": 13, "tasks": ["7.3", "7.4"] },
    { "id": 14, "tasks": ["8.1", "8.7"] },
    { "id": 15, "tasks": ["8.2", "8.6"] },
    { "id": 16, "tasks": ["8.3"] },
    { "id": 17, "tasks": ["8.4"] },
    { "id": 18, "tasks": ["8.5"] },
    { "id": 19, "tasks": ["9"] }
  ]
}
```
