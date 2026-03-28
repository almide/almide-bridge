# almide-bridge

Universal FFI binding generator for Almide. Written in Almide.

Purpose-built for Almide's type system. Implemented entirely in Almide.

## How it works

```
Almide source (.almd)
    │
    ├─ almide compile --json → interface.json (types + functions + ABI)
    ├─ almide --target rust --repr-c → source.rs (#[repr(C)] types)
    │
    ▼
almide-bridge reads interface.json + source.rs
    │
    ├─ Generates Rust scaffolding (byte buffer pack/unpack + extern "C" entry points)
    ├─ Generates language bindings (Python/Go/C#/Kotlin/Dart/Ruby/Swift)
    │
    ▼
cargo build → shared library (.so/.dylib/.dll)
    +
Pure language wrapper (no Rust toolchain needed on user side)
```

## Byte buffer protocol

All types are serialized to/from byte buffers. No C struct mapping needed.

```
Rust side:  extern "C" fn bridge_distance(args: *const u8, args_len: i32, out: *mut u8, out_len: *mut i32)
Lang side:  buf = pack([Point(0,0), Point(3,4)])  →  result = unpack_f64(call(buf))
```

### Serialization format

| Type | Encoding |
|------|----------|
| Int | 8 bytes, big-endian i64 |
| Float | 8 bytes, big-endian f64 |
| Bool | 1 byte (0=false, 1=true) |
| String | 4 bytes length (BE i32) + UTF-8 bytes |
| Option[T] | 1 byte tag (0=none, 1=some) + T if some |
| Result[T,E] | 1 byte tag (0=ok, 1=err) + T or E |
| List[T] | 4 bytes count (BE i32) + count × T |
| Record | fields in declaration order |
| Variant | 4 bytes tag (BE i32) + payload |

### Why byte buffers instead of C FFI?

| | C FFI (argtypes, repr(C)) | Byte buffer |
|---|---|---|
| Struct passing | Flatten to scalars or use pointers | Pack/unpack — uniform |
| String handling | char*, manual free | Length-prefixed, automatic |
| Variant dispatch | Split into per-case functions | Tag byte, single function |
| argtypes setup | Per-function, per-language | Not needed |
| New language | Write C FFI mapping from scratch | Implement pack/unpack once |

## Architecture

```
almide-bridge/
├── protocol.almd          ← Serialization spec (pack/unpack)
├── scaffolding.almd       ← Rust scaffolding generator
├── bindings/
│   ├── python.almd        ← Python binding generator
│   ├── go.almd            ← Go binding generator
│   ├── csharp.almd        ← C# binding generator
│   ├── kotlin.almd        ← Kotlin binding generator
│   ├── dart.almd          ← Dart binding generator
│   ├── ruby.almd          ← Ruby binding generator
│   └── swift.almd         ← Swift binding generator
└── bridge.almd            ← CLI entry point
```

All generators are written in Almide. No external tools (no UniFFI, no PyO3, no napi-rs).

## Status

Early development. Design phase.

## License

MIT
