# Music Theory in FARD

A deterministic, cryptographically-verifiable Musical Theory Tower written in FARD.

The project models eight certified layers:

1. Note
2. Interval
3. Chord
4. Scale / Mode
5. Rhythm
6. Melody / Phrase
7. Harmonic Progression
8. Piece / Movement

Every constructed object has:

- a formal schema name and version
- deterministic validation
- canonical JSON and bytes
- SHA-256 identity
- replayable trace records
- dependency digests for lower-layer provenance

The implementation is intentionally data-first. Each layer exposes constructors, validators, canonical encoders, digest functions, certificates, and inventories. No generated object is accepted without validation.

## Layout

```text
src/core/canon.fard          canonical JSON/bytes/hash/cert helpers
src/core/validate.fard       shared result and validation helpers
src/layers/note.fard         Layer 1 -- pitch, frequency, register, transpose
src/layers/interval.fard     Layer 2 -- quality, consonance, inversion
src/layers/chord.fard        Layer 3 -- qualities, voicings, inversions
src/layers/scale.fard        Layer 4 -- modes, traditions, temperament labeling
src/layers/rhythm.fard       Layer 5 -- duration, meter, syncopation
src/layers/melody.fard       Layer 6 -- note events, contour, motive windows, interval chains
src/layers/progression.fard  Layer 7 -- Roman numeral analysis, cadence detection, secondary dominants
src/layers/piece.fard        Layer 8 -- sections, instrumentation, form analysis
src/tower.fard               full digest tower and exports
src/analysis/engine.fard     unified analysis engine -- motive, cadence, and style analysis
src/analysis/motives.fard    motive window extraction, repetition detection, section mapping
src/analysis/cadences.fard   cadence classification from certified chord data
src/analysis/style.fard      rhythm/contour/cadence/harmony histograms, style fingerprinting
src/query/search.fard        query engine -- reads only certified analysis output
src/corpus/corpus.fard       certified collections of pieces
src/corpus/index.fard        deterministic corpus indices (by title, by digest)
src/corpus/query.fard        corpus-level lookup
src/compare/similarity.fard  style/motive/cadence similarity between two analyses
src/compare/corpus_match.fard rank a piece against a corpus
src/export/report.fard       structured analysis report (key, cadence, motives, form)
src/export/json.fard         canonical JSON export payloads
src/export/markdown.fard     student and professor-facing markdown summaries
src/cli/analyze.fard         analyze one piece end to end
src/cli/query.fard           ask questions over analysis and corpus
src/cli/compare.fard         compare two pieces
src/cli/corpus.fard          inspect a corpus
corpus/*.fard                certified real-repertoire pieces (7 works spanning folk, Baroque, hymn, modal, and Classical traditions)
tests/test_*.fard            layer-spec and tower-behavior tests
tests/diagnostics/diag_*.fard  meta-property audit probes (determinism, inversion sanity, validation bypass resistance, labeling honesty)
```

## Status

All eight layers of the Musical Theory Tower are implemented.

Layers 6, 7, and 8 have been expanded using a red-test-first discipline:

- Layer 6 (Melody): certified note events with deterministic onsets, contour classification, motive window extraction, interval chains between consecutive notes
- Layer 7 (Progression): Roman numeral derivation from scale-degree position, cadence classification (authentic, plagal, half, deceptive), secondary dominant detection
- Layer 8 (Piece): certified sections, timeline construction, overlap detection, material-reference validation, form fingerprints, deterministic form digests

Above the tower sits a full analysis and reporting pipeline that consumes certified layer output without inventing new lower-layer truth -- the architecture is Tower -> Analysis -> Query -> Corpus -> Compare -> Export -> CLI:

- Analysis Engine (src/analysis/): motive extraction and repetition detection, cadence classification derived from certified chord/scale data (not caller-supplied labels), and style fingerprinting across rhythm, contour, cadence, and harmonic function
- Query Engine (src/query/search.fard): reads only the certified output of analysis.engine.analyze_piece -- it never re-analyzes the tower directly
- Corpus (src/corpus/): certified collections of pieces with their analyses, deterministic indices by title and by digest, corpus-level lookup
- Compare (src/compare/): style, motive, and cadence similarity between two analyses; ranks a piece against a whole corpus
- Export (src/export/): structured analysis reports, canonical JSON payloads, and student/professor-facing markdown summaries (key, terminal cadence, form, motive breakdown)
- CLI (src/cli/): thin orchestration over the stack -- analyze a piece end to end, query analysis or corpus data, compare two pieces, inspect a corpus

