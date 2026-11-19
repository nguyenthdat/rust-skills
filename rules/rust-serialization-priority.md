---
name: rust-serialization-priority
description: "Rust serialization format priority: Cap'n Proto → Protobuf → JSON; no other wire formats without a measured reason."
# TTSR: rust-skills `m11-ecosystem/serialization.md`. Inherits the
# profile-wide `interruptMode: never`.
# Conditions: adding a non-Cap'n-Proto serialization crate to Cargo.toml
# (capnp itself stays quiet), or encoding with one of them in Rust source.
# serde_json *reads* (from_str/from_slice) stay quiet: parsing an existing
# JSON input is not a format choice.
condition:
  - '(?m)^[ \t]*(?:serde_json|simd-json|sonic-rs|prost|protobuf|rmp-serde|rmp|bincode|ciborium|serde_cbor|postcard|flatbuffers|rkyv|serde_yaml|serde_yml|borsh|bitcode|speedy)[ \t]*='
  - '\bserde_json::to_(?:string|string_pretty|vec|vec_pretty|writer|writer_pretty)\b'
  - '\b(?:bincode|rmp_serde|postcard|ciborium|serde_cbor|bitcode|borsh)::(?:serialize|to_\w+|encode\w*)\b'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
  - tool:edit(Cargo.toml)
  - tool:write(Cargo.toml)
---

# Rust serialization: Cap'n Proto → Protobuf → JSON

Pick the first format that satisfies the boundary:

| Priority | Format | Crates | When |
|---|---|---|---|
| 1 | **Cap'n Proto** | `capnp`, `capnpc` (build.rs), `capnp-rpc` | Rust or other controlled peers, IPC, mmap/storage, zero-copy hot paths, RPC |
| 2 | **Protobuf** | `prost` + `prost-build`, `tonic` (gRPC) | gRPC, cross-team or multi-language schemas, cloud-native services |
| 3 | **JSON** | `serde` + `serde_json` | Public or browser HTTP APIs, webhooks, debug dumps, interop of last resort |

- Write the schema first: check in `.capnp`/`.proto` files, generate code in `build.rs`, convert to domain types at the boundary, and only ever add fields (never reuse ordinals or tags).
- Do not add a fourth format (MessagePack, bincode, CBOR, postcard, FlatBuffers, rkyv, YAML…) without a measured reason recorded in the PR. Human-edited config is TOML and does not count.
- Within one process, do not serialize. Pass Rust types directly, or use BoltFFI across languages.

If the format is already fixed by an external contract (a third-party JSON API, an existing `.proto`, a file format you must read or write), or JSON is the right tier for a public endpoint, move on.
