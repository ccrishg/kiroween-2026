# Requirements Document

## Introduction

KUR is a browser application for the Kiroween 2026 hackathon, entered in the Resurrection category. KUR resurrects two deciphered artefacts and joins them into one system:

1. **The Royal Game of Ur**, a Sumerian board game of roughly 4,500 years ago, excavated at Ur and playable today through the rule set reconstructed by Irving Finkel from the Babylonian cuneiform tablet BM 33333B.
2. **Hurrian Hymn no. 6** (circa 1400 BC, Ugarit), the oldest substantially complete notated music that survives. The cuneiform notation of the hymn describes *intervals* rather than absolute pitches, so KUR implements a tuning system from frequency ratios and synthesises every tone at run time.

The British Museum records that the central squares of the Ur board were also used for divination. KUR takes that as its setting: a single human player plays against a machine opponent called The_Dead, each roll is framed as an omen, and the visual language is archaeological rather than generically festive.

The target audience is secondary and upper-secondary classrooms. A roll in the Royal Game of Ur uses four tetrahedral dice with two marked vertices each, so the result is a binomial variable B(4, 1/2) over the values 0 to 4 with probabilities 1/16, 4/16, 6/16, 4/16 and 1/16. KUR makes that distribution part of the interface instead of hiding it: a live visualisation, a prediction mode in which the player commits to a guess before rolling, and an end-of-match summary comparing observed frequencies against expected frequencies. The teaching claim is that playing the game builds probability intuition.

KUR runs entirely in the browser as a static site: TypeScript, Vite and React for the interface, the native Web Audio API for synthesis, and Canvas 2D for the board and the visualisation. There is no backend, no account and no server-held state. The Game_Engine is a pure TypeScript library and communicates with the rest of the application through one ordered Event_Stream, which is the only agreed interface between the two developers working in parallel.

**Out of scope for this spec:** online multiplayer, user accounts, any server-side persistence, a native mobile application, Hurrian hymns other than h.6, and a three-dimensional board.

## Glossary

- **KUR**: The complete browser application described by this document, comprising Game_Engine, Audio_Engine, Board_Renderer, Probability_Visualizer and the surrounding interface.
- **Game_Engine**: The pure TypeScript library that holds the rules of the Royal Game of Ur under the Finkel_Ruleset, validates moves, owns Match_State and emits the Event_Stream. The Game_Engine references no DOM API and no user-interface framework.
- **Finkel_Ruleset**: The reconstruction of the rules of the Royal Game of Ur derived from tablet BM 33333B, used by KUR as the single authority on legal play.
- **Match_State**: The complete state of one match: the position of every Piece, the Player to move, the pending Roll_Value and the match outcome.
- **Player**: One of the two participants in a match. The two Players are the Human_Player and The_Dead.
- **Human_Player**: The Player controlled through the interface by the person using KUR.
- **The_Dead**: The machine-controlled Player, selected by the Opponent_Policy.
- **Opponent_Policy**: The decision procedure that selects one legal Move for The_Dead.
- **Piece**: One of the seven tokens belonging to a Player. A Piece is in exactly one of three places: Hand, Board or Retired.
- **Hand**: The set of Pieces of a Player that have not yet entered the Board.
- **Retired**: The set of Pieces of a Player that have completed the Path and left the Board.
- **Board**: The 20 squares of the Royal Game of Ur, arranged as a middle row of 8 squares and two outer rows of 6 squares.
- **Path**: The ordered sequence of 14 squares that a Piece of a given Player traverses, consisting of 4 squares of the own outer row, the 8 shared squares of the middle row, and the final 2 squares of the own outer row.
- **Shared_Square**: Any of the 8 squares of the middle row, which belong to the Path of both Players.
- **Rosette_Square**: Any of the 5 squares of the Board marked with a rosette. A Piece standing on a Rosette_Square cannot be captured, and a Move ending on a Rosette_Square grants the moving Player an additional Roll.
- **Dice_Model**: The component that produces a Roll_Value and exposes the exact probability of each Roll_Value.
- **Roll**: One act of throwing the four tetrahedral dice.
- **Roll_Value**: The integer result of a Roll, in the range 0 to 4 inclusive, distributed as B(4, 1/2).
- **Move**: The transfer of one Piece forward along the Path by exactly the current Roll_Value squares, including entry from Hand and retirement from the Board.
- **Event_Stream**: The ordered sequence of immutable events emitted by the Game_Engine. The event kinds are `roll`, `move`, `capture`, `rosette`, `retire` and `match_end`.
- **Audio_Engine**: The component that derives frequencies from the Interval_Sequence of Hurrian Hymn no. 6 and synthesises audio through the Web Audio API in response to the Event_Stream.
- **Hymn_Notation**: The textual encoding, bundled with KUR, of the interval names recorded in the cuneiform notation of Hurrian Hymn no. 6.
- **Notation_Parser**: The component that reads a Hymn_Notation document and produces an Interval_Sequence.
- **Notation_Printer**: The component that formats an Interval_Sequence back into a Hymn_Notation document.
- **Interval_Sequence**: The ordered list of intervals obtained from Hymn_Notation, where each interval is a frequency ratio expressed as a pair of integers.
- **Tuning_System**: The mapping from a Tonic_Frequency and an Interval_Sequence to absolute frequencies, computed with just-intonation integer ratios.
- **Tonic_Frequency**: The reference frequency, in hertz, from which the Tuning_System derives all other frequencies.
- **Board_Renderer**: The Canvas 2D component that draws the Board, the Pieces and the Move affordances.
- **Probability_Visualizer**: The Canvas 2D component that draws the expected distribution of Roll_Value together with the frequencies observed so far in the current match.
- **Prediction_Mode**: The optional mode in which the Human_Player commits to a predicted Roll_Value before each own Roll and receives the outcome of that prediction.
- **Match_Summary**: The end-of-match report comparing observed Roll_Value frequencies with expected frequencies and reporting Prediction_Mode accuracy when Prediction_Mode is enabled.
- **Match_Record**: The serialisable text record of one completed match, containing five elements: the ordered Roll_Values of the match, the ordered Moves of the match, the winning Player, the total Roll count, and, where a fixed seed was supplied to the Dice_Model, that seed.
- **Provenance_Panel**: The interface surface that states the archaeological sources of the rules and of the hymn, and the licensing position of the generated audio.
- **Judging_Period**: The hackathon evaluation window, ending 29 January 2027, during which KUR must remain publicly reachable.

