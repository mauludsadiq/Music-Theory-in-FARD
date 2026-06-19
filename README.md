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
src/layers/note.fard         Layer 1
src/layers/interval.fard     Layer 2
src/layers/chord.fard        Layer 3
src/layers/scale.fard        Layer 4
src/layers/rhythm.fard       Layer 5
src/layers/melody.fard       Layer 6
src/layers/progression.fard  Layer 7
src/layers/piece.fard        Layer 8
src/tower.fard               full digest tower and exports
tests/*.fard                 executable layer tests
```

## Run

From this directory:

```sh
fardrun run --program tests/test_tower.fard --out out/music_theory_tower
```

## Design invariant

A higher layer never invents lower-layer truth. It only commits to certified objects already validated beneath it.
