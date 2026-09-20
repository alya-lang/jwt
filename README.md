# jwt

[![CI](https://github.com/alya-lang/jwt/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/jwt/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/jwt?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fjwt%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fjwt%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

Native RFC 7519 JSON Web Token (JWT) library for the [Alya Programming Language](https://github.com/alya-lang/alya). Provides secure HMAC-SHA256 (HS256) signing, constant-time verification, Base64URL encoding/decoding, claim validation, and a fluent builder API.

---

## 🌟 Features

- 🔒 **RFC 7519 Compliant**: Standard HS256 signing and Base64URL encoding backed by `alya-lang/crypto`.
- 🛡️ **Timing Attack Protection**: Constant-time signature verification (`constant_time_eq`) prevents side-channel timing attacks.
- ⏱️ **Full Claim Validation**: Built-in verification for standard claims:
  - `exp` (Expiration Time)
  - `nbf` (Not Before)
  - `iss` (Issuer match)
  - `aud` (Audience match)
  - Clock skew tolerance (`leeway` window in seconds)
- 🏗️ **Fluent Builder API**: Convenient builder pattern to assemble claims and sign tokens without boilerplate.
- ⚡ **High Performance**: Built for speed with sub-microsecond signing, verification, and token decoding
- 🧪 **Fully Tested**: Tested against NIST SHA-256 test vectors, RFC 4231 HMAC test vectors, and standard RFC 7519 scenarios.

---

## 📁 Project Architecture

```
jwt/
├── alya.toml               # Package manifest (depends on alya-lang/crypto)
├── src/
│   ├── lib.alya            # Public API facade
│   ├── types.alya          # JwtToken & JwtVerifyOptions structs
│   ├── builder.alya        # Fluent JwtBuilder API
│   └── core/
│       └── verifier.alya   # Claim verification logic (exp, nbf, iss, aud)
├── examples/
│   └── demo.alya           # Full end-to-end authentication workflow
├── tests/
│   └── test_basic.alya     # 48 test assertions (NIST vectors, tamper tests)
└── benches/
    └── bench_basic.alya    # Micro-benchmarks
```

---

## 📦 Installation

Add `jwt` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
jwt = { git = "https://github.com/alya-lang/jwt", branch = "main" }
```

Or install it directly using the Alya package CLI:

```bash
alya add jwt --git https://github.com/alya-lang/jwt --branch main
alya install
```

---

## 🚀 Quick Start

### 1. Simple Sign & Verify

```alya
import "jwt" as jwt

function main()
    let secret = "your-256-bit-secret"
    let payload = {"sub": "user_123", "role": "admin"}

    # Encode and sign token (HS256)
    let token = jwt::encode(payload, secret)
    say "Token: " + token

    # Verify signature and standard claims
    let res = jwt::verify(token, secret)
    if res.valid == 1
        say "Token is valid for subject: " + res.payload["sub"]
    else
        say "Verification failed: " + res.error
    end
end

main()
```

### 2. Auto Expiration Helper (`jwt::sign`)

```alya
import "jwt" as jwt

function main()
    let secret = "secret"
    let payload = {"sub": "user_42"}

    # Automatically adds 'iat' (now) and 'exp' (now + 3600 seconds)
    let token = jwt::sign(payload, secret, 3600)

    say "Is valid: " + str(jwt::is_valid(token, secret))
    say "Is expired: " + str(jwt::is_expired(token))
    say "Subject: " + jwt::get_claim(token, "sub")
end

main()
```

### 3. Fluent Builder API

```alya
import "jwt" as jwt

function main()
    let secret = "secret"

    let b = jwt::builder()
    jwt::builder_subject(b, "user_42")
    jwt::builder_issuer(b, "auth.myapi.com")
    jwt::builder_audience(b, "api.myapi.com")
    jwt::builder_expires_in(b, 7200) # Valid for 2 hours
    jwt::builder_jwt_id(b, "jti-987654")
    jwt::builder_claim_str(b, "role", "maintainer")
    jwt::builder_claim_int(b, "tier", 3)
    jwt::builder_claim_bool(b, "active", 1)

    let token = jwt::builder_sign(b, secret)
    say "Signed Token:\n" + token
end

main()
```

### 4. Verification with Options & Leeway

```alya
import "jwt" as jwt

function main()
    let secret = "secret"

    # Configure validation rules:
    # leeway = 60s, expected issuer, expected audience, ignore_exp = 0
    let opts = jwt::options(60, "auth.myapi.com", "api.myapi.com")

    let verified = jwt::verify_with_options(token, secret, opts)
    if verified.valid == 1
        say "Token verified successfully!"
    else
        say "Rejection reason: " + verified.error
    end
end

main()
```

---

## 📖 API Reference

### Enums & Types

| Symbol | Type | Description |
|---|---|---|
| `JwtAlgorithm` | `enum` | Supported signing algorithms (`HS256`, `HS384`, `HS512`). |
| `JwtErrorCode` | `enum` | Standard verification error codes (`NONE`, `MALFORMED_TOKEN`, `INVALID_SIGNATURE`, etc.). |
| `JwtToken` | `struct` | Decoded token struct containing `raw`, `header`, `payload`, `signature`, `valid`, and `error`. |
| `JwtVerifyOptions` | `struct` | Options model with builder methods (`with_leeway`, `with_issuer`, `with_audience`). |
| `JwtBuilder` | `struct` | Fluent builder struct for constructing claims and signing tokens. |

### Core Functions

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `encode(payload, secret)` | `payload: Map, secret: string` | `string` | Signs payload map using HS256 and returns a compact JWT string. |
| `encode_raw(json_str, secret)` | `json_str: string, secret: string` | `string` | Signs raw JSON payload string using HS256. |
| `decode(token)` | `token: string` | `JwtToken` | Decodes header and payload without verifying signature. |
| `verify(token, secret)` | `token: string, secret: string` | `JwtToken` | Verifies HS256 signature and checks standard claims with default options. |
| `verify_with_options(token, secret, opts)` | `token: string, secret: string, opts: JwtVerifyOptions` | `JwtToken` | Verifies signature and validates claims with custom leeway, issuer, audience. |
| `sign(payload, secret, expires_in_sec)` | `payload: Map, secret: string, expires_in_sec = 3600` | `string` | Signs payload with auto-populated `iat` and `exp` claims. |
| `is_valid(token, secret)` | `token: string, secret: string` | `int (0/1)` | Returns 1 if token signature and claims are valid, 0 otherwise. |
| `is_expired(token)` | `token: string` | `int (0/1)` | Returns 1 if token's `exp` claim is in the past, 0 otherwise. |
| `get_claim(token, claim_name)` | `token: string, claim_name: string` | `any` | Extracts a claim value from payload without verifying signature. |
| `options(leeway, iss, aud, ignore_exp)` | `leeway = 0, iss = "", aud = "", ignore_exp = 0` | `JwtVerifyOptions` | Creates verification configuration. |

### Builder Functions

| Function | Description |
|---|---|
| `builder()` | Creates a new `JwtBuilder` instance. |
| `builder_subject(b, sub)` | Sets the `sub` claim. |
| `builder_issuer(b, iss)` | Sets the `iss` claim. |
| `builder_audience(b, aud)` | Sets the `aud` claim. |
| `builder_expires_in(b, sec)` | Sets `iat` to now and `exp` to now + `sec`. |
| `builder_expires_at(b, timestamp)` | Sets explicit `exp` timestamp. |
| `builder_issued_at(b, timestamp)` | Sets explicit `iat` timestamp. |
| `builder_not_before(b, timestamp)` | Sets `nbf` timestamp. |
| `builder_jwt_id(b, jti)` | Sets `jti` claim. |
| `builder_claim_str(b, key, val)` | Adds custom string claim. |
| `builder_claim_int(b, key, val)` | Adds custom integer claim. |
| `builder_claim_bool(b, key, val)` | Adds custom boolean claim. |
| `builder_sign(b, secret)` | Signs assembled claims into a compact JWT token. |

### Cryptographic Primitives

| Function | Description |
|---|---|
| `sha256(msg)` | Computes SHA-256 hex digest of string message. |
| `hmac_sha256(secret, msg)` | Computes HMAC-SHA256 hex digest using secret key. |
| `b64url_encode(s)` | RFC 7515 Base64URL string encoding without padding. |
| `b64url_decode(s)` | RFC 7515 Base64URL decoding. |

---

## 🧪 Running Tests & Benchmarks

Run the automated test suite using `alya test`:

```bash
alya test
```

Generate static API documentation:

```bash
alya doc . -o docs --markdown
```

Run the benchmark suite:

```bash
alya run benches/bench_basic.alya
```

Run the example demo:

```bash
alya run examples/demo.alya
```

Check code formatting:

```bash
alya fmt . --check
```

Run static code linter:

```bash
alya lint . --check
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install dependencies:
   ```bash
   alya install
   ```
3. Create your feature branch (`git checkout -b feature/my-feature`)
4. Verify tests and formatting before opening a PR:
   ```bash
   alya test
   alya fmt . --check
   ```
5. Commit your changes (`git commit -m "feat: add feature"`) and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.