# jwt

[![CI](https://github.com/alya-lang/jwt/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/jwt/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/jwt?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fjwt%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fjwt%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

Zero-dependency JSON Web Token (JWT) library for Alya

---

## 🌟 Features

- ⚡ **Lightweight & Fast**: Built for speed with minimal overhead
- 📦 **Zero Dependencies**: Pure Alya code, entirely self-contained
- 🧩 **Modular Architecture**: Multi-module design supporting flat modules (`types.alya`) and subfolder hierarchies (`core/formatter.alya`)
- 🛡️ **Reliable & Typed**: Explicit struct definitions and clean namespaced APIs
- 🧪 **Well Tested**: Comprehensive test suite with standard assertions

---

## 📁 Project Architecture

```
jwt/
├── alya.toml               # Package manifest
├── src/
│   ├── lib.alya            # Public API facade
│   ├── types.alya          # Data structures & struct definitions
│   └── core/               # Subdirectory module hierarchy (optional for larger packages)
│       └── formatter.alya  # Domain formatting logic & internal helpers
├── examples/
│   └── demo.alya           # Runnable usage examples
├── tests/
│   └── test_basic.alya     # Automated test suite
└── benches/
    └── bench_basic.alya    # Micro-benchmarks
```

> [!NOTE]
> Modules can be structured flat inside `src/` (e.g. `src/types.alya`) or grouped into subdirectories (e.g. `src/core/formatter.alya`). Relative imports like `import "../types.alya"` or `import "./core/formatter.alya"` are resolved relative to the importing file and deduplicated transitively.

---

## 📦 Installation

Add `jwt` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
jwt = { git = "https://github.com/alya-lang/jwt", tag = "v0.1.0" }
```

Or install it directly using the Alya package CLI:

```bash
alyac add jwt --git https://github.com/alya-lang/jwt --tag v0.1.0
alyac install
```

---

## 🚀 Quick Start

```alya
import "jwt" as pkg

function main()
    # Basic facade call
    let greeting = pkg::hello("Alya")
    say greeting

    # Struct construction and domain helpers
    let cfg = pkg::new_config("Community", 2)
    say "Target: " + cfg.name
    say "Formatted: " + pkg::core_format_custom(cfg)
end

main()
```

---

## 📖 API Reference

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `hello(name)` | `name = "World"` | `string` | Returns a friendly greeting message. |
| `new_config(name, count)` | `name = "World", count = 1` | `JwtConfig` | Constructs a new configuration struct. |
| `core_format_greeting(name)` | `name` | `string` | Core formatter producing `Hello, {name}!`. |
| `core_format_custom(config)` | `config: JwtConfig` | `string` | Formats greeting using prefix and name from config. |

---

## 🧪 Running Tests & Benchmarks

Run the test suite using `alyac`:

```bash
alyac run tests/test_basic.alya
```

Run the benchmark suite:

```bash
alyac run benches/bench_basic.alya
```

Run the example demo:

```bash
alyac run examples/demo.alya
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps to contribute:

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/my-new-feature`)
3. Commit your changes (`git commit -m "feat: add some feature"`)
4. Push to the branch (`git push origin feature/my-new-feature`)
5. Open a Pull Request

Please make sure tests pass before submitting a PR.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.