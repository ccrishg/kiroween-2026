# Implementation Plan: KUR

## Overview

This plan builds the MVP declared in `design.md`: Requirements 1–8 (Game_Engine, Dice_Model, rules, rosettes, capture, retirement, The_Dead, Event_Stream), 9 in minimal form (Notation_Parser and Notation_Printer), 10 (Tuning_System and synthesis), 11 (audio response to the match), 12 (Probability_Visualizer), 15 (Board_Renderer and art direction), 17 (deployment to a public URL) and 19.1, 19.2, 19.6 (audio degradation). Requirements 13, 14, 16, 18 and 20 are deferred; they appear as optional sub-tasks in task 19, in the pick-up order the design recommends.

Two developers work in parallel on two lanes that meet at three checkpoints. Every task carries its owner:

| Tag | Owner | Surface |
|---|---|---|
| `[M1]` | Member 1 | `src/engine/` — Game_Engine, Dice_Model, rules, Opponent_Policy, Match_Controller turn loop, property harness, Properties 1–10 |
| `[M2]` | Member 2 | `src/audio/`, `src/render/`, `src/app/` — Audio_Engine, Tuning_System, Notation_Parser, Probability_Visualizer, Board_Renderer, interface and art direction, Properties 11–14 |
| `[ambos]` | Both | The event contract, the engine/presentation wiring, Property 15, the checkpoints |

The ordering serves one hard date: **Sunday 22 November, a recordable end-to-end demo must exist.** The video and the write-up are 40% of the score and are never cut, so the plan reaches a playable vertical slice on 15 November and spends the following week completing and polishing it, rather than perfecting any single layer first. Feature freeze is **Friday 27 November**; the internal delivery deadline is **Friday 4 December**.

Task 1.2 is a blocking gate. Nothing else starts until the Event_Stream contract is agreed and merged, because it is what lets M2 build the whole presentation layer against `Fake_Event_Source` while M1 is still writing the engine.

## Tasks

- [ ] 1. Foundations and the shared contract — Tue 3 to Sun 8 November
  - [ ] 1.1 `[ambos]` Scaffold the repository toolchain
    - Create `package.json`, `vite.config.ts`, `tsconfig.json` with `strict: true` and `noUncheckedIndexedAccess: true`, and `vitest.config.ts` with two projects: `node` for `src/engine` and `src/audio`, `jsdom` for `src/render` and `src/app`
    - Install Vitest, `fast-check` and `@fast-check/vitest`; set the default `numRuns` to 100
    - Create the empty directory tree of the design: `src/engine`, `src/audio`, `src/render`, `src/app`, `src/data`, `src/devtools`, `tests/engine`, `tests/audio`, `tests/render`, `tests/smoke`
    - Toolchain configuration only, no product code
    - _Requirements: 17.1_

  - [ ] 1.2 `[ambos]` **BLOCKING GATE — write the Event_Stream contract**
    - Write `src/engine/events.ts` exactly as specified in the design: `PlayerId`, `PieceId`, `RollValue`, `PathPos`, `Rational`, `EventKind`, `EventBase` carrying `kind`, `seq` and `player`
    - Define the six event interfaces with their full payloads: `RollEvent` (`rollValue`, `probability`, `extraRoll`, `rollCount`, `legalMoves`), `MoveEvent` (`pieceId`, `from`, `to`, `traversed`, `exposedOnShared`), `CaptureEvent` (`pathPos`, `capturedPlayer`, `capturedPieceId`, `handCountAfter`), `RosetteEvent` (`pathPos`, `grantedTo`), `RetireEvent` (`pieceId`, `from`, `retiredCountAfter`), `MatchEndEvent` (`winner`, `reason`, `rollCount`, `rollTally`, `captureCount`)
    - Define `LegalMoveSummary`, the `GameEvent` union, `EventConsumer` and `Unsubscribe`
    - Write the `deepFreeze` helper that the engine applies to every event before delivery
    - **No task below may start until this file is merged and both developers have signed off on it.** Changing it later breaks both lanes at once
    - _Requirements: 8.1, 8.2, 8.3, 8.7, 8.8, 8.9_

  - [ ] 1.3 `[M2]` Write the Fake_Event_Source
    - Write `src/devtools/fakeEventSource.ts`: a scripted generator that replays a hand-written `GameEvent[]` with correct `seq` numbering, and a random-walk generator that produces plausible event sequences of a configurable length
    - Expose the same `subscribe(consumer)` shape as the engine so that swapping sources is a one-line change
    - This is what unblocks every M2 task from day one and doubles as the fixture for the render and audio tests
    - _Requirements: 8.1, 8.2, 8.5_

  - [ ] 1.4 `[M2]` Configure the public deployment pipeline
    - Write `.github/workflows/deploy.yml`: build with the Vite production build and publish `dist` to GitHub Pages on every push to the default branch
    - Set the Vite `base` path for the Pages URL and commit a minimal `index.html` shell so the first deploy produces a reachable page
    - Target: a public URL serving the skeleton by **Sunday 8 November**. Deployment is a submission rule, not a feature, and it is not left for the last week
    - _Requirements: 17.1, 17.2, 17.3, 17.5_

  - [ ] 1.5 `[M1]` Write the exact rational arithmetic and the engine type surface
    - Write `src/engine/rational.ts`: construction with `gcd` reduction, addition, multiplication and equality over integer pairs, with no float path
    - Write `src/engine/types.ts`: `PieceLocation`, `MatchState`, `Move`, `RuleId`, `Rejection` and `Result<T, E>`
    - _Requirements: 2.2, 2.3, 1.7_

  - [ ] 1.6 `[M1]` Write the board model and the initial state
    - Write `src/engine/board.ts`: square ids 0–19 over the 3×8 grid, `SHARED_SQUARES`, `ROSETTE_SQUARES`, `PATHS` for both players, `PATH_LENGTH` and `PIECES_PER_PLAYER`
    - Write the derived `occupancy(state)` map, recomputed rather than stored
    - Write the initial-state factory: fourteen pieces in Hand, `toMove: 'human'`, `pendingRoll: null`, counters at zero, `phase: 'awaiting_roll'`, `outcome: undecided`
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6_

  - [ ]* 1.7 `[M1]` Write unit tests for the board constants and the initial state
    - Assert 20 squares, 8 Shared_Squares, 5 Rosette_Squares at path positions 4, 8 and 14, and that path positions 5–12 are the only overlap between the two paths
    - Assert the initial state field by field
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6_

  - [ ] 1.8 `[M2]` Declare the visual system and the layout
    - Write `src/render/theme.ts`: at most 6 colours and at most 2 typefaces derived from clay, cuneiform forms and lamplight, exported as named constants with their contrast ratios computable from the palette
    - Declare the layout rectangles for a 1280×720 viewport: board 640×240 as a 3×8 grid of 72 px squares with 8 px gutters, hand and retired trays flanking it, Probability_Visualizer in a 1280×240 band beneath
    - _Requirements: 15.4, 15.5, 15.6_

  - [ ]* 1.9 `[M2]` Write unit tests for the palette and the layout
    - Compute contrast ratios from the palette and assert rosette against non-rosette at 3:1 or better, text against background at 4.5:1 or better
    - Assert that the declared layout rectangles do not overlap and fit inside 1280×720 without scrolling
    - _Requirements: 4.6, 12.9, 15.2, 15.4, 15.5, 15.6_