## Requirements

### Requirement 1: Board and match setup

**User Story:** As a student opening KUR for the first time, I want a match that is already set up and ready to play, so that I can start playing without reading a manual.

#### Acceptance Criteria

1. THE Game_Engine SHALL represent a Board of exactly 20 squares composed of one middle row of exactly 8 Shared_Squares and two outer rows of exactly 6 squares each, where each outer row belongs to exactly one Player and no square of an outer row appears in the Path of the other Player.
2. THE Game_Engine SHALL mark exactly 5 squares of the Board as Rosette_Squares: the square at position 4 of the Path of each Player, the square at position 14 of the Path of each Player, and the single Shared_Square at position 8 of the Path of both Players.
3. THE Game_Engine SHALL define for each Player a Path of exactly 14 positions numbered 1 to 14 in order of travel, where positions 1 to 4 are 4 squares of the own outer row, positions 5 to 12 are the 8 Shared_Squares in one fixed order, and positions 13 and 14 are the remaining 2 squares of the own outer row.
4. THE Game_Engine SHALL make positions 5 to 12 the only positions at which the Path of one Player and the Path of the other Player refer to the same Board square.
5. WHEN a new match starts, THE Game_Engine SHALL place all 7 Pieces of each Player in the Hand of that Player, with 0 Pieces on the Board and 0 Pieces Retired for each Player.
6. WHEN a new match starts, THE Game_Engine SHALL set the Human_Player as the Player to move, set the pending Roll_Value of the Match_State to none, and record the match outcome as undecided.
7. THE Game_Engine SHALL keep the count of Pieces in Hand plus Pieces on Board plus Pieces Retired equal to exactly 7 for each Player in every Match_State, including every Match_State reached during the application of a Move.
8. THE Game_Engine SHALL permit at most 1 Piece in total, counting the Pieces of both Players, on any square of the Board, including every Shared_Square, in every Match_State.
9. WHEN the KUR interface finishes loading, THE KUR interface SHALL present a match in the state defined by criteria 5 and 6, in which the only interaction required to begin play is the Roll control of the Human_Player and no configuration input is required.

### Requirement 2: Dice model

**User Story:** As a probability teacher, I want the dice of the game to follow the historical four-tetrahedron throw with exact published probabilities, so that I can use the game as a worked example of a binomial variable.

#### Acceptance Criteria

1. WHEN a Roll is requested, THE Dice_Model SHALL produce exactly one Roll_Value equal to the number of the four dice that show a marked vertex, which is an integer in the range 0 to 4 inclusive.
2. THE Dice_Model SHALL expose the probability of each of the five Roll_Values as an exact rational, expressed as a pair of positive integers, equal to 1/16, 4/16, 6/16, 4/16 and 1/16 for the Roll_Values 0, 1, 2, 3 and 4 respectively, with no rounding to a decimal approximation.
3. THE Dice_Model SHALL report the sum of the five exposed probabilities as the exact rational 1/1.
4. THE Dice_Model SHALL produce each Roll_Value by simulating four independent two-outcome dice, each with probability 1/2 of a marked vertex, and SHALL make the outcome of a Roll independent of every previous Roll_Value and of Match_State.
5. WHERE a seed accepted by the Dice_Model is supplied, THE Dice_Model SHALL produce, on every run, an identical ordered sequence for at least the first 10000 Roll_Values generated from that seed.
6. WHEN 100000 consecutive Roll_Values are produced, THE Dice_Model SHALL produce observed relative frequencies that differ from 1/16, 4/16, 6/16, 4/16 and 1/16 by at most 0.01 in absolute value for the Roll_Values 0, 1, 2, 3 and 4 respectively.
7. WHERE no seed is supplied, THE Dice_Model SHALL select its own seed and SHALL expose that seed, so that the sequence of Roll_Values of the run can be reproduced by supplying the exposed seed.
8. IF a supplied seed is not a value that the Dice_Model accepts, THEN THE Dice_Model SHALL produce no Roll_Value and SHALL return an error indicating that the supplied seed was rejected.

### Requirement 3: Legal moves

**User Story:** As a player, I want KUR to allow only the moves that the reconstructed rules allow, so that I learn the historical game and not an approximation.

#### Acceptance Criteria

1. WHEN a Roll_Value of 0 is produced, THE Game_Engine SHALL apply no Move, leave the position of every Piece unchanged and pass the turn to the other Player.
2. WHEN a Roll_Value between 1 and 4 inclusive is produced, THE Game_Engine SHALL compute, within 100 milliseconds, the set of legal Moves of the Player to move, containing every Move permitted by the Finkel_Ruleset exactly once and no other Move.
3. WHERE the Player to move holds at least one Piece in Hand, WHEN a Roll_Value between 1 and 4 inclusive is produced, THE Game_Engine SHALL include in the set of legal Moves the entry of one Piece from Hand onto the Path position numbered equal to the Roll_Value.
4. IF a Move whose destination square already holds a Piece of the moving Player is submitted, THEN THE Game_Engine SHALL exclude that Move from the set of legal Moves, leave Match_State unchanged and return a rejection that identifies the violated rule.
5. IF a Move whose destination lies beyond Path position 14 is submitted and that Move is not the retirement Move defined in Requirement 6, THEN THE Game_Engine SHALL exclude that Move from the set of legal Moves, leave Match_State unchanged and return a rejection that identifies the violated rule.
6. IF the set of legal Moves is empty for the Player to move, THEN THE Game_Engine SHALL leave the position of every Piece unchanged, emit no `move` event and pass the turn to the other Player.
7. IF a Move request that is absent from the set of legal Moves is submitted, THEN THE Game_Engine SHALL leave Match_State unchanged, emit no event of the Event_Stream and return a rejection that identifies the violated rule.
8. IF a Move request is submitted while no Roll_Value is pending, or is submitted for a Player that is not the Player to move, THEN THE Game_Engine SHALL leave Match_State unchanged and return a rejection that identifies the violated rule.
9. IF a Move request names a Piece that is neither in the Hand nor on the Board of the Player to move, THEN THE Game_Engine SHALL leave Match_State unchanged and return a rejection that identifies the violated rule.
10. WHEN a legal Move is applied, THE Game_Engine SHALL advance the moved Piece forward by exactly the current Roll_Value positions along the Path of the moving Player, consume the pending Roll_Value and leave the position of every other Piece of the moving Player unchanged.

