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
tests/test_*.fard            layer-spec and tower-behavior tests
tests/diagnostics/diag_*.fard  meta-property audit probes (determinism, inversion sanity, validation bypass resistance, labeling honesty)
```

## Status

All eight layers of the Musical Theory Tower are implemented.

Layers 6, 7, and 8 have been expanded using a red-test-first discipline:

- Layer 6 (Melody): certified note events with deterministic onsets, contour classification, motive window extraction, interval chains between consecutive notes
- Layer 7 (Progression): Roman numeral derivation from scale-degree position, cadence classification (authentic, plagal, half, deceptive), secondary dominant detection
- Layer 8 (Piece): certified sections, timeline construction, overlap detection, material-reference validation, form fingerprints, deterministic form digests

## Run

From this directory.

Run the test suite (pass/fail per assertion):

```sh
fardrun test --program tests/test_tower.fard
fardrun test --program tests/test_negative.fard
fardrun test --program tests/test_melody.fard
fardrun test --program tests/test_progression.fard
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
