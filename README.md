# almide-bindgen

Universal FFI binding generator for [Almide](https://github.com/almide/almide). Written entirely in Almide.

## 14 Languages

| Language | FFI | Status |
|----------|-----|--------|
| Python | ctypes | verified |
| Go | cgo | verified |
| Ruby | FFI gem | verified |
| Swift | C header | verified |
| C# | P/Invoke | verified |
| Dart | dart:ffi | verified |
| Kotlin | JNA | ready |
| Java | JNA | ready |
| C | header | ready |
| Zig | extern | ready |
| Nim | dynlib | ready |
| Elixir | NIF | ready |
| PHP | FFI ext | ready |
| Lua | LuaJIT FFI | ready |

## How it works

```
mylib.almd (Almide source)
    │
    ├─ almide compile --json         → interface.json
    ├─ almide --target rust --repr-c → source.rs
    │
    ▼
almide run scaffolding.almd          → src/lib.rs (byte buffer FFI)
    │
    ▼
cargo build                          → libalmide_mylib.dylib
    │
    ▼
almide run bindings/python.almd      → almide_mylib.py (pure Python)
almide run bindings/go.almd          → almide_mylib.go (pure Go)
almide run bindings/swift.almd       → almide_mylib.swift (pure Swift)
    ...
```

## Quick start

```bash
# 1. Generate interface + Rust source
almide compile mylib.almd --json > interface.json
almide mylib.almd --target rust --repr-c > source.rs

# 2. Generate FFI scaffolding
almide run scaffolding.almd -- interface.json source.rs
cargo build

# 3. Generate binding for your language
almide run bindings/python.almd -- interface.json

# 4. Use it
python3 -c "from almide_mylib import Point, distance; print(distance(Point(x=0,y=0), Point(x=3,y=4)))"
# → 5.0
```

## Byte buffer protocol

All types are serialized to/from byte buffers. No C struct mapping needed.

| Type | Encoding | Size |
|------|----------|------|
| Int | big-endian i64 | 8 bytes |
| Float | big-endian f64 | 8 bytes |
| Bool | 0 or 1 | 1 byte |
| String | u32 BE length + UTF-8 | 4 + N bytes |
| Record | fields in order | sum of fields |
| Variant | u32 BE tag + payload | 4 + payload |

## No external dependencies

- No UniFFI
- No PyO3 / maturin
- No napi-rs
- No Magnus

Every generator is a `.almd` file. The only build tool needed is `cargo` (to compile the Rust scaffolding into a shared library).

## License

MIT
