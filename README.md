# almide-bindgen

Universal FFI binding generator for [Almide](https://github.com/almide/almide). Written entirely in Almide.

This is a **library** — import it with `import bindgen`. The CLI tool is [almide-lander](https://github.com/almide/almide-lander).

## 20 Languages

| Language | FFI | Language | FFI |
|----------|-----|----------|-----|
| Python | ctypes | C++ | extern "C" |
| Go | cgo | Rust | FFI |
| Ruby | FFI gem | JavaScript | koffi |
| Swift | C header | Scala | JNA |
| C# | P/Invoke | Julia | ccall |
| Dart | dart:ffi | PowerShell | DllImport |
| Kotlin | JNA | Elixir | NIF |
| Java | JNA | PHP | FFI ext |
| C | header | Lua | LuaJIT FFI |
| Zig | extern | Nim | dynlib |

## Library API

```almide
import bindgen

// Version + supported languages
bindgen.version()                    // "0.1.0"
bindgen.supported_languages()        // ["python", "go", ..., "powershell"]

// Generate Rust FFI scaffolding
let lib_rs = bindgen.scaffolding.generate(iface_json, rust_source)
let cargo  = bindgen.scaffolding.generate_cargo(iface_json)

// Generate language bindings (all return String)
let py_code = bindgen.bindings.python.generate(iface_json)
let go_code = bindgen.bindings.go.generate(iface_json)
// ... 20 languages
```

## Byte buffer protocol

All types are serialized to/from byte buffers across the FFI boundary. No C struct mapping needed.

```
Rust:   extern "C" fn bridge_distance(args: *const u8, args_len: i32, out: *mut u8, out_cap: i32) -> i32
Python: buf = struct.pack('>d', a.x) + ... → result = struct.unpack_from('>d', call(buf))
```

| Type | Encoding | Size |
|------|----------|------|
| Int | big-endian i64 | 8 bytes |
| Float | big-endian f64 | 8 bytes |
| Bool | 0 or 1 | 1 byte |
| String | u32 BE length + UTF-8 | 4 + N bytes |
| Record | fields in order | sum of fields |
| Variant | u32 BE tag + payload | 4 + payload |

## No external dependencies

Every generator is a `.almd` file. No UniFFI, no PyO3, no napi-rs.

## License

MIT