- [ ] 2. `[ambos]` Checkpoint — contract frozen and public URL live, Sun 8 November
  - The public URL serves the skeleton, `events.ts` is merged and signed off, and `Fake_Event_Source` emits events both lanes can consume
  - Ensure all tests pass, ask the user if questions arise

- [ ] 3. `[M1]` Dice_Model and exact probability — Mon 9 to Tue 10 November
  - [ ] 3.1 Implement the seeded roll
    - Write `src/engine/dice.ts`: `mulberry32` PRNG seeded by an FNV-1a hash of the seed string
    - One roll draws four independent values and counts those below 0.5, simulating four two-outcome dice with p = 1/2 rather than sampling a precomputed distribution
    - _Requirements: 2.1, 2.4, 2.5_

  - [ ] 3.2 Implement the probability surface and the seed contract
    - `probabilityOf`, `probabilities` and `totalProbability` returning `Rational` values 1/16, 4/16, 6/16, 4/16 and 1/16, folded with exact rational addition to 1/1 with no decimal path
    - When no seed is supplied, generate one from `crypto.getRandomValues` with a `Math.random` fallback and expose it on `seed`
    - Reject a seed that is not a non-empty string with a `Result` error carrying rule `'bad_seed'` and produce no `DiceModel`
    - _Requirements: 2.2, 2.3, 2.7, 2.8_

  - [ ]* 3.3 Write the property test for the probability fold
    - **Property 1: The dice probabilities sum to exactly one**
    - **Validates: Requirements 2.2, 2.3**
    - Generator: `fc.shuffledSubarray` over the five exposed `Rational` values taking the full array, plus `fc.array(arbRational)` to exercise the fold itself

  - [ ]* 3.4 Write the property test for seeded reproducibility
    - **Property 2: Seeded rolls are reproducible and always in range**
    - **Validates: Requirements 2.1, 2.4, 2.5, 2.7**
    - Generator: `arbSeed`. Assert prefix equality of 10000 values across two independently constructed models and range membership of every value

  - [ ]* 3.5 Write the unit test for the distribution tolerance
    - Three fixed seeds, 100000 draws each, observed relative frequencies within 0.01 of the expected values
    - A statistical check that would be absurd to run 100 times, so it is an example and not a property
    - _Requirements: 2.6_

- [ ] 4. `[M1]` Rules: legal moves and move application — Tue 10 to Thu 12 November
  - [ ] 4.1 Implement legal move generation
    - Write `legalMoves(state, rollValue)` in `src/engine/rules.ts` returning, in fixed order: entry from Hand onto the path position equal to the Roll_Value, advance for each board piece where `p + rollValue <= 14`, and retirement where `p + rollValue === 15` exactly
    - Return the empty array for a Roll_Value of 0, and no move at all for `p + rollValue > 15`
    - Canonicalise entry from Hand to the lowest-id piece in Hand (Decision D3), so the set holds each game move exactly once
    - _Requirements: 3.1, 3.2, 3.3, 6.1_

  - [ ] 4.2 Implement destination admission and the capture flag
    - Exclude a destination holding a piece of the mover, and a destination that is a Rosette_Square holding a piece of the opponent
    - Set `capture` on a move whose destination is a non-rosette Shared_Square holding an opponent piece, and mark `rosette` on a move ending on a Rosette_Square
    - _Requirements: 3.4, 3.5, 4.5, 5.1, 5.3, 5.5_

  - [ ] 4.3 Implement the pure state transition
    - Write `applyMove(state, move)` returning a new `MatchState`: advance the piece by exactly the Roll_Value, consume `pendingRoll`, leave every other piece of the mover untouched
    - On a capture, return the opposing piece to its Hand and discard its path progress, removing it from the square before placing the mover so that no intermediate state holds two pieces on one square
    - On a retirement, move the piece to `retired` and increment the retired count by exactly 1
    - _Requirements: 3.10, 5.1, 5.2, 5.4, 6.2_

  - [ ] 4.4 Implement the single rejection point
    - Write `validateRequest(state, request)` returning `Result<Move>`, the only place the engine rejects: `not_your_turn`, `no_pending_roll`, `match_ended`, `piece_not_available`, `piece_retired`, `own_piece_on_destination`, `rosette_is_sheltered`, `beyond_path_end`, `move_not_legal`
    - Each rejection carries a short human-readable `message` naming the violated rule, which the interface shows verbatim
    - Never throw for a rule violation and never mutate state on a rejection
    - _Requirements: 3.4, 3.5, 3.7, 3.8, 3.9, 6.6, 6.7_

