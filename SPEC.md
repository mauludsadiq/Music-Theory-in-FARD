# Music Theory Tower Specification v1.0.0

This specification defines the canonical object model implemented in `src/`.

## Certification rule

Every layer certificate is computed as:

```text
SHA256(layer_name || lower_digest || schema_digest || rules_digest || inventory_digest)
```

`src/core/canon.fard` implements canonical JSON, canonical bytes, object digest, layer digest, and trace records.

## Layer dependency rule

A constructor may reference lower-layer objects, but validation is local and deterministic. Certification commits to the lower universe digest so downstream objects carry provenance.

## Object identity

Object identity is the SHA-256 digest of canonical JSON `{ layer, payload }`.

## Replay

`tower.trace_tower()` emits one deterministic trace record per layer with input dependency names and the final tower digest.