### Requirement 4: Rosette squares

**User Story:** As a player, I want rosette squares to grant shelter and another throw, so that the tactical shape of the original game is preserved.

#### Acceptance Criteria

1. WHEN a Move ends on a Rosette_Square, THE Game_Engine SHALL emit a `rosette` event that names the moving Player and the Rosette_Square reached.
2. WHEN a Move ends on a Rosette_Square, THE Game_Engine SHALL grant the moving Player exactly one additional Roll, independently of the number of Rosette_Squares that Player occupies, and SHALL keep that Player as the Player to move until that additional Roll is resolved.
3. WHEN an additional Roll granted under criterion 2 resolves into an applied Move whose destination is not a Rosette_Square, including a Move that retires a Piece, THE Game_Engine SHALL end the turn of the moving Player and SHALL grant no further additional Roll.
4. IF an additional Roll granted under criterion 2 produces a Roll_Value of 0 or an empty set of legal Moves, THEN THE Game_Engine SHALL end the turn of the moving Player and SHALL grant no further additional Roll.
5. IF the destination of a Move of the Player to move is a Rosette_Square that holds a Piece of the opposing Player, THEN THE Game_Engine SHALL omit that Move from the set of legal Moves, SHALL leave Match_State unchanged if that Move is submitted, and SHALL emit no `capture` event.
6. THE Board_Renderer SHALL draw each of the 5 Rosette_Squares with a rosette marking and with a fill that differs from the fill of the other 15 squares at a contrast ratio of at least 3 to 1 on a viewport of 1280 by 720 pixels.

### Requirement 5: Capture

**User Story:** As a player, I want to send an opposing piece back to its owner's hand when I land on it, so that the central row carries real risk.

#### Acceptance Criteria

1. WHEN a Move ends on a Shared_Square that is not a Rosette_Square and that holds a Piece of the opposing Player, THE Game_Engine SHALL return that opposing Piece to the Hand of the opposing Player and SHALL emit a `capture` event identifying the captured Piece, the Player owning the captured Piece and the Shared_Square on which the capture occurred.
2. WHEN a Piece is returned to a Hand by a capture, THE Game_Engine SHALL discard all Path progress of that Piece and SHALL allow that Piece to re-enter the Board only through the entry Move defined in Requirement 3, onto the Path square of the owning Player at the position equal to the Roll_Value of a later Roll of that Player.
3. THE Game_Engine SHALL emit a `capture` event only for a Move that ends on one of the 8 Shared_Squares, and SHALL leave every Piece of the opposing Player unchanged for a Move that ends on any square that is not a Shared_Square.
4. WHEN a capture occurs, THE Game_Engine SHALL remove the captured Piece from the Shared_Square before placing the moving Piece on that Shared_Square, so that no Match_State observable through the Event_Stream holds two Pieces on one square and the count of Pieces in Hand plus Pieces on Board plus Pieces Retired remains 7 for each Player.
5. IF a Move is requested whose destination is a Rosette_Square that holds a Piece of the opposing Player, THEN THE Game_Engine SHALL leave Match_State unchanged, SHALL emit no `capture` event and SHALL return a rejection naming the shelter rule of Rosette_Squares.
6. WHEN a `capture` event is emitted, THE Game_Engine SHALL grant no additional Roll for that capture and SHALL pass the turn to the opposing Player after that Move.

### Requirement 6: Retirement, victory and termination

**User Story:** As a player, I want a clear ending with a declared winner, so that a match is a complete experience I can finish inside one class period.

#### Acceptance Criteria

1. IF the sum of the current Path position of a Piece of the Player to move and the current Roll_Value equals exactly 15, which is one position past the last of the 14 squares of the Path, THEN THE Game_Engine SHALL include the retirement Move of that Piece in the set of legal Moves.
2. WHEN a retirement Move is applied, THE Game_Engine SHALL remove that Piece from its Board square, place that Piece in Retired, increase the count of Retired Pieces of that Player by exactly 1, and emit a `retire` event naming that Player.
3. WHEN a `retire` event raises the count of Retired Pieces of a Player to 7, THE Game_Engine SHALL emit exactly one `match_end` event naming that Player as the winner before accepting any further Roll, and SHALL reject every further Move request with Match_State unchanged.
4. WHILE a match is in progress in which every turn either applies a Move taken from the set of legal Moves produced by the Game_Engine or ends with no Move because the set of legal Moves is empty or the Roll_Value is 0, THE Game_Engine SHALL emit a `match_end` event within 2000 Rolls counted from the start of that match.
5. THE Game_Engine SHALL never decrease the count of Retired Pieces of either Player while a match is in progress.
6. IF a Move request would place a Piece at a Path position greater than 15, THEN THE Game_Engine SHALL leave Match_State unchanged, emit no `retire` event, and return a rejection naming the exact-Roll_Value retirement rule.
7. IF a Move request names a Piece that is in Retired, THEN THE Game_Engine SHALL leave Match_State unchanged and return a rejection stating that a Piece in Retired cannot re-enter play.
8. WHEN a `match_end` event is emitted, THE Game_Engine SHALL expose the Match_Summary data defined in Requirement 14 within 1000 milliseconds of that event and SHALL keep that data available until a new match starts.

### Requirement 7: The Dead as opponent

**User Story:** As a single student at a single machine, I want a machine opponent that plays the game competently, so that I can play a full match alone.

#### Acceptance Criteria