- [ ] 5. `[M1]` Game_Engine: phases, emission and termination — Thu 12 to Sat 14 November
  - [ ] 5.1 Implement the phase machine and `roll()`
    - Write `createEngine` in `src/engine/engine.ts` with phases `awaiting_roll`, `awaiting_move` and `ended`, plus `getState`, `getLegalMoves` and `reset`
    - `roll()` emits the `roll` event with its exact `probability`, `extraRoll`, `rollCount` and `legalMoves` summary, and resolves the degenerate turns itself: a Roll_Value of 0 or an empty legal set passes the turn and returns to `awaiting_roll` with no `move` event
    - _Requirements: 3.1, 3.6, 8.7_

  - [ ] 5.2 Implement `submitMove`, the extra roll and turn passing
    - Apply the validated move, emit `move` then `capture`, `rosette` or `retire` as the move requires, and return `MoveOutcome` with `extraRoll` and `matchEnded`
    - A move ending on a Rosette_Square keeps the mover to move and grants exactly one further roll regardless of how many rosettes that player occupies; any other ending, including a capture and a retirement, passes the turn
    - A retirement emits `retire` only and never a `move` event (Decision D4)
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 5.6, 6.2_

  - [ ] 5.3 Implement event emission and delivery
    - `subscribe` returning `Unsubscribe`; numbering `seq` consecutively from 1 with no gap and no repeat
    - Deep-freeze every event before delivery; deliver synchronously in registration order over a snapshot of the consumer list taken at emission, completing for all consumers before the next event is emitted and before the next move is applied; support at least 8 consumers
    - _Requirements: 8.1, 8.2, 8.3, 8.6_

  - [ ] 5.4 Implement termination and the match_end payload
    - Emit exactly one `match_end` when a retirement raises a player's retired count to 7, before accepting any further roll, and reject every later move request with `'match_ended'` and state unchanged
    - Enforce the 2000-roll cap, ending the match with the winner on retired count and `reason: 'roll_cap'`
    - Carry `winner`, `rollCount`, `rollTally` and `captureCount` on the event
    - _Requirements: 6.3, 6.4, 6.5, 8.9_

- [ ] 6. `[M1]` Opponent_Policy — Sat 14 November
  - [ ] 6.1 Implement the deterministic scoring policy
    - Write `src/engine/opponent.ts`: a weighted scoring function over the at most seven legal moves, not a search. Weights in descending order: capture on a Shared_Square, landing on the central rosette at path position 8, landing on any other rosette, retiring a piece, entering from Hand, advancing the piece closest to retirement, and a penalty for ending on a non-rosette Shared_Square within four squares ahead of an enemy piece
    - `'dead'` takes the highest score, `'restless'` the lowest; ties break on `(pieceId, to)`
    - Return `null` exactly when the legal set is empty; select only from the set it was given; stay a pure function of `(state, rollValue, legal, strength)`
    - _Requirements: 7.1, 7.3, 7.6, 7.8_

- [ ] 7. `[M2]` Hymn notation and Tuning_System — Mon 9 to Wed 11 November
  - [ ] 7.1 Write the bundled Hymn_Notation document
    - Write `src/data/hymn-h6.kur` in the line format of the design: `#` comment header carrying the citation by author, title and year and the licensing position of the transcription, `vocab <name> <num>/<den>` lines declaring the complete accepted vocabulary, `seq <name>...` lines concatenating into the Interval_Sequence
    - Reproduce nothing from the cited publication beyond the interval names and the citation itself
    - Configure the Vite `?raw` import of the file
    - _Requirements: 9.5, 9.6, 9.8_

  - [ ] 7.2 Implement the Notation_Parser
    - Write `parseHymnNotation` in `src/audio/notation.ts` returning `Result<IntervalSequence, NotationError>`
    - Enforce 1 to 500 intervals and members in 1 to 1024, and report the first offending symbol with its 1-based line and column, with codes `unknown_symbol`, `bad_ratio`, `bound_violated` and `malformed_line`
    - Preserve document order and return no Interval_Sequence on any error
    - _Requirements: 9.1, 9.2, 9.7_

  - [ ] 7.3 Implement the Notation_Printer
    - Write `printHymnNotation`: assign deterministic synthetic names `i1, i2, …` to distinct ratios in order of first occurrence, emit one `vocab` line per distinct ratio and `seq` lines of eight names each
    - The output is a function of the input alone, so printing the same sequence twice is byte-identical
    - _Requirements: 9.3_

  - [ ]* 7.4 Write the property test for the notation round trip
    - **Property 12: Printing then parsing the hymn notation returns the same intervals**
    - **Validates: Requirements 9.1, 9.3, 9.4**
    - Generator: `fc.array(arbIntervalRatio, { minLength: 1, maxLength: 500 })` with both members drawn from 1 to 1024. Assert `parse(print(s))` is `ok` and deep-equal to `s`, and that `print(s) === print(s)`. Raise `numRuns` to 1000; this property is cheap and parsers are where round trips earn their keep

  - [ ] 7.5 Implement the Tuning_System
    - Write `src/audio/tuning.ts`: `frequencyOf(degree)` computing `tonicHz * num / den` to within 0.01 Hz of the exact rational result, then folding into 110–1760 Hz by doubling or halving at most ten times
    - `setTonic` accepts 110 to 440 Hz only, otherwise retains the previously accepted tonic and returns an out-of-range error; default tonic 220 Hz
    - A ratio member that is not a positive integer yields an error naming the degree and no frequency
    - _Requirements: 10.1, 10.4, 10.5, 10.7, 10.9_

  - [ ]* 7.6 Write the property test for the audible band
    - **Property 11: Every synthesised frequency lands inside the audible band**
    - **Validates: Requirements 10.1, 10.4, 10.5, 11.5**
    - Generator: `fc.double({ min: 110, max: 440 })` × `fc.record({ num: fc.integer({min:1,max:1024}), den: fc.integer({min:1,max:1024}) })` × `fc.integer({ min: 0, max: 1 })` for the register shift. Assert `110 <= f && f <= 1760` and that `Math.log2(f / exact)` is within 1e-9 of an integer of absolute value at most 10

  - [ ]* 7.7 Write unit tests for the notation and tonic boundaries
    - Bound violations and unknown symbols reported with the right line and column, and the three error codes distinguished
    - Tonic outside 110–440 Hz retains the previous value and returns an error
    - _Requirements: 9.2, 9.7, 10.9_