A real-repertoire corpus of 7 works lives under corpus/, spanning folk, Baroque, hymn, modal/Renaissance, and Classical traditions, with binary, ternary, and through-composed forms:

- Twinkle Twinkle Little Star (folk, binary) -- no exact-pitch motivic repetition, since its phrases vary pitch rather than repeating literally
- Minuet in G, BWV Anh. 114, opening (Baroque, binary; commonly attributed to Bach, actually by Christian Petzold) -- a literal G-B-A-G cell recurs across both A and B sections
- Amazing Grace (hymn, binary) -- ends on a plagal cadence, the classic hymn "Amen" quality
- Greensleeves (modal/Renaissance, binary, A Dorian) -- first modal piece in the corpus; exposed and fixed a Roman-numeral casing bug where a Major-quality chord on a normally-minor scale degree was rendered lowercase
- Ode to Joy (Classical, ternary A-B-A) -- first non-binary form in the corpus
- Prelude in C Major, BWV 846, opening (Baroque, through-composed) -- harmonic-density stress test with 8 distinct chords in one section, including a genuine secondary dominant (V7/V) correctly detected
- Scarborough Fair (ballad, binary, A Dorian) -- second modal piece; exposed and fixed an octave-mismatch authoring bug where a chord root didn't match the scale's actual pitch, causing an out-of-bounds crash in degree lookup

Every piece's form, terminal cadence, and motivic repetition have been verified against the actual music. The corpus also demonstrates genuine cross-piece musicological reasoning: ranking Greensleeves against the other six pieces by total similarity (style + motive + cadence) correctly puts Scarborough Fair first by a wide margin (0.75 vs. 0.24 for the next closest), with a perfect 1.0 cadence-similarity score -- both pieces share identical Dorian i-VII-i cadential motion, discovered purely from certified structural analysis with no hardcoded rule about modal similarity.

## Run

From this directory.

Run the test suite (pass/fail per assertion):

```sh
fardrun test --program tests/test_tower.fard
fardrun test --program tests/test_negative.fard
fardrun test --program tests/test_melody.fard
fardrun test --program tests/test_progression.fard
fardrun test --program tests/test_piece.fard
fardrun test --program tests/test_analysis_engine.fard
fardrun test --program tests/test_query_search.fard
fardrun test --program tests/test_useful_surface.fard
fardrun test --program tests/test_corpus_twinkle.fard
fardrun test --program tests/test_corpus_bach_minuet_g.fard
fardrun test --program tests/test_corpus_amazing_grace.fard
fardrun test --program tests/test_corpus_greensleeves.fard
fardrun test --program tests/test_corpus_ode_to_joy.fard
fardrun test --program tests/test_corpus_prelude_c_major.fard
fardrun test --program tests/test_corpus_scarborough_fair.fard
fardrun test --program tests/test_export_markdown.fard
fardrun test --program tests/test_compare_corpus_ranking.fard
```

Run a program and capture its digest, trace, and module graph:

```sh
fardrun run --program tests/test_tower.fard --out out/music_theory_tower
```

This writes result.json, digests.json, trace.ndjson, and module_graph.json to the output directory. digests.json records the SHA-256 of every output file plus the runtime version and stdlib root digest, so a run's provenance can be independently verified with fardrun verify --out <dir>.

## Design invariant

A higher layer never invents lower-layer truth. It only commits to certified objects already validated beneath it.

## Testing discipline

New layer behavior is specified as a failing test suite before it is implemented. tests/test_<layer>.fard defines the contract; src/layers/<layer>.fard is then built incrementally until every test passes. tests/diagnostics/ holds a separate audit suite that probes the system's meta-properties -- determinism across reruns, interval inversion correctness, validation reachability on malformed input, and honesty of tradition/temperament labeling -- rather than layer-specific musical behavior.