1. WHEN the turn passes to The_Dead, or The_Dead is granted an additional Roll by a Rosette_Square, and the set of legal Moves produced by the Game_Engine for the pending Roll_Value contains at least one Move, THE Opponent_Policy SHALL select exactly one Move from that set and submit it to the Game_Engine.
2. WHEN the Game_Engine emits a `roll` event for The_Dead, THE Opponent_Policy SHALL return the selected Move within 500 milliseconds of that event and SHALL occupy the browser main thread for no more than 50 milliseconds in any single continuous span.
3. THE Opponent_Policy SHALL select Moves only from the set of legal Moves produced by the Game_Engine for the pending Roll_Value.
4. WHEN The_Dead completes a Move, THE KUR interface SHALL pause for a duration between 400 and 1200 milliseconds inclusive before the next Roll, including after each Move made on an additional Roll granted by a Rosette_Square, and SHALL accept no Roll input and no Move input from the Human_Player for the duration of that pause, so that the Human_Player can read the change on the Board.
5. WHERE the interface offers an opponent strength setting, THE Opponent_Policy SHALL apply the strength selected at the start of the match to every Move of The_Dead for the remainder of that match, and SHALL apply a strength selected during a match only from the start of the next match.
6. IF the turn passes to The_Dead and the set of legal Moves produced by the Game_Engine for the pending Roll_Value is empty, THEN THE Opponent_Policy SHALL submit no Move, SHALL leave Match_State unchanged and SHALL report within 500 milliseconds that no Move is available.
7. IF the Opponent_Policy has not selected a Move 500 milliseconds after the `roll` event for The_Dead, THEN THE Opponent_Policy SHALL stop evaluating further Moves and SHALL submit the first Move of the set of legal Moves produced by the Game_Engine for that Roll_Value.
8. WHEN the Opponent_Policy is given the same Match_State, the same pending Roll_Value and the same opponent strength setting on two separate occasions, THE Opponent_Policy SHALL select the same Move on both occasions.

### Requirement 8: Event stream boundary

**User Story:** As one of two developers building KUR in parallel, I want a single documented event interface between the engine and the presentation layers, so that audio and visualisation can be built without reading engine internals.

#### Acceptance Criteria

1. THE Game_Engine SHALL emit an ordered Event_Stream whose event kinds are `roll`, `move`, `capture`, `rosette`, `retire` and `match_end`, numbered consecutively from 1 within a match with no gap and no repeated number.
2. THE Game_Engine SHALL include in every event of the Event_Stream the event kind, the event number and the Player to which the event belongs, so that no consumer reads Match_State directly.
3. THE Game_Engine SHALL emit event objects whose values, including nested values, remain equal to the emitted values for every consumer irrespective of any write attempted by any consumer on an emitted event.
4. THE Game_Engine SHALL import no DOM API and no user-interface framework module, and SHALL play a complete match to a `match_end` event in a TypeScript runtime that exposes no DOM.
5. THE Audio_Engine, THE Board_Renderer and THE Probability_Visualizer SHALL derive every rendered output from the Event_Stream alone and SHALL read no Match_State.
6. WHEN an event is emitted, THE Game_Engine SHALL deliver that event, in registration order, to every consumer registered at the moment of emission and SHALL support at least 8 registered consumers, before emitting any further event and before applying the next Move.
7. WHEN a `roll` event is emitted, THE Game_Engine SHALL include in that event the Roll_Value in the range 0 to 4 inclusive, the exact probability of that Roll_Value as reported by the Dice_Model, whether the Roll is an additional Roll granted by a Rosette_Square, the count of Rolls of the current match including this Roll, and the set of legal Moves for that Roll_Value stated as origin and destination Path positions.
8. WHEN a `move` event is emitted, THE Game_Engine SHALL include in that event the origin of the moved Piece as either Hand or a Path position from 1 to 14 inclusive, the destination as a Path position from 1 to 14 inclusive, the ordered Path positions traversed, and the count of Pieces of each Player standing after the Move on a Shared_Square that is not a Rosette_Square.
9. WHEN a `capture`, `rosette`, `retire` or `match_end` event is emitted, THE Game_Engine SHALL include in that event: for `capture`, the Path position of the Shared_Square of the capture and the resulting Hand count from 1 to 7 inclusive of the Player losing the Piece; for `rosette`, the Path position of the Rosette_Square and the Player granted the additional Roll; for `retire`, the resulting Retired count from 1 to 7 inclusive of the retiring Player; for `match_end`, the winning Player, the total count of Rolls, the observed count of each of the five Roll_Values and the total count of captures of the match.

### Requirement 9: Hymn notation parsing and printing

**User Story:** As a developer implementing the music, I want the interval notation of the hymn to live in a readable data file with a parser and a printer, so that the musical source is auditable and the implementation is testable.

#### Acceptance Criteria

1. THE Notation_Parser SHALL parse a Hymn_Notation document of up to 20000 characters into an Interval_Sequence of 1 to 500 intervals in which each interval is an ordered pair of integers whose two members each lie in the range 1 to 1024 inclusive, and SHALL preserve the order in which the intervals occur in the document.
2. IF a Hymn_Notation document contains a symbol that is not a member of the interval-name vocabulary declared in that document, THEN THE Notation_Parser SHALL return an error that names the first such symbol together with its line number and its character position within that line, and SHALL return no Interval_Sequence.
3. THE Notation_Printer SHALL format an Interval_Sequence into a Hymn_Notation document that the Notation_Parser accepts, and SHALL produce the same document on every formatting of the same Interval_Sequence.
4. WHEN THE Notation_Printer formats an Interval_Sequence whose intervals satisfy the bounds stated in criterion 1 and THE Notation_Parser parses the resulting document, THE Notation_Parser SHALL return an Interval_Sequence of the same length as the original Interval_Sequence whose intervals are pairwise identical, in the same order, to the intervals of the original Interval_Sequence.
5. THE Hymn_Notation document bundled with KUR SHALL cite the published transcription from which the interval data is taken by naming its author, its title and its year of publication, and SHALL reproduce no part of that publication other than the interval names and the citation itself.
6. THE Hymn_Notation document bundled with KUR SHALL declare the complete interval-name vocabulary that THE Notation_Parser accepts, stating for each interval name the pair of integers that the Notation_Parser produces for it.
7. IF a Hymn_Notation document encodes fewer than 1 interval, encodes more than 500 intervals, or encodes an interval with a member outside the range 1 to 1024 inclusive, THEN THE Notation_Parser SHALL return an error that names the violated bound and the position in the document at which the violation is detected, and SHALL return no Interval_Sequence.
8. THE Hymn_Notation document bundled with KUR SHALL state the licensing position of the cited transcription, including whether the interval data may be redistributed as part of KUR.

### Requirement 10: Tuning system and synthesis