- [ ] 8. `[M2]` Audio_Engine — Wed 11 to Thu 12 November
  - [ ] 8.1 Implement the voice pool and the gain budget
    - Write `src/audio/audioEngine.ts`: a fixed pool of eight voices, each `OscillatorNode` (triangle) → `GainNode` (10 ms attack, exponential decay) → shared `masterGain` → destination
    - Per-voice peak gain 0.1 and `masterGain` 0.9, so the summed peak cannot exceed 0.72 and Requirement 10.6 holds by arithmetic rather than by a limiter
    - A ninth request stops the longest-sounding voice first; schedule every tone at `context.currentTime + 0.01`
    - Generate every tone through synthesis nodes created at run time and decode no audio file
    - _Requirements: 10.2, 10.3, 10.6, 10.8, 11.6, 11.9_

  - [ ] 8.2 Implement the degree map and the event response
    - Write `src/audio/degreeMap.ts`: five distinct `ROLL_DEGREES` indexed by Roll_Value, a `move` degree derived from the destination modulo the sequence length, and `CAPTURE_DEGREE`, `SHELTER_DEGREE` and `RETIRE_DEGREE` all distinct from one another
    - Write `handle(event)` as the only Event_Stream entry point, with durations 200–1500 ms for `roll`, `capture`, `rosette` and `retire`, 100–800 ms for `move`, and the final four degrees in recorded order on `match_end`, all tones stopped within 2000 ms of it
    - While the Human_Player has been captured twice or more in the current match, raise every degree index by one before the frequency lookup; reset the counter on `match_end`
    - Add `setMuted`, which stops every sounding tone within 100 ms
    - _Requirements: 11.1, 11.2, 11.3, 11.4, 11.5, 11.7, 11.8_

  - [ ]* 8.3 Write the property test for polyphony and degree stability
    - **Property 13: Polyphony never exceeds eight voices and the same cause sounds the same**
    - **Validates: Requirements 11.1, 11.6, 11.7, 11.9**
    - Generator: `fc.array(arbGameEvent, { minLength: 1, maxLength: 400 })` from `devtools/fakeEventSource`, played through a stub `AudioContext` with a controllable clock recording every `start`, `stop` and frequency. Assert the live-voice count never exceeds 8, that the voice stolen on overflow is the oldest live one, that every onset is within 0.1 s of its event, that every duration sits inside its kind's bounds, and that the degree for a given Roll_Value is constant modulo the register shift

  - [ ]* 8.4 Write unit tests for the gain budget and the closing phrase
    - Assert the gain arithmetic bounds the summed peak at or below 1.0
    - Assert the `match_end` response is the final four degrees of the Interval_Sequence in recorded order
    - _Requirements: 10.6, 11.4_

- [ ] 9. `[M2]` Board_Renderer and Event_Queue — Fri 13 to Sat 14 November
  - [ ] 9.1 Draw the static board
    - Write `src/render/boardRenderer.ts`: the 20 squares on the declared layout, the 5 rosettes with their marking and distinct fill, the hand and retired trays, and the pieces
    - Human pieces as discs and The_Dead's as squares, so ownership survives greyscale
    - If `canvas.getContext('2d')` returns null, report it to the caller rather than drawing a blank surface
    - _Requirements: 4.6, 15.1, 15.4, 16.5_

  - [ ] 9.2 Render the event stream
    - Implement `render(event)` as an exhaustive `switch` over the `GameEvent` union with an explicit default branch, deriving every drawn value from the event stream alone and reading no `MatchState`
    - Show the state resulting from an event within 700 ms of receiving it; on a `capture` show the piece leaving its square and the owning player's hand count rising within 600 ms
    - On an event it cannot render, retain the last successful frame, show a staleness indicator and keep accepting events
    - _Requirements: 8.5, 15.1, 15.7, 15.8, 15.9_

  - [ ] 9.3 Implement the move animation and the reduced-motion path
    - Interpolate the moved piece along `traversed` over 300 ms at `requestAnimationFrame` cadence, resolving `render` only when the event is on screen
    - `setReducedMotion(true)`, and an operating-system reduced-motion preference, place the piece in a single 16 ms frame instead, with no flashing and no looping motion
    - _Requirements: 15.3, 16.3, 16.8_

  - [ ] 9.4 Implement the Event_Queue
    - Write `src/render/eventQueue.ts`: `push`, `pending` and `drained()`, awaiting each `render` before starting the next event so that events reach the screen in stream order, one at a time
    - _Requirements: 15.7_

  - [ ] 9.5 Implement the legal destination affordance
    - `setLegalDestinations(summaries)` drawing each legal destination with a lighter fill plus an inset ring, a shape cue and not a colour-only cue, at 3:1 contrast or better against a non-destination square, and drawing no treatment at all when the human's legal set is empty
    - `onSquareClick(handler)` reporting a `PathPos` or `'hand'`
    - _Requirements: 15.2, 16.9_

  - [ ]* 9.6 Write unit tests for the queue, the motion branches and the fallback
    - Event_Queue ordering with a stub renderer and fake timers
    - The animation and reduced-motion branches with fake timers
    - The unrenderable-event branch retaining the last frame, and the missing Canvas 2D branch
    - _Requirements: 15.3, 15.7, 15.8, 15.9, 17.9_

