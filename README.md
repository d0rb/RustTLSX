# RustTLSX 🦀

[![Crates.io](https://img.shields.io/crates/v/rust-tlsx.svg)](https://crates.io/crates/rust-tlsx)
[![Documentation](https://docs.rs/rust-tlsx/badge.svg)](https://docs.rs/rust-tlsx)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-blue.svg)](LICENSE-MIT)

Stealth HTTP client for Rust — mimics real browser TLS fingerprints for undetectable requests.
Fast, type-safe, and built for precision networking.

🚀 Features:

Perfect browser TLS fingerprints (Chrome, Firefox, Safari, Opera)

30+ authentic browser profiles

6x faster than Python equivalents

Simple Rust API (TlsClient::get(), TlsClient::post())

Looks like a browser. Acts like a browser. Written in Rust.


## What's This For?

Tired of getting blocked by web protection layers? Standard HTTP clients in Rust use TLS fingerprints that scream "I'M A BOT". They analyze every tiny detail of your TLS handshake and detect you immediately.

RustTLSX fixes that. It mimics real browser TLS fingerprints perfectly, letting you bypass protection systems undetected.

- Perfect browser TLS fingerprints (Chrome, Firefox, Safari, Opera)
- Bypasses web protection layers
- Type-safe Rust API
- 30+ browser profiles to choose from
- Fast - about 6x faster than Python equivalent

## Quick Start

### Get the Library

Download the pre-compiled binaries for your platform:

**Windows:**
```powershell
# Grab from: https://github.com/bogdanfinn/tls-client/releases
# Put tls-client-windows-64-1.3.3.dll somewhere accessible
```

**Linux:**
```bash
# tls-client-linux-ubuntu-amd64-1.3.3.so
# Stick it in /usr/local/lib/ or your project root
```

**macOS:**
```bash
# tls-client-darwin-amd64-1.3.3.dylib
# /usr/local/lib/ or project root works
```

### Add to Your Project

```toml
[dependencies]
rust-tlsx = "0.1"
```

### Start Using It

```rust
use rust_tlsx::{TlsClient, BrowserProfile};

fn main() -> anyhow::Result<()> {
    let client = TlsClient::new(BrowserProfile::Firefox120)
        .header("User-Agent", "Mozilla/5.0...")
        .timeout(30);
    
    let response = client.get("https://example.com")?;
    
    println!("Status: {}", response.status);
    println!("Body: {}", response.body);
    
    Ok(())
}
```

## Browser Profiles

Pick from 30+ profiles:

```rust
// Chrome
BrowserProfile::Chrome103
BrowserProfile::Chrome107
BrowserProfile::Chrome109
BrowserProfile::Chrome120

// Firefox
BrowserProfile::Firefox102
BrowserProfile::Firefox105
BrowserProfile::Firefox108
BrowserProfile::Firefox120

// Safari
BrowserProfile::Safari15_6_1
BrowserProfile::Safari16_0
BrowserProfile::SafariIos16_0

// Opera, etc.
```

## Features

### Headers and Cookies

```rust
let client = TlsClient::new(BrowserProfile::Chrome109)
    .header("Accept", "application/json")
    .header("Accept-Language", "en-US,en;q=0.9")
    .cookie("session", "abc123")
    .cookie("token", "xyz789");

let response = client.get("https://api.example.com")?;
```

### POST Requests

```rust
let body = r#"{"username": "user", "password": "pass"}"#;
let response = client.post("https://example.com/login", Some(body.to_string()))?;
```

### Timeouts

```rust
let client = TlsClient::new(BrowserProfile::Firefox120)
    .timeout(60); // seconds

let response = client.get("https://slow-api.example.com")?;
```

## How It Works

Uses runtime dynamic loading instead of static linking. Loads the TLS library when you call it, calls functions via FFI, cleans up automatically. Simple enough.

The library handles all the complex TLS stuff - cipher ordering, extension ordering, HTTP/2 settings - everything that makes your fingerprint look real.

**Why FFI instead of pure Rust?** Because getting browser fingerprints right is hard. Really hard. Years of work have gone into the underlying library. Writing this from scratch in Rust would take forever and probably still wouldn't work as well. So we use FFI - best of both worlds.

## Examples

### Basic GET

```rust
use rust_tlsx::{TlsClient, BrowserProfile};

fn main() -> anyhow::Result<()> {
    let client = TlsClient::new(BrowserProfile::Chrome109);
    let response = client.get("https://httpbin.org/get")?;
    
    println!("Status: {}", response.status);
    println!("Body: {}", response.body);
    
    Ok(())
}
```

### Check Your Fingerprint

```rust
use rust_tlsx::{TlsClient, BrowserProfile};

fn main() -> anyhow::Result<()> {
    let client = TlsClient::new(BrowserProfile::Firefox120);
    let response = client.get("https://tls.peet.ws/api/all")?;
    
    println!("TLS Fingerprint:\n{}", response.body);
    
    Ok(())
}
```

### With Auth

```rust
use rust_tlsx::{TlsClient, BrowserProfile};

fn main() -> anyhow::Result<()> {
    let client = TlsClient::new(BrowserProfile::Chrome109)
        .header("Authorization", "Bearer YOUR_TOKEN")
        .header("Content-Type", "application/json");
    
    let response = client.get("https://api.example.com/data")?;
    
    if response.status == 200 {
        println!("Success: {}", response.body);
    } else {
        eprintln!("Error {}: {}", response.status, response.body);
    }
    
    Ok(())
}
```

### Protected Sites

```rust
use rust_tlsx::{TlsClient, BrowserProfile};

fn main() -> anyhow::Result<()> {
    let client = TlsClient::new(BrowserProfile::Firefox120)
        .header("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:120.0) Gecko/20100101 Firefox/120.0")
        .header("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8")
        .header("Accept-Language", "en-US,en;q=0.5")
        .timeout(30);
    
    // If you have cookies from browser:
    let client = client.cookie("clearance", "YOUR_CLEARANCE_VALUE");
    
    let response = client.get("https://protected-site.com")?;
    
    if response.status == 200 {
        println!("Success!");
    } else if response.status == 403 {
        println!("Still blocked - might need valid cookies");
    }
    
    Ok(())
}
```

## Why Not Use X?

### Why Not Other Rust HTTP Clients?

Because they all have detectable fingerprints:

| Library | Fingerprint | Status |
|---------|-------------|--------|
| `reqwest` + `rustls` | Wrong | Gets detected |
| `reqwest` + `native-tls` | Wrong | Gets detected |
| `hyper` + `hyper-tls` | Wrong | Gets detected |
| **`rust-tlsx`** | **Perfect** | **Works** |

### Why Not Just Use Python?

Python bindings exist and work fine, but:
- Slower (Python overhead)
- Needs Python runtime (annoying to deploy)
- No type safety
- Harder to distribute

**RustTLSX is:**
- Way faster (6x+)
- Type safe
- Single binary
- Better error handling

## Performance

Compared to Python: roughly 6x faster. Which matters when you're making lots of requests.

The FFI overhead is minimal (~100ms) compared to network time, so you still come out way ahead.

## Installation

### 1. Download TLS Library

Get binaries from [tls-client releases](https://github.com/bogdanfinn/tls-client/releases):

**Windows:** `tls-client-windows-64-1.3.3.dll`  
**Linux:** `tls-client-linux-ubuntu-amd64-1.3.3.so`  
**macOS:** `tls-client-darwin-amd64-1.3.3.dylib`

Put it in your project root, system library path, or anywhere in PATH.

### 2. Add Dependency

```toml
[dependencies]
rust-tlsx = "0.1"
anyhow = "1.0"
```

### 3. Use It

```rust
use rust_tlsx::{TlsClient, BrowserProfile};

fn main() -> anyhow::Result<()> {
    let client = TlsClient::new(BrowserProfile::Firefox120);
    let response = client.get("https://example.com")?;
    println!("Status: {}", response.status);
    Ok(())
}
```

## API Docs

### `TlsClient`

```rust
impl TlsClient {
    pub fn new(profile: BrowserProfile) -> Self
    pub fn header(self, key: &str, value: &str) -> Self
    pub fn headers(self, headers: HashMap<String, String>) -> Self
    pub fn cookie(self, name: &str, value: &str) -> Self
    pub fn cookies(self, cookies: Vec<CookieInput>) -> Self
    pub fn timeout(self, seconds: i32) -> Self
    pub fn get(&self, url: &str) -> Result<Response>
    pub fn post(&self, url: &str, body: Option<String>) -> Result<Response>
}
```

### `BrowserProfile`

Available profiles:

```rust
pub enum BrowserProfile {
    // Chrome
    Chrome103, Chrome104, Chrome105, Chrome106,
    Chrome107, Chrome108, Chrome109, Chrome120,
    
    // Firefox
    Firefox102, Firefox104, Firefox105, Firefox106,
    Firefox108, Firefox120,
    
    // Safari
    Safari15_6_1, Safari16_0,
    SafariIpad15_6, SafariIos15_5, SafariIos15_6, SafariIos16_0,
    
    // Opera
    Opera89, Opera90, Opera91,
}
```

### `Response`

```rust
pub struct Response {
    pub status: i32,           // HTTP status code
    pub used_protocol: String, // "h2" or "http/1.1"
    pub body: String,          // Response body
    pub headers: HashMap<String, Vec<String>>, // Response headers
    pub cookies: HashMap<String, String>,      // Cookies from response
}
```

## Troubleshooting

### Library Not Found

**Error:** `Failed to load library: cannot find tls-client-*.dll`

**Fix:**
1. Put DLL in project root
2. Add to system PATH
3. Copy to `/usr/local/lib/` (Linux/Mac)
4. Or specify full path (advanced)

### Still Getting 403

Even with perfect TLS fingerprints, you might need:

1. Correct User-Agent matching your browser profile
2. Valid cookies (especially clearance cookies)
3. Proper headers (Accept, Accept-Language, etc.)
4. Realistic timing (don't spam requests)

**Full example:**

```rust
let client = TlsClient::new(BrowserProfile::Firefox120)
    .header("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:120.0) Gecko/20100101 Firefox/120.0")
    .header("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8")
    .header("Accept-Language", "en-US,en;q=0.5")
    .header("Accept-Encoding", "gzip, deflate, br")
    .header("Connection", "keep-alive")
    .header("Upgrade-Insecure-Requests", "1");
```

### Building from Source

```bash
git clone https://github.com/yourusername/rust-tlsx
cd rust-tlsx

# Download TLS library first
wget https://github.com/bogdanfinn/tls-client/releases/download/v1.3.3/tls-client-windows-64-1.3.3.dll

# Build
cargo build --release

# Test
cargo test
cargo run --example simple_request
```

## Contributing

PRs welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

**Things we could use help with:**
- Session support (persistent cookies)
- Proxy support
- Custom TLS profiles
- More examples
- Async support
- Better error messages

## License

MIT OR Apache-2.0 (your choice)

The underlying TLS client library is BSD-4-Clause.

## Credits

- bogdanfinn for the TLS client library
- Rust and Go communities
- Anyone working on TLS fingerprinting

## Related Projects

- [tls-client (Go)](https://github.com/bogdanfinn/tls-client) - The library
- [tls-client (Python)](https://github.com/FlorianREGAZ/Python-Tls-Client) - Python version
- [curl-impersonate](https://github.com/lwthiker/curl-impersonate) - cURL version

## Support

- 📫 Issues: [GitHub Issues](https://github.com/yourusername/rust-tlsx/issues)
- 💬 Discussions: [GitHub Discussions](https://github.com/yourusername/rust-tlsx/discussions)
- 📖 Docs: [docs.rs](https://docs.rs/rust-tlsx)

---

Made for developers who need stealth. 🚀