**User Story:** As a listener, I want to hear tones built from the ancient interval system and generated by the application itself, so that the music is a reconstruction rather than a recording.

#### Acceptance Criteria

1. THE Tuning_System SHALL compute the frequency of each interval of the Interval_Sequence by multiplying the Tonic_Frequency, whose default value is 220 hertz, by the integer numerator of that interval and dividing by the integer denominator of that interval, and SHALL return a value within 0.01 hertz of the exact rational result.
2. THE Audio_Engine SHALL generate every tone through Web Audio API synthesis nodes created at run time, and SHALL decode no audio file for any tone.
3. THE Audio_Engine SHALL contain no recorded audio sample of any performance of Hurrian Hymn no. 6, SHALL contain no recorded audio sample of any other performance, and SHALL request no audio data from any network source while a match is running.
4. For every interval of the Interval_Sequence and every accepted Tonic_Frequency, THE Audio_Engine SHALL produce only frequencies within the range 110 hertz to 1760 hertz inclusive, which lies inside the audible range of 20 hertz to 20000 hertz.
5. IF the Tuning_System computes a frequency below 110 hertz, THEN THE Tuning_System SHALL double that frequency repeatedly until the frequency lies within 110 hertz to 1760 hertz inclusive, and IF the Tuning_System computes a frequency above 1760 hertz, THEN THE Tuning_System SHALL halve that frequency repeatedly until the frequency lies within that same range, in both cases completing the transposition within at most 10 octave steps.
6. THE Audio_Engine SHALL keep the peak absolute sample value of its summed output at or below 1.0, equal to 0 decibels full scale, at every moment of a match, so that the output does not clip.
7. IF an interval of the Interval_Sequence has a numerator or a denominator that is not a positive integer, THEN THE Tuning_System SHALL compute no frequency for that interval, SHALL return an error that names the position of that interval in the Interval_Sequence, and THE Audio_Engine SHALL sound no tone for that interval and SHALL continue to accept further events.
8. WHEN the Audio_Engine sounds a tone, THE Audio_Engine SHALL set the fundamental frequency of that tone to the frequency computed by the Tuning_System within a tolerance of 0.1 percent of that computed frequency.
9. WHERE a Tonic_Frequency other than 220 hertz is configured, THE Tuning_System SHALL accept the configured value only within the range 110 hertz to 440 hertz inclusive, and SHALL otherwise retain the previously accepted Tonic_Frequency and return an error indicating that the configured value is out of range.

### Requirement 11: Audio response to the match

**User Story:** As a player, I want the music to answer what happens on the board, so that the match and the hymn are one system rather than a game with background music.

#### Acceptance Criteria

1. WHEN a `roll` event is received, THE Audio_Engine SHALL sound, within 100 milliseconds of receiving that event and for a duration between 200 and 1500 milliseconds, one tone whose frequency is derived from the degree of the Interval_Sequence mapped to the Roll_Value carried by that event, where each of the five Roll_Values 0 to 4 is mapped to a distinct degree and the same Roll_Value selects the same degree on every `roll` event of the match.
2. WHEN a `capture` event is received, THE Audio_Engine SHALL sound, within 100 milliseconds of receiving that event and for a duration between 200 and 1500 milliseconds, the interval of the Interval_Sequence assigned to capture, which SHALL be the same interval on every `capture` event and SHALL differ from the interval assigned to shelter.
3. WHEN a `rosette` event is received, THE Audio_Engine SHALL sound, within 100 milliseconds of receiving that event and for a duration between 200 and 1500 milliseconds, the interval of the Interval_Sequence assigned to shelter, which SHALL be the same interval on every `rosette` event.
4. WHEN a `match_end` event is received, THE Audio_Engine SHALL begin sounding the closing phrase, defined as the final 4 intervals of the Interval_Sequence in their recorded order, within 100 milliseconds of receiving that event, and SHALL stop every sounding tone no later than 2000 milliseconds after receiving that event.
5. WHILE the count of `capture` events received since the most recent `match_end` event, or since the first event of the Event_Stream when no `match_end` event has been received, whose captured Player is the Human_Player is 2 or greater, THE Audio_Engine SHALL raise the register of every tone it sounds by exactly one interval step of the Interval_Sequence, within the frequency range required by Requirement 10.
6. THE Audio_Engine SHALL sound at most 8 simultaneous tones.
7. WHEN a `move` event is received, THE Audio_Engine SHALL sound, within 100 milliseconds of receiving that event and for a duration between 100 and 800 milliseconds, one tone derived from the Interval_Sequence.
8. WHEN a `retire` event is received, THE Audio_Engine SHALL sound, within 100 milliseconds of receiving that event and for a duration between 200 and 1500 milliseconds, the interval of the Interval_Sequence assigned to retirement, which SHALL differ from the intervals assigned to capture and to shelter.
9. IF an event that requires a tone is received while 8 tones are already sounding, THEN THE Audio_Engine SHALL stop the longest-sounding tone before sounding the tone for that event, so that the count of simultaneous tones never exceeds 8.

### Requirement 12: Live probability visualisation

**User Story:** As a probability student, I want to see the exact distribution of the dice and what the match has actually rolled so far, so that I can watch observed frequencies approach expected frequencies.

#### Acceptance Criteria

1. THE Probability_Visualizer SHALL display, simultaneously and at all times, the expected probability reported by the Dice_Model for each of the five Roll_Values 0, 1, 2, 3 and 4, as the exact values 1/16, 4/16, 6/16, 4/16 and 1/16 respectively.
2. WHEN a `roll` event carrying a Roll_Value in the range 0 to 4 inclusive is received, THE Probability_Visualizer SHALL increase the observed count of that Roll_Value by 1 and SHALL display, within 100 milliseconds of receiving the event, both the observed count and the observed relative frequency of each of the five Roll_Values, where the observed relative frequency of a Roll_Value is its observed count divided by the total number of Rolls recorded in the current match.
3. THE Probability_Visualizer SHALL plot the observed relative frequency and the expected probability of each Roll_Value against one shared axis that starts at 0 and ends at a value no smaller than the greater of the largest expected probability and the largest observed relative frequency, so that the two quantities for a given Roll_Value are directly comparable in length.
4. THE Probability_Visualizer SHALL display the total number of Rolls recorded in the current match as an integer, for totals from 0 to 2000 inclusive.
5. WHILE the total number of Rolls in the current match is 0, THE Probability_Visualizer SHALL display the expected probability of all five Roll_Values, an observed count of 0 and an observed relative frequency of 0 for each Roll_Value, and a statement that no Roll has been recorded.
6. WHEN the displayed observed counts or observed relative frequencies change, THE Probability_Visualizer SHALL complete the redraw within 16 milliseconds.
7. WHEN a new match starts, THE Probability_Visualizer SHALL reset the observed count of every Roll_Value and the total number of Rolls to 0 before the first `roll` event of that match is displayed.
8. IF a received `roll` event carries a Roll_Value outside the range 0 to 4 inclusive, THEN THE Probability_Visualizer SHALL leave every displayed observed count, observed relative frequency and Roll total unchanged, SHALL display an indication that one Roll was not counted, and SHALL continue to accept further events of the Event_Stream.
9. THE Probability_Visualizer SHALL render every numeric label and Roll_Value label at a text height of at least 16 pixels on a viewport of 1280 by 720 pixels, so that the five Roll_Values and their values remain readable at classroom projection distance.