- [ ] 10. `[M2]` Probability_Visualizer — Sat 14 November
  - [ ] 10.1 Draw the distribution
    - Write `src/render/probabilityVisualizer.ts`: five groups of two bars, expected against observed, on one shared axis from 0 to `max(6/16, maxObserved)` so lengths are directly comparable
    - Expected bars outlined with a hatch pattern and observed bars solid, so the distinction survives greyscale; labels at 18 px; the total roll count shown as an integer
    - Display the five expected probabilities as the exact values 1/16, 4/16, 6/16, 4/16 and 1/16 at all times, and the empty-match state with a statement that no roll has been recorded
    - _Requirements: 12.1, 12.3, 12.4, 12.5, 12.9, 16.9_

  - [ ] 10.2 Implement the tally and the redraw
    - `handle(event)` increments the observed count of the Roll_Value and redraws within 16 ms as a single `fillRect` pass over ten bars and fifteen labels, displaying both the observed count and the observed relative frequency
    - `reset()` zeroes every count and the total before the first `roll` event of a new match
    - A `roll` event carrying a value outside 0 to 4 leaves every counter and the total untouched, shows a "1 roll not counted" notice and keeps accepting events
    - _Requirements: 12.2, 12.6, 12.7, 12.8_

  - [ ]* 10.3 Write the property test for the observed tally
    - **Property 14: The visualiser's observed tally matches the stream it was given**
    - **Validates: Requirements 12.2, 12.3, 12.8**
    - Generator: `fc.array(arbGameEvent)` with a configurable proportion of deliberately out-of-range `roll` events. Assert each count against an independent count of the generated sequence, the total against the sum of the five, the relative frequencies summing to 1 whenever the total exceeds 0, the axis maximum covering both the largest expected and the largest observed value, and that an out-of-range value changes nothing

- [ ] 11. `[ambos]` Match_Controller and the first end-to-end wiring — Sat 14 to Sun 15 November
  - [ ] 11.1 `[M1]` Implement the Match_Controller turn loop
    - Write `src/app/MatchController.ts`, the only component that knows about time: it owns the engine, the policy and the presentation clock, and exposes the current phase, whether Roll is enabled, the pending Roll_Value, the human's legal destinations, the last rejection message and the audio status
    - On a human roll, enable destinations; on a destination pick, submit the move and on rejection keep `pendingRoll` so another legal move can follow
    - After any turn end, await `queue.drained()`; for The_Dead, roll, call `selectMove` behind the 500 ms fallback that takes `legal[0]`, submit, await `drained()`, then pause between 400 and 1200 ms with all input disabled, repeating while it keeps the turn
    - Apply the opponent strength selected at the start of a match for the rest of that match
    - _Requirements: 7.2, 7.4, 7.5, 7.7_

  - [ ] 11.2 `[M2]` Build the application shell
    - Write `src/app/App.tsx` and `src/app/components/`: the board canvas, the visualiser canvas, the Roll control, the readouts for the current Roll_Value and the player to move at 32 px or more, and the rejection message surface
    - Boot straight into a set-up match where the Roll control is the only live affordance and no configuration input is required
    - Keep a dev-only toggle between `Fake_Event_Source` and the real engine
    - _Requirements: 1.9, 15.4, 15.6_

  - [ ] 11.3 `[ambos]` Wire the real engine to the three consumers
    - Subscribe the Event_Queue, the Audio_Engine and the Probability_Visualizer to the engine's `subscribe`, with the Audio_Engine and the visualiser attached directly rather than through the queue because both must react within 100 ms and neither animates
    - Take the fake source out of the production path and verify a complete human-versus-The_Dead match runs from the first roll to `match_end`
    - _Requirements: 8.5, 8.6, 11.1, 12.2_

  - [ ] 11.4 `[M2]` Implement the degraded audio start and the audio gate
    - If `AudioContext` is undefined at load, start the match with `status: 'disabled'` and show a notice within 1000 ms of the interface becoming playable
    - If the context is `suspended` after construction, render an `AudioGate` control that stays visible until activated or the match ends, calling `resume()` from inside the gesture handler and discarding events received while suspended rather than queueing them
    - While audio is disabled, keep accepting rolls and move requests and keep updating the visualiser on every `roll` event
    - _Requirements: 19.1, 19.2, 19.6, 19.7_

  - [ ]* 11.5 `[M1]` Write unit tests for the controller timing
    - The 400–1200 ms pause after The_Dead's move with input refused, with fake timers
    - The strength setting applying only from the start of a match
    - The 500 ms fallback taking `legal[0]`
    - _Requirements: 7.4, 7.5, 7.7_

- [ ] 12. `[ambos]` Checkpoint — vertical slice playable end to end, Sun 15 November
  - A full match can be played in the browser against The_Dead with audio and the live distribution responding, ugly but complete
  - Ensure all tests pass, ask the user if questions arise

