# Music Theory in FARD

A music analysis system where every claim it makes is backed by a cryptographic chain of evidence, not a black-box judgment.

## What this is

Most music theory software (Music21, Humdrum, MuseScore plugins) analyzes a piece and hands you an answer: this is a V chord, this is an authentic cadence. You have to trust the tool.

This project is built differently. Every musical object -- a note, an interval, a chord, a scale, a rhythm, a melody, a chord progression, a whole piece -- is constructed through a validator that rejects malformed input, then certified: hashed (SHA-256) into a unique identity that depends on its own content plus the certified identity of everything it was built from. A chord's identity depends on the notes it contains; a melody's identity depends on its notes, its scale, and its rhythm; a piece's identity depends on every melody, chord progression, and rhythm inside it.

That chain is called the tower. It has eight layers, each one built only out of certified objects from the layer below:

1. Note -- pitch, octave, frequency, MIDI number
2. Interval -- the distance and relationship between two notes
3. Chord -- built from notes; quality, voicing, inversion
4. Scale / Mode -- built from notes; Major, Dorian, Phrygian, and others
5. Rhythm -- duration, meter, syncopation (independent of pitch)
6. Melody / Phrase -- built from notes, a scale, and a rhythm; contour, motive windows
7. Harmonic Progression -- built from chords and a scale; Roman numeral analysis, cadences
8. Piece / Movement -- built from melodies, progressions, and rhythms; sections, form

The rule that makes this matter: a higher layer can never invent a fact about a lower layer. A progression cannot claim a chord is 'V' because someone typed that label in -- it has to derive 'V' from the chord's actual root note and quality, the same way every time, or the claim is rejected. This was tested directly: an earlier version of the analysis code classified cadences from a caller-supplied label string instead of the certified chord data, and was rewritten once the gap was found (see Status below).

## What it actually produces

On top of the tower sits a pipeline that turns certified objects into answers a student or professor can read, without ever bypassing the tower to get there:

Tower -> Analysis -> Query -> Corpus -> Compare -> Export -> CLI

- **Analysis** finds motives (repeated melodic cells), cadences, and a style fingerprint (histograms of rhythm, contour, cadence type, harmonic function), all derived from certified data
- **Query** answers specific questions against one piece's analysis (how many authentic cadences, what motives repeat) without re-deriving anything
- **Corpus** holds a set of certified pieces together with their analyses
- **Compare** measures how similar two pieces are -- separately by style, by motive, and by cadence pattern, plus a combined score
- **Export** turns an analysis into a plain-English summary or a structured report
- **CLI** (./bin/mtif) is the door into all of this from a terminal, no code required

## See it work

Seven real pieces are certified in corpus/: Twinkle Twinkle Little Star, the opening of Bach's Minuet in G (BWV Anh. 114, actually composed by Christian Petzold), Amazing Grace, Greensleeves, Ode to Joy, the opening of Bach's Prelude in C Major (BWV 846), and Scarborough Fair.

Ask which piece in the corpus is most similar to Greensleeves:

```sh
$ ./bin/mtif corpus rank greensleeves
Scarborough Fair: 0.747354117798217
Amazing Grace: 0.23566020467361035
Ode to Joy: 0.22561031454106298
Minuet in G (BWV Anh. 114, opening): 0.22189494283350905
Prelude in C Major (BWV 846, opening 8 measures): 0.20464598075816112
Twinkle Twinkle Little Star: 0.17230916487442507
```

Scarborough Fair wins by more than 3x the next closest piece. That is not a coincidence and not a hardcoded rule -- Greensleeves and Scarborough Fair are both in A Dorian and both built on the same i-VII-i chord motion. The system found that on its own by comparing certified harmonic and rhythmic data between the two pieces; nothing in the code says 'modal folk songs are similar.'

Compare two pieces with very little in common:

```sh
$ ./bin/mtif compare twinkle prelude_c_major
twinkle vs prelude_c_major
  shared motives: 0
  cadence similarity: 0.6666666666666666
  style similarity: 0.8904159429330013
  overall similarity: 0.5190275365332226
```

Zero shared motives is itself a finding, not a gap: Twinkle's melody and the Prelude's arpeggio-derived top line genuinely do not share a melodic cell, and the system says so plainly instead of forcing a similarity score.

## How similarity actually works

Motive matching happens two ways, and both are real, tested, separately exposed measures:

- **Exact-pitch matching**: a 3-note window of notes and durations is hashed as-is. Two pieces only match if the identical pitches and durations recur.
- **Transposition-invariant matching**: the same window is hashed as a sequence of semitone intervals plus a duration pattern, with the starting pitch thrown away. Two pieces match if they share a melodic shape, even in different keys.

Proof this distinction is real, not just two names for the same thing: a melody transposed up a whole step scores 1.0 on the transposition-invariant measure and 0.0 on the exact-pitch measure, every time. That comparison is a permanent test (tests/test_transposition_invariant_motives.fard), not a one-off claim.

## Layout

```text
src/core/         canonical JSON/bytes/hashing, shared validation helpers
src/layers/       the 8-layer tower (note, interval, chord, scale, rhythm, melody, progression, piece)
src/tower.fard    full digest tower and exports
src/analysis/     motive, cadence, and style analysis -- reads only certified tower data
src/query/        question-answering over one piece's analysis
src/corpus/       certified collections of pieces, with indices
src/compare/      similarity between two analyses, ranking against a corpus
src/export/       structured reports and plain-English summaries
src/cli/          dispatchers invoked by bin/mtif
corpus/           7 certified real-repertoire pieces
bin/mtif          shell wrapper around fardrun -- the actual command-line tool
tests/            tests for every layer and every analysis claim
tests/diagnostics/  audit probes for the system itself: determinism, validation bypass resistance, labeling honesty
```

## Quickstart

```sh
./bin/mtif analyze bach_minuet_g
./bin/mtif compare greensleeves scarborough_fair
./bin/mtif query repeated-motives bach_minuet_g
./bin/mtif corpus rank greensleeves
```

Available pieces: twinkle, bach_minuet_g, amazing_grace, greensleeves, ode_to_joy, prelude_c_major, scarborough_fair.

Commands: analyze <piece>; compare <piece-a> <piece-b>; query <cadences|motives-in-section|repeated-motives|style-value> <piece> [args]; corpus <list|rank> [piece].

## Status

Every layer of the tower, the full analysis pipeline, and all 7 corpus pieces pass their tests as of this writing. Run the full suite yourself:

```sh
for f in tests/*.fard; do fardrun test --program "$f"; done
```

Two real bugs were found and fixed by testing against actual repertoire rather than synthetic examples:

- A Roman-numeral case bug: a Major-quality chord on a scale degree that is normally minor or diminished (e.g. the VII chord in a Dorian-mode piece) was rendered in lowercase. Found while certifying Greensleeves.
- An octave-mismatch authoring bug: a chord was built on the wrong octave of the correct pitch class, causing its root to be absent from its own scale's certified note list and crashing the analysis with an out-of-bounds error. Found while certifying Scarborough Fair.

Both fixes are permanent regression tests, not just commit messages.

## Run a program and inspect its provenance

```sh
fardrun run --program tests/test_tower.fard --out out/music_theory_tower
```

This writes result.json, digests.json, trace.ndjson, and module_graph.json to the output directory. digests.json records the SHA-256 of every output file plus the runtime version and stdlib root digest, so a run's provenance can be independently re-verified with fardrun verify --out <dir>.