### Requirement 13: Prediction mode

**User Story:** As a teacher, I want students to commit to a guess before each throw, so that the lesson tests their intuition instead of only showing them a chart.

#### Acceptance Criteria

1. WHERE Prediction_Mode is enabled, THE KUR interface SHALL offer all five Roll_Values from 0 to 4 inclusive as selectable and SHALL require the Human_Player to select exactly one predicted Roll_Value before each Roll of the Human_Player, including each additional Roll granted by a Rosette_Square within the same turn, and SHALL require no predicted Roll_Value before a Roll of The_Dead.
2. WHERE Prediction_Mode is enabled, WHEN a Roll of the Human_Player for which a predicted Roll_Value was selected resolves, THE KUR interface SHALL display within 100 milliseconds the predicted Roll_Value, the actual Roll_Value, the probability of the predicted Roll_Value as reported by the Dice_Model and whether the predicted Roll_Value equals the actual Roll_Value, and SHALL keep that display visible until the next predicted Roll_Value is selected or the match ends.
3. THE KUR interface SHALL count one total prediction for each resolved Roll of the Human_Player for which a predicted Roll_Value was selected, SHALL count one correct prediction for each such Roll whose actual Roll_Value equals the predicted Roll_Value, and SHALL expose both counts for the current match to the Match_Summary.
4. WHERE Prediction_Mode is disabled, THE KUR interface SHALL allow the Human_Player to request a Roll without selecting a predicted Roll_Value and SHALL add no total prediction for that Roll.
5. WHEN the Human_Player enables or disables Prediction_Mode, THE KUR interface SHALL apply the change from the next Roll of the Human_Player for which no predicted Roll_Value has yet been selected, including an additional Roll granted by a Rosette_Square, and SHALL retain the counts of correct predictions and total predictions already recorded in the current match.
6. IF the Human_Player requests a Roll while Prediction_Mode is enabled and no predicted Roll_Value is selected for that Roll, THEN THE KUR interface SHALL produce no Roll_Value, SHALL leave Match_State unchanged and SHALL display a message stating that a predicted Roll_Value must be selected first.
7. WHERE Prediction_Mode is enabled, WHEN a Roll of the Human_Player starts, THE KUR interface SHALL reject every change to the predicted Roll_Value of that Roll and SHALL keep the selected predicted Roll_Value unchanged until that Roll resolves.
8. WHEN a new match starts, THE KUR interface SHALL set the count of correct predictions and the count of total predictions to 0.

### Requirement 14: Match summary

**User Story:** As a teacher closing the lesson, I want a summary that compares what the class observed with what the model predicts, so that the match ends in a discussion point.

#### Acceptance Criteria

1. WHEN a `match_end` event is emitted, THE Match_Summary SHALL display within 1000 milliseconds, for the five Roll_Values in ascending order from 0 to 4, the observed count as a non-negative integer, the observed relative frequency as a percentage to one decimal place, the expected probability reported by the Dice_Model as a percentage to one decimal place, and the signed difference between the observed relative frequency and the expected probability in percentage points to one decimal place.
2. WHEN a `match_end` event is emitted, THE Match_Summary SHALL display the total number of Rolls of the match, the number of `capture` events of the match, and the winning Player named by the `match_end` event, and SHALL display the value 0 for a count whose underlying events did not occur.
3. WHERE Prediction_Mode was enabled during the match and the count of total predictions is 1 or greater, THE Match_Summary SHALL display the count of correct predictions, the count of total predictions, and the accuracy as the count of correct predictions divided by the count of total predictions expressed as a percentage to one decimal place.
4. IF Prediction_Mode was enabled during the match and the count of total predictions is 0, THEN THE Match_Summary SHALL display both prediction counts as 0 and a statement that no prediction was recorded, in place of an accuracy value.
5. WHEN a `match_end` event is emitted, THE Match_Summary SHALL display the total number of Rolls of the match labelled as the sample size, together with a statement that one match is a single small sample in which deviation of the observed relative frequencies from the expected probabilities is normal.
6. THE Match_Summary SHALL present the comparison between observed relative frequencies and expected probabilities as an observation of one match only, and SHALL state no claim that the observed deviation is or is not statistically significant.
7. WHEN the control that starts a new match is activated, THE KUR interface SHALL start a new match and SHALL reset the observed count of every Roll_Value, the number of captures, the count of correct predictions and the count of total predictions to 0.

### Requirement 15: Board rendering and art direction

**User Story:** As a student at the back of a classroom, I want a board that reads clearly on a projector and looks like the artefact it comes from, so that the application feels finished and legible.

#### Acceptance Criteria

