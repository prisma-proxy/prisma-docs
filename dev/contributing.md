---
title: Contributing Guide
---

# Contributing Guide

How to extend the Prisma proxy system: adding transports, CLI commands, API endpoints, config fields, testing, fuzzing, and benchmarking.

---

## Adding a New Transport

1. **Client stream adapter:** `crates/prisma-client/src/my_transport_stream.rs` -- implement `AsyncRead + AsyncWrite + Send + Unpin`
2. **Register in transport selector:** `crates/prisma-client/src/transport_selector.rs` -- add match arm
3. **Add module:** `crates/prisma-client/src/lib.rs`
4. **Server listener:** `crates/prisma-server/src/listener/my_transport.rs` -- implement `listen()` function
5. **Register listener:** `crates/prisma-server/src/listener/mod.rs` and `crates/prisma-server/src/lib.rs`
6. **Add config fields:** `crates/prisma-core/src/config/server.rs` and `client.rs`
7. **Update version command:** `crates/prisma-cli/src/main.rs`

---

## Adding a New CLI Command

1. Add command variant to `Commands` enum in `crates/prisma-cli/src/main.rs`
2. Create handler module in `crates/prisma-cli/src/`
3. Register module and add match arm in `main()`

---

## Adding a New Management API Endpoint

1. Create handler in `crates/prisma-mgmt/src/handlers/`
2. Register route in `crates/prisma-mgmt/src/router.rs`
3. Optionally add corresponding CLI command

---

## Adding a New Config Field

1. Add field with `#[serde(default)]` to config struct
2. Add default function
3. Add validation in `config/validation.rs`
4. Use in the appropriate crate
5. Add to reload handler if hot-reloadable

---

## Testing

```bash
cargo test --workspace              # All tests
cargo test -p prisma-core           # Specific crate
cargo test -- --nocapture           # With output
```

Conventions: `#[tokio::test]` for async, `proptest` for codec/protocol, mock `AuthVerifier` for handshake tests.

---

## Fuzzing

```bash
cargo install cargo-fuzz
cargo fuzz run fuzz_decode_client_init
```

Key targets: `fuzz_decode_client_init`, `fuzz_decode_server_init`, `fuzz_decode_data_frame`, `fuzz_mux_frame`.

---

## Benchmarks

```bash
cargo bench --workspace
cargo bench -p prisma-core -- handshake
```

Key benchmarks: `bench_aead_encrypt`, `bench_handshake`, `bench_handshake_pq`, `bench_relay_throughput`, `bench_router_evaluate`.

---

## Code Style

- `cargo fmt --all` before committing
- `cargo clippy --workspace --all-targets` -- fix all warnings
- Use `tracing` for logging (not `println!`)
- Use `anyhow::Result` for fallible functions
- FFI functions must use `ffi_catch!` macro

---

## Project Structure

```
prisma/                          # Core monorepo
├── crates/
│   ├── prisma-core/             # Shared library (crypto, protocol, config)
│   ├── prisma-server/           # Server binary (listeners, relay, auth)
│   ├── prisma-client/           # Client library (SOCKS5, TUN, pool)
│   ├── prisma-cli/              # CLI binary (clap 4)
│   ├── prisma-mgmt/             # Management API (axum)
│   └── prisma-ffi/              # C FFI for GUI/mobile
├── scripts/                     # Install scripts (bash, powershell)
├── fuzz/                        # Fuzz test targets
└── tools/prisma-mcp/            # MCP development server

prisma-gui/                      # Desktop GUI (separate repo)
├── src/                         # React 19 frontend
├── src-tauri/                   # Tauri 2 Rust backend
└── prisma/                      # Git submodule → core monorepo

prisma-console/                  # Web dashboard (separate repo)
├── src/app/                     # Next.js 15 App Router pages
└── src/components/              # Dashboard widgets

prisma-docs/                     # Documentation (separate repo)
├── docs/                        # Reference documentation
├── guide/                       # Beginner's guide
└── dev/                         # Developer documentation
```

---

## Development Workflow

1. **Branch**: Create a feature branch from `master`
2. **Implement**: Make changes, run `cargo fmt` and `cargo clippy`
3. **Test**: Run `cargo test --workspace` and verify no regressions
4. **Commit**: Use conventional commits (`feat:`, `fix:`, `perf:`, `chore:`)
5. **Push**: Push to your fork and open a PR against `master`

CI runs automatically on every push: format check, clippy, tests (Linux/macOS/Windows/ARM).

---

## Running Locally

```bash
# Terminal 1: Run the server
cargo run -p prisma-cli -- server -c examples/server.toml

# Terminal 2: Run the client
cargo run -p prisma-cli -- client -c examples/client.toml

# Terminal 3: Test through the proxy
curl -x socks5h://127.0.0.1:1080 https://httpbin.org/ip

# Run the web console (connects to management API)
cargo run -p prisma-cli -- console --port 3000
```

For the GUI:
```bash
cd prisma-gui
git submodule update --init --recursive
npm install
npm run tauri dev
```

---

## Adding an FFI Function

1. **Declare** in `crates/prisma-ffi/src/lib.rs`:
   ```rust
   #[no_mangle]
   pub unsafe extern "C" fn prisma_my_function(
       handle: *mut PrismaClient,
       arg: *const c_char,
   ) -> c_int {
       ffi_catch!(handle, {
           let arg_str = CStr::from_ptr(arg).to_str()?;
           // implementation...
           Ok(PRISMA_OK as c_int)
       })
   }
   ```
2. **Add header** in `crates/prisma-ffi/include/prisma_ffi.h`
3. **Add Tauri command** in GUI's `src-tauri/src/commands.rs` if needed
4. **Add frontend wrapper** in GUI's `src/lib/commands.ts`

---

## Debugging Tips

```bash
# Enable detailed tracing output
RUST_LOG=prisma_client=debug,prisma_core=debug cargo run -p prisma-cli -- client -c client.toml

# Run a single test with output
cargo test -p prisma-core -- --nocapture test_name

# Check for unsafe code issues
cargo clippy --workspace -- -W clippy::undocumented_unsafe_blocks

# Profile with flamegraph
cargo install flamegraph
cargo flamegraph -p prisma-cli -- server -c server.toml
```

---

## Release Process

1. Ensure all CI checks pass on `master`
2. Update version in all files (use the version-sync agent or manually update 8 files)
3. Commit: `chore: bump version to X.Y.Z`
4. Tag: `git tag vX.Y.Z && git push origin master --tags`
5. GitHub Actions builds release assets automatically
6. Repeat for prisma-gui, prisma-console, prisma-docs
7. Update prisma-gui submodule to point to new monorepo tag
8. Create GitHub Releases with changelogs