- [ ] 13. `[M1]` Property harness and the engine properties — Mon 16 to Thu 19 November
  - [ ] 13.1 Build the shared generators
    - Write `tests/engine/harness.ts`: `arbSeed` (non-empty printable string), `arbStrength`, `selfPlay(seed, strength)` playing a full match with both sides driven by the Opponent_Policy and returning the trace of every emitted event with the `MatchState` after each, and `arbReachableState` drawing a seed and a trace index
    - Reachable states come from the trace rather than from a hand-rolled state generator, so every generated state is legally reachable
    - Each property test below lives in its own file under `tests/engine/`, named after its property number, and carries the tag comment `// Feature: kur, Property N: <title>`
    - _Requirements: 7.1, 7.8_

  - [ ]* 13.2 Write the property test for square occupancy
    - **Property 3: At most one piece stands on any square**
    - **Validates: Requirements 1.8, 5.4**
    - Generator: `arbSeed` × `arbStrength` through `selfPlay`. Map every board location of both players through `PATHS` and assert no repeated square id at every state in the trace. `numRuns: 1000`

  - [ ]* 13.3 Write the property test for piece conservation
    - **Property 4: Each player always owns exactly seven pieces**
    - **Validates: Requirements 1.7, 5.2, 5.4, 6.5**
    - Generator: the same `selfPlay` trace. Assert `hand + board + retired === 7` per player at every state, and that the retired count never decreases between consecutive states. `numRuns: 1000`

  - [ ]* 13.4 Write the property test for the legal move set
    - **Property 5: The legal move set is sound, complete at its boundaries and duplicate-free**
    - **Validates: Requirements 3.2, 3.3, 6.1**
    - Generator: `arbReachableState` × `fc.integer({ min: 0, max: 4 })`. Assert no two moves share a `(from, to)` pair; every member applies and advances its piece by exactly the Roll_Value or retires it from position 15 − Roll_Value; the entry and retirement membership predicates, restated in the test independently of the production code, agree with the set; and the set is empty for a Roll_Value of 0

  - [ ]* 13.5 Write the property test for rejected requests
    - **Property 6: A rejected request changes nothing**
    - **Validates: Requirements 3.4, 3.5, 3.7, 3.8, 3.9, 4.5, 5.5, 6.6, 6.7**
    - Generator: `arbReachableState` × an adversarial request generator producing requests for the wrong player, requests with no pending roll, requests after `match_end`, piece ids outside 0–6, retired pieces, destinations holding an own piece, destinations past position 15, and rosette destinations holding an opposing piece. Assert that `!result.ok` implies deep equality of the state before and after and an empty emitted-event list, and that `result.error.rule` is the `RuleId` the generator intended

  - [ ]* 13.6 Write the property test for turn passing
    - **Property 7: Turns pass exactly when the rules say they pass**
    - **Validates: Requirements 3.1, 3.6, 4.2, 4.3, 4.4, 5.6**
    - Generator: the `selfPlay` trace read as an event sequence. Assert that the `player` of each next `roll` event follows from what happened in between, that no single rosette landing yields two consecutive extra rolls, and that no `move` event follows a Roll_Value of 0

  - [ ]* 13.7 Write the property test for termination
    - **Property 8: Every valid match ends, within 2000 rolls**
    - **Validates: Requirements 6.3, 6.4**
    - Generator: `arbSeed` × `fc.tuple(arbStrength, arbStrength)`. Assert exactly one `match_end` in the trace, `rollCount <= 2000`, the named winner holding 7 retired pieces when `reason === 'all_retired'` and at least as many as the loser when `reason === 'roll_cap'`, and that requests after the end are rejected with `'match_ended'`

  - [ ]* 13.8 Write the property test for the event stream
    - **Property 9: Events are numbered, faithful, frozen and delivered to everyone**
    - **Validates: Requirements 4.1, 5.1, 5.3, 6.2, 8.1, 8.2, 8.3, 8.6, 8.7, 8.8, 8.9**
    - Generator: `arbSeed` × `fc.integer({ min: 8, max: 16 })` consumer count, with one hostile mutating consumer inserted at a generated index. Assert gapless `seq` numbering from 1, identical sequences for every consumer in registration order with no consumer receiving event n+1 before another has received event n, that the hostile consumer's writes change nothing for the others, and that each payload agrees with the transition it reports

  - [ ]* 13.9 Write the property test for the opponent
    - **Property 10: The Dead plays legally and deterministically**
    - **Validates: Requirements 7.1, 7.3, 7.6, 7.8**
    - Generator: `arbReachableState` × `fc.integer({ min: 0, max: 4 })` × `arbStrength`. Assert membership of the returned move in the input set, `null` if and only if the set is empty, deep equality between two successive calls with the same arguments, and deep equality of the state before and after the call

- [ ] 14. `[M2]` Art direction pass and the accessibility floor — Mon 16 to Wed 18 November
  - [ ] 14.1 Apply the art direction to the board
    - Render the board, pieces and trays in the declared visual system of clay, cuneiform forms and lamplight, with no decorative imagery outside it
    - Keep each square at 56×56 px or more and each piece at 32 px or more across on a 1280×720 viewport, and keep the piece shapes distinguishable in greyscale
    - _Requirements: 15.4, 15.5, 16.5_

  - [ ] 14.2 Apply the typography and the readouts
    - At most two typefaces; all text at 16 px or more and 4.5:1 contrast or better; the current Roll_Value and the player to move at 32 px or more; the visualiser's numeric and Roll_Value labels at 16 px or more
    - Draw the focus indicator at 3:1 contrast or better against its background
    - _Requirements: 12.9, 15.6, 16.4_

  - [ ] 14.3 Implement the mute control
    - One control that mutes and unmutes the Audio_Engine, operable by keyboard alone, indicating its state by text or shape and not by colour alone, retained across matches for the rest of the browser session
    - While muted, sound no tone but keep delivering every event to the Board_Renderer and the Probability_Visualizer, leaving `MatchState` and the Event_Stream identical to the unmuted run
    - Activating it while tones are sounding stops every tone within 100 ms
    - _Requirements: 16.1, 16.2, 16.7_

  - [ ] 14.4 Implement the reduced-motion control
    - A session-scoped control that forces the single-frame position change regardless of the operating-system preference, and the operating-system preference path when the control is off
    - _Requirements: 15.3, 16.3, 16.8_

  - [ ]* 14.5 Write unit tests for the non-colour cues
    - Assert that piece ownership, legal-destination highlighting and the expected-against-observed distinction each carry a shape, pattern or text cue and remain identifiable in greyscale
    - _Requirements: 16.5, 16.9_

