# Serialization Format Priority

> Project rule: **Cap'n Proto → Protobuf → JSON.** Pick the first one that satisfies the constraints.
> Applies to wire protocols, IPC, persisted data, and payloads crossing FFI as bytes.
> (Does not apply to human-edited config: use TOML for that.)

## Decision

| Priority | Format | Crates | Choose when | Skip to next when |
|----------|--------|--------|-------------|-------------------|
| 1 | **Cap'n Proto** | `capnp`, `capnpc` (build.rs codegen), `capnp-rpc` | Rust↔Rust or controlled peers; IPC / shared memory / mmap'd files; hot paths where zero-copy reads matter; RPC with promise pipelining | A peer has no good Cap'n Proto support (browsers, some mobile/3rd-party stacks), or you must speak gRPC |
| 2 | **Protobuf** | `prost` + `prost-build` (messages), `tonic` (gRPC), `pbjson` (proto3 JSON mapping) | gRPC / microservices; cross-team, multi-language schemas; Kubernetes/cloud-native ecosystems; grpc-web | Consumer is a public/browser/3rd-party HTTP API, or humans need to read/write the payload |
| 3 | **JSON** | `serde` + `serde_json` (`sonic-rs` / `simd-json` if measured hot) | Public REST APIs, browser clients, webhooks, debugging dumps, interoperability of last resort | — |

Don't introduce a 4th format (MessagePack, bincode, CBOR, FlatBuffers, rkyv, …) unless a measured
requirement rules out all three — and document why in the PR.

## Rules

1. **Schema first.** `.capnp` / `.proto` files are the source of truth; check them in, generate code in `build.rs`
   (`capnpc::CompilerCommand`, `prost_build::compile_protos`). Never hand-edit generated code.
2. **Keep generated types at the edge.** Convert into domain types (`TryFrom<proto::Order> for Order`) at the
   boundary; don't let generated structs leak into business logic (see m09-domain).
3. **Evolution:** only add fields; never renumber / reuse field ordinals or tags; mark removed proto fields `reserved`.
   Cap'n Proto: new fields go at the end with the next `@N`.
4. **One format per boundary.** Don't dual-encode the same API unless serving two consumer classes (e.g. gRPC
   internal + JSON public via `pbjson` or a gateway).
5. **JSON is `serde`-derived**, never hand-built strings. Use `#[serde(deny_unknown_fields)]` on inputs you control,
   `#[serde(rename_all = "camelCase")]` for JS consumers.
6. Inside a single process, don't serialize at all — pass Rust types. For Rust → other language in-process, use
   BoltFFI (see `ffi-boltffi`).

## Snippets

```toml
# Cap'n Proto
[dependencies]
capnp = "0.28"
[build-dependencies]
capnpc = "0.28"
```

```rust
// build.rs
fn main() {
    capnpc::CompilerCommand::new()
        .src_prefix("schema")
        .file("schema/order.capnp")
        .run()
        .expect("capnp schema compile failed");
}
```

```toml
# Protobuf / gRPC
[dependencies]
prost = "0.14"
tonic = "0.14"
[build-dependencies]
tonic-prost-build = "0.14"   # or prost-build for messages only
```

> Versions above are indicative — confirm the latest with the `crate-researcher` agent before writing Cargo.toml.