1. THE Board_Renderer SHALL draw the 20 squares of the Board, every Piece on the Board, the count of Pieces in each Hand and the count of Retired Pieces of each Player, deriving every drawn value from the Event_Stream alone.
2. WHILE the Human_Player has a pending Roll_Value, THE Board_Renderer SHALL draw every square that is a legal destination with a treatment whose contrast ratio against the treatment of a non-destination square is at least 3 to 1, and SHALL draw no destination treatment on any square while the set of legal Moves of the Human_Player is empty.
3. WHERE the operating system reports no reduced-motion preference and the reduced-motion control of the KUR interface is disabled, WHEN a `move` event is received, THE Board_Renderer SHALL animate the moved Piece along the traversed Path squares over a duration of between 200 and 600 milliseconds at a rate of at least 30 frames per second, and under either reduced-motion condition THE Board_Renderer SHALL apply the immediate position change required by Requirement 16 instead of that animation.
4. THE Board_Renderer SHALL render the Board, the Probability_Visualizer and the turn controls simultaneously, without scrolling and without overlap between them, on a viewport of 1280 by 720 pixels, with each square of the Board at least 56 by 56 pixels and each Piece at least 32 pixels across.
5. THE KUR interface SHALL draw the Board, the Probability_Visualizer, the Match_Summary and the Provenance_Panel from one declared visual system of at most 6 colours and at most 2 typefaces derived from clay, cuneiform forms and lamplight, and SHALL display no decorative imagery outside that declared visual system.
6. THE KUR interface SHALL render all text at a contrast ratio of at least 4.5 to 1 against the background and at a height of at least 16 pixels on a viewport of 1280 by 720 pixels, and SHALL render the current Roll_Value and the Player to move at a height of at least 32 pixels on that viewport.
7. WHEN an event of the Event_Stream is received, THE Board_Renderer SHALL render that event in Event_Stream order, starting the rendering of an event only after the rendering of the preceding event has completed, and SHALL show the Board state resulting from that event within 700 milliseconds of receiving it.
8. WHEN a `capture` event is received, THE Board_Renderer SHALL show the captured Piece leaving its square and the count of Pieces in the Hand of the owning Player increasing, within 600 milliseconds of receiving the event.
9. IF the Board_Renderer receives an event that it cannot render, THEN THE Board_Renderer SHALL retain the last successfully rendered frame, SHALL display an indication that the drawn Board is not current, and SHALL continue to accept further events of the Event_Stream.

### Requirement 16: Controls and accessibility

**User Story:** As a teacher using KUR in a classroom, I want to silence the audio and reduce motion, so that the application suits the room I am in and the students I have.

#### Acceptance Criteria

1. THE KUR interface SHALL provide one control that mutes and unmutes the Audio_Engine, SHALL make that control operable by keyboard alone, SHALL indicate the current muted or unmuted state by text or shape and not by colour alone, and SHALL retain the selected state across matches for the remainder of the browser session.
2. WHILE the Audio_Engine is muted, THE KUR interface SHALL sound no tone, SHALL continue to deliver every event of the Event_Stream to the Board_Renderer and the Probability_Visualizer, and SHALL leave Match_State and the Event_Stream identical to the state and stream produced when the Audio_Engine is unmuted.
3. WHERE the operating system reports a reduced-motion preference, WHEN a `move` event is received, THE Board_Renderer SHALL place the moved Piece on its destination square within one frame of 16 milliseconds instead of animating it along the traversed Path squares, and SHALL render no flashing and no looping motion.
4. THE KUR interface SHALL allow a complete match, from the first Roll to the Match_Summary, to be played with keyboard input alone, SHALL make every interactive control reachable by sequential keyboard navigation, SHALL trap keyboard focus in no control, and SHALL draw the focus indicator at a contrast ratio of at least 3 to 1 against its background.
5. THE KUR interface SHALL distinguish the Pieces of the two Players by shape in addition to colour, such that the owning Player of every Piece on the Board, in each Hand and in each Retired set is identifiable when the interface is rendered in greyscale.
6. WHEN an event of the Event_Stream is emitted, THE KUR interface SHALL expose to assistive technology, within 500 milliseconds, a text description that names the event kind, the Player to move, the current Roll_Value and the count of Retired Pieces of each Player.
7. WHEN the control that mutes the Audio_Engine is activated while one or more tones are sounding, THE Audio_Engine SHALL stop every sounding tone within 100 milliseconds.
8. WHERE the reduced-motion control of the KUR interface is enabled, THE Board_Renderer SHALL replace Piece animation with a position change completed within one frame of 16 milliseconds for the remainder of the browser session, irrespective of the preference reported by the operating system.
9. THE KUR interface SHALL convey the distinction between highlighted legal destination squares and other squares, and the distinction between observed frequencies and expected probabilities in the Probability_Visualizer, by shape, pattern or text label in addition to colour, such that each distinction remains identifiable when the interface is rendered in greyscale.

### Requirement 17: Deployment and judge access

**User Story:** As a hackathon judge, I want to open one public URL and play KUR immediately, so that I can evaluate the submission without setup or credentials.

#### Acceptance Criteria

1. THE KUR application SHALL be served as static build artefacts produced by the Vite production build, published at exactly one public URL that requires no payment, no subscription and no access credentials from the visitor.
2. THE KUR application SHALL allow a visitor to play a match from the first Roll to the `match_end` event without creating an account, without signing in, without any payment, and without installing any plug-in, extension or additional application.
3. THE KUR application SHALL run with no server-side component, SHALL hold no state on a server, and SHALL require no network request other than the retrieval of its own static assets in order to play a complete match.
4. WHEN the published URL is opened in a desktop browser with an empty cache over a connection of 10 megabit per second, THE KUR application SHALL present, within 5 seconds of the request, an interface that accepts the first Roll with all settings at their default values and with no configuration step required of the visitor.
5. WHILE the Judging_Period is in progress, THE KUR application SHALL remain reachable at the published URL, and that URL SHALL not change, SHALL not require a password and SHALL not expire before the end of the Judging_Period on 29 January 2027.
6. THE KUR application SHALL run a complete match in the current stable release and in the immediately preceding stable release of Chrome, Firefox, Safari and Edge on desktop, at a viewport of 1280 by 720 pixels.
7. THE KUR application SHALL keep the total transferred size of the assets retrieved on a first load at or below 5 megabytes.
8. IF the network connection is lost after the initial load has completed, THEN THE KUR application SHALL allow the match in progress to be played to the `match_end` event without any further network request.
9. IF the browser does not provide Canvas 2D, THEN THE KUR application SHALL display a message that names the missing capability and the supported desktop browsers, instead of an empty or partially drawn interface.

### Requirement 18: Provenance and attribution

**User Story:** As a teacher using KUR as teaching material, I want to see where the rules and the music come from, so that I can tell students what is evidence and what is reconstruction.