- [ ] 15. `[ambos]` Resilience, so a browser restriction never ends the lesson — Thu 19 to Sat 21 November
  - [ ] 15.1 `[M1]` Surface engine rejections in the interface
    - On a rejected human move request, leave the drawn board unchanged, retain the pending Roll_Value so another legal move can be submitted, and show the `Rejection.message` verbatim within 200 ms
    - _Requirements: 19.5_

  - [ ] 15.2 `[M2]` Implement the audio error path
    - try/catch around voice scheduling with a per-match error counter: stop the affected tone within 100 ms, record the error and keep accepting events; three consecutive errors in one match disable the Audio_Engine for the rest of it with a notice
    - _Requirements: 19.3, 19.8_

  - [ ] 15.3 `[M2]` Implement the Canvas 2D fallback
    - When `getContext('2d')` returns null, show a message naming the missing capability and the supported desktop browsers instead of a blank or partially drawn interface
    - _Requirements: 17.9_

  - [ ] 15.4 `[M2]` Implement the notation failure path
    - If the bundled `hymn-h6.kur` fails to parse at startup, start the match with audio disabled and show a message naming the offending symbol with its line and column as returned by the parser
    - _Requirements: 19.4_

  - [ ]* 15.5 `[ambos]` Write the property test for audio independence
    - **Property 15: Disabling audio changes nothing about the match**
    - **Validates: Requirements 16.2, 19.6**
    - Generator: `arbSeed` × `arbStrength`, run twice against the same engine configuration with and without the Audio_Engine subscribed. Assert deep equality of the two recorded event sequences including `seq` numbering, and that both traces contain a `match_end`

  - [ ]* 15.6 `[M2]` Write unit tests for the three degraded-start branches
    - Missing Canvas 2D, missing `AudioContext`, and a suspended context requiring the gate
    - _Requirements: 17.9, 19.1, 19.2_

- [ ] 16. `[ambos]` Checkpoint — recordable end-to-end demo, Sun 22 November (HARD MILESTONE)
  - A complete match on the public URL, with the art direction applied, audio responding, the distribution filling in and every degraded path surviving: enough to record the demo video against
  - If this checkpoint is missed, cut scope immediately. The video and the write-up are never cut
  - Ensure all tests pass, ask the user if questions arise

- [ ] 17. `[ambos]` Hardening to the freeze — Mon 23 to Thu 26 November
  - [ ] 17.1 `[M1]` Write the DOM-free match smoke test
    - `tests/smoke/headless-match.test.ts` deletes `globalThis.document` and `globalThis.window` and plays a full match to `match_end`, making Requirement 8.4 an executable claim rather than an assertion
    - _Requirements: 8.4_

  - [ ] 17.2 `[M1]` Add the import boundary rule
    - An ESLint rule forbidding DOM and React imports inside `src/engine`, and engine imports inside `src/audio` and `src/render`, wired into the CI run
    - _Requirements: 8.4, 8.5_

  - [ ] 17.3 `[M2]` Add the bundle budget gate
    - Fail the build if `dist` exceeds 5 MB, and fail it if any audio asset is present at all, which is how "every tone is synthesised at run time" stays true as the bundle grows
    - Report the first-load transfer size in the build output
    - _Requirements: 10.2, 10.3, 17.4, 17.7_

  - [ ] 17.4 `[ambos]` Fix every defect the suites found and leave them green
    - Work through the failures the property suite surfaced, replaying each from its reported seed, and the unit failures alongside
    - No new features from here; this task is the last one before the freeze
    - _Requirements: 1.7, 1.8, 3.2, 6.3, 6.4, 8.1, 8.3, 11.6, 12.2_

- [ ] 18. `[ambos]` Checkpoint — feature freeze, Fri 27 November
  - From here only bug fixes and polish. What is not in, does not go in
  - Ensure all tests pass, ask the user if questions arise

- [ ] 19. Deferred scope — post-freeze only, in the order the design recommends
  - [ ]* 19.1 `[M2]` Provenance_Panel (Requirement 18)
    - A React component plus a `provenance.ts` constant: the Finkel_Ruleset citation, the hymn transcription citation matching the one in `hymn-h6.kur`, the statement that every tone is synthesised at run time, the five items each labelled "attested by the artefact" or "supplied by modern reconstruction", the redistribution and licence statements, a control reachable before, during and after a match, and the missing-citation fallback
    - Cheapest of the deferred items and it strengthens the Resurrection framing, so it is first
    - _Requirements: 18.1, 18.2, 18.3, 18.4, 18.5, 18.6, 18.7, 18.8_

  - [ ]* 19.2 `[M2]` Match_Summary (Requirement 14)
    - A read-only view over data that already exists: the `match_end` event's `rollTally`, `captureCount` and `winner`, and the visualiser's observed counts. Observed count, observed relative frequency, expected probability and signed difference per Roll_Value to one decimal place, the sample-size statement, and the new-match reset
    - _Requirements: 14.1, 14.2, 14.5, 14.6, 14.7_

  - [ ]* 19.3 `[ambos]` Prediction_Mode (Requirement 13)
    - Controller-side state plus a comparison against the next `roll` event, which already carries the exact probability of the value rolled. No engine change. Selection required before each human roll including extra rolls, locked once the roll starts, the outcome display, the two counters, the disabled path and the mid-match toggle
    - _Requirements: 13.1, 13.2, 13.3, 13.4, 13.5, 13.6, 13.7, 13.8_

  - [ ]* 19.4 `[M2]` Full accessibility beyond the MVP floor (Requirement 16)
    - A live region fed from the existing event subscription, describing the event kind, the player to move, the current Roll_Value and both retired counts within 500 ms, and a full keyboard path over the board attaching to the existing `onSquareClick` handler, with no focus trap
    - _Requirements: 16.4, 16.6_

  - [ ]* 19.5 `[M1]` Match_Record and local storage (Requirement 20)
    - Serialisation and parsing of the ordered Roll_Values, the ordered Moves, the winner, the roll count and the seed, all of which the engine already holds; replay as a loop over `submitMove`; the 2000-roll limit, the two rejection paths, and local storage of the 20 most recent records with the storage-failure fallback
    - _Requirements: 20.1, 20.2, 20.3, 20.5, 20.6, 20.7, 20.8_

  - [ ]* 19.6 `[M1]` Property test for the Match_Record round trip
    - **Property 16: Match_Record serialisation round trip**
    - **Validates: Requirements 20.4**
    - Generator: an arbitrary Match_Record of 0 to 2000 rolls. Assert that serialising then parsing returns a record whose Roll_Values, Moves, winner, roll count and seed are each equal to the original