#### Acceptance Criteria

1. THE Provenance_Panel SHALL state that the rules of play follow the Finkel_Ruleset reconstructed from tablet BM 33333B and SHALL cite that reconstruction by author, title and year of publication.
2. THE Provenance_Panel SHALL state that the musical material derives from the interval notation of Hurrian Hymn no. 6 from Ugarit and SHALL cite the transcription used by author, title and year of publication, naming the same transcription that the bundled Hymn_Notation document cites under Requirement 9.
3. THE Provenance_Panel SHALL state that every tone heard in KUR is synthesised by the Audio_Engine at run time and that KUR contains no recorded performance of Hurrian Hymn no. 6 and no other recorded audio.
4. THE Provenance_Panel SHALL label each of the following five items with exactly one of the two labels "attested by the artefact" and "supplied by modern reconstruction": the rules of play under the Finkel_Ruleset, the Interval_Sequence of Hurrian Hymn no. 6, the Tonic_Frequency and the absolute frequencies derived from it by the Tuning_System, the assignment of intervals to the event kinds of the Event_Stream, and the visual language of the Board_Renderer.
5. THE KUR interface SHALL display a control that opens the Provenance_Panel on the main interface before a match starts, during a match and after a `match_end` event.
6. THE Provenance_Panel SHALL state, for each source cited under criteria 1 and 2, whether KUR redistributes material from that source or cites it only, and SHALL state the licence under which KUR distributes the audio that the Audio_Engine generates.
7. WHEN the control that opens the Provenance_Panel is activated, THE KUR interface SHALL display the complete content required by criteria 1 to 4 and 6 without any further interaction and SHALL leave Match_State unchanged.
8. IF a citation required by criterion 1, criterion 2 or criterion 6 is absent from the provenance data bundled with KUR, THEN THE Provenance_Panel SHALL display the affected item together with a statement that its source is not recorded and SHALL display the remaining items required by criteria 1 to 4 and 6.

### Requirement 19: Error handling and resilience

**User Story:** As a student on a locked-down school machine, I want KUR to keep working when audio is blocked or something fails, so that a browser restriction does not end the lesson.

#### Acceptance Criteria

1. IF the Web Audio API is unavailable in the browser when KUR loads, THEN THE KUR interface SHALL start the match with the Audio_Engine disabled and SHALL display, within 1000 milliseconds of the interface becoming playable, a message stating that audio is unavailable.
2. IF the browser blocks audio playback until a user gesture, THEN THE KUR interface SHALL display a control that enables the Audio_Engine and SHALL keep that control visible until the control is activated or the match ends.
3. IF the Audio_Engine raises an error while sounding an Event_Stream event, THEN THE Audio_Engine SHALL stop the affected tone within 100 milliseconds, SHALL record the error, and SHALL continue to accept further Event_Stream events.
4. IF the Notation_Parser fails to parse the bundled Hymn_Notation document, THEN THE KUR interface SHALL start the match with the Audio_Engine disabled and SHALL display a message naming the offending symbol and its position as returned by the Notation_Parser.
5. IF the Game_Engine returns a rejection for a Move request of the Human_Player, THEN THE KUR interface SHALL leave the displayed Board unchanged, SHALL retain the pending Roll_Value so that another legal Move can be submitted, and SHALL display within 200 milliseconds a message naming the violated rule returned by the Game_Engine.
6. WHILE the Audio_Engine is disabled, THE KUR interface SHALL continue to accept Rolls and Move requests, SHALL update the Probability_Visualizer on every `roll` event, and SHALL make the Match_Summary available when the `match_end` event is emitted.
7. WHEN the control that enables the Audio_Engine is activated, THE Audio_Engine SHALL discard every Event_Stream event received while audio was blocked and SHALL sound the tone of the first event received after activation within 500 milliseconds.
8. IF the Audio_Engine raises 3 consecutive errors within the same match, THEN THE KUR interface SHALL disable the Audio_Engine for the remainder of that match and SHALL display a message stating that audio is unavailable.

### Requirement 20: Match record

**User Story:** As a developer preparing the demo video and the tests, I want a completed match to be expressible as text and reloadable, so that a specific match can be replayed and verified.

#### Acceptance Criteria

1. WHEN a match ends, THE Game_Engine SHALL produce a Match_Record containing the ordered Roll_Values of the match, the ordered Moves of the match, the winning Player, the total Roll count, and, where a fixed seed was supplied to the Dice_Model, that seed.
2. THE Game_Engine SHALL produce a Match_Record for a match of 0 to 2000 Rolls inclusive, and SHALL reject a Match_Record whose Roll count exceeds 2000 with an error that states the Roll count limit.
3. WHEN a Match_Record is submitted for replay, THE Game_Engine SHALL produce one Match_State per recorded Move in the recorded order, each Match_State obtained by applying that Move under the Finkel_Ruleset to the preceding Match_State, and the final Match_State SHALL name the winning Player recorded in the Match_Record.
4. WHEN THE Game_Engine serialises a Match_Record and then parses the serialised result, THE Game_Engine SHALL produce a Match_Record whose ordered Roll_Values, ordered Moves, winning Player, Roll count and seed are each equal to those of the original Match_Record, for every Match_Record of 0 to 2000 Rolls inclusive.
5. IF a Match_Record contains a Roll_Value outside the range 0 to 4 inclusive, or a Move that the Finkel_Ruleset does not allow for the Roll_Value recorded at that position, THEN THE Game_Engine SHALL reject the Match_Record, SHALL return a rejection naming the first offending Move and its position in the recorded order, and SHALL leave the current Match_State unchanged.
6. IF the text submitted as a serialised Match_Record cannot be parsed into a Match_Record, THEN THE Game_Engine SHALL reject that text, SHALL return a rejection naming the position in the text at which parsing failed, and SHALL leave the current Match_State unchanged.
7. WHERE browser local storage is available, THE KUR interface SHALL store the Match_Records of the current browser session up to the 20 most recent Match_Records, discarding the oldest stored Match_Record when a further Match_Record is stored, and SHALL list every stored Match_Record with its winning Player and its Roll count.
8. IF storing a Match_Record in browser local storage fails, THEN THE KUR interface SHALL continue the session with the Match_Records held in memory and SHALL display a message stating that the match was not stored.