## Notes

- **Owners.** `[M1]` is the engine lane and Properties 1–10; `[M2]` is the audio, render and interface lane and Properties 11–14; `[ambos]` marks only what genuinely needs agreement: the event contract, the engine wiring, Property 15 and the checkpoints.
- **Task 1.2 is a blocking gate.** `src/engine/events.ts` is written and signed off before any other implementation task begins. From that moment M2 works against `src/devtools/fakeEventSource.ts` and never reads `MatchState`, so the two lanes do not wait on each other until task 11.
- **Deployment is early, not late.** Task 1.4 puts the project on a public URL by Sunday 8 November. It is a submission requirement, and discovering a Pages base-path problem in the last week would be a new problem at the worst moment.
- **The 22 November milestone drives the order.** A playable vertical slice (task 12) comes before the perfection of any single layer. The property suite, the art pass and the resilience work all land between the slice and the recordable demo.
- **Tasks marked `*` are optional and can be skipped for a faster MVP.** In this plan they are the test sub-tasks and the whole of the deferred task 19. The property tests are a scored criterion of the hackathon, so skipping them costs points even though the application still runs; the deferred items in task 19 only start once the video and the write-up are done.
- **One property, one test.** Each of Properties 1–15 is implemented by exactly one property-based test with Vitest and `fast-check` through the `@fast-check/vitest` binding, in its own file, tagged `// Feature: kur, Property N: <title>`. Minimum `numRuns` is 100, raised to 1000 for Properties 1, 3, 4 and 12, which are cheap. Property-based testing is not implemented from scratch.
- **Half a day to a day each.** No task here is sized in weeks. Eighty-two leaf tasks, seventy-six of them inside the MVP, against roughly forty person-days of two-developer capacity between 3 and 27 November. The optional test sub-tasks are what absorbs the slippage that will happen.
- **Not in this list, because they are not coding tasks:** the manual browser matrix on the current and previous stable Chrome, Firefox, Safari and Edge at 1280×720, the mid-match network-loss run, the throttled first-load measurement and the projector legibility check, all of which belong in `docs/submission-checklist.md` before the 27 November freeze; and the demo video, the Kiro write-up and the Devpost submission itself.
- **No end-to-end browser automation.** Playwright against a single-page static app would duplicate what the property harness and the manual matrix already cover, and the days it costs are worth more in the video.

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1"] },
    { "id": 1, "tasks": ["1.2"] },
    { "id": 2, "tasks": ["1.3", "1.4", "1.5", "1.8"] },
    { "id": 3, "tasks": ["1.6", "1.9"] },
    { "id": 4, "tasks": ["1.7"] },
    { "id": 5, "tasks": ["2"] },
    { "id": 6, "tasks": ["3.1", "7.1", "9.1"] },
    { "id": 7, "tasks": ["3.2", "7.2", "9.2"] },
    { "id": 8, "tasks": ["3.3", "3.4", "3.5", "7.3", "9.3"] },
    { "id": 9, "tasks": ["4.1", "7.4", "9.4"] },
    { "id": 10, "tasks": ["4.2", "7.5", "9.5"] },
    { "id": 11, "tasks": ["4.3", "7.6", "9.6"] },
    { "id": 12, "tasks": ["4.4", "7.7", "10.1"] },
    { "id": 13, "tasks": ["5.1", "8.1", "10.2"] },
    { "id": 14, "tasks": ["5.2", "8.2", "10.3"] },
    { "id": 15, "tasks": ["5.3", "8.3"] },
    { "id": 16, "tasks": ["5.4", "8.4", "11.2"] },
    { "id": 17, "tasks": ["6.1", "11.4"] },
    { "id": 18, "tasks": ["11.1"] },
    { "id": 19, "tasks": ["11.3", "11.5"] },
    { "id": 20, "tasks": ["12"] },
    { "id": 21, "tasks": ["13.1", "14.1"] },
    { "id": 22, "tasks": ["13.2", "13.3", "14.2"] },
    { "id": 23, "tasks": ["13.4", "13.5", "14.3"] },
    { "id": 24, "tasks": ["13.6", "13.7", "14.4"] },
    { "id": 25, "tasks": ["13.8", "13.9", "14.5"] },
    { "id": 26, "tasks": ["15.1", "15.2"] },
    { "id": 27, "tasks": ["15.3", "15.4"] },
    { "id": 28, "tasks": ["15.5", "15.6"] },
    { "id": 29, "tasks": ["16"] },
    { "id": 30, "tasks": ["17.1", "17.3"] },
    { "id": 31, "tasks": ["17.2"] },
    { "id": 32, "tasks": ["17.4"] },
    { "id": 33, "tasks": ["18"] },
    { "id": 34, "tasks": ["19.1"] },
    { "id": 35, "tasks": ["19.2"] },
    { "id": 36, "tasks": ["19.3"] },
    { "id": 37, "tasks": ["19.4"] },
    { "id": 38, "tasks": ["19.5"] },
    { "id": 39, "tasks": ["19.6"] }
  ]
}
```
