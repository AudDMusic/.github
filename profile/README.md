<img src="https://audd.io/images/audd_cover_image.png" alt="AudD" height="60">

# AudD — Music Recognition API

Identify songs from audio clips, long broadcasts, and live audio streams.

[**Get an API token**](https://dashboard.audd.io/) · [**Full docs**](https://docs.audd.io/) · [Support](mailto:api@audd.io)

---

## Quick taste

```python
from audd import AudD

audd = AudD("your-api-token")
song = audd.recognize("https://audd.tech/example.mp3")
print(f"{song.artist} — {song.title}")
```

```bash
curl https://api.audd.io/ \
  -F url='https://audd.tech/example.mp3' \
  -F api_token='your-api-token'
```

The HTTP API is intentionally simple — one POST with a couple of form fields, callable from any HTTP client. **You don't need an SDK**: the curl above is the entire shape of a recognition request, and the same shape works in any language. The official SDKs below are useful when you want typed results, retries, longpoll consumers, or web-framework recipes — otherwise, hit the endpoint directly.

The public token `"test"` works for hello-worlds (capped at 10 requests/day). [Get your own token at dashboard.audd.io](https://dashboard.audd.io).

## Pick your SDK

| Language | Install | Source | Docs |
|---|---|---|---|
| **Python** | `pip install audd` | [audd-python](https://github.com/AudDMusic/audd-python) | [docs](https://docs.audd.io/sdks/python) |
| **Node / TypeScript** | `npm install @audd/sdk` | [audd-node](https://github.com/AudDMusic/audd-node) | [docs](https://docs.audd.io/sdks/node) |
| **Go** | `go get github.com/AudDMusic/audd-go` | [audd-go](https://github.com/AudDMusic/audd-go) | [docs](https://docs.audd.io/sdks/go) |
| **Rust** | `cargo add audd` | [audd-rust](https://github.com/AudDMusic/audd-rust) | [docs](https://docs.audd.io/sdks/rust) |
| **PHP** | `composer require audd/audd` | [audd-php](https://github.com/AudDMusic/audd-php) | [docs](https://docs.audd.io/sdks/php) |
| **Swift** | Swift Package Manager | [audd-swift](https://github.com/AudDMusic/audd-swift) | [docs](https://docs.audd.io/sdks/swift) |
| **Kotlin** | `io.audd:audd-kotlin` (Maven Central) | [audd-kotlin](https://github.com/AudDMusic/audd-kotlin) | [docs](https://docs.audd.io/sdks/kotlin) |
| **C#** / **.NET** | `dotnet add package AudD` | [audd-dotnet](https://github.com/AudDMusic/audd-dotnet) | [docs](https://docs.audd.io/sdks/dotnet) |
| **Java** | `io.audd:audd` (Maven Central) | [audd-java](https://github.com/AudDMusic/audd-java) | [docs](https://docs.audd.io/sdks/java) |
| **C** | CMake / source dist | [audd-c](https://github.com/AudDMusic/audd-c) | [docs](https://docs.audd.io/sdks/c) |
| **C++** | CMake / source dist | [audd-cpp](https://github.com/AudDMusic/audd-cpp) | [docs](https://docs.audd.io/sdks/cpp) |

## What you can build

- **Recognize a clip** — short-audio identification, like Shazam.
- **Process long files** — broadcasts, podcasts, full DJ sets, multi-hour recordings; returns every match with timestamps.
- **Monitor a live stream** — radio URLs, Twitch / YouTube live, HLS / Icecast / DASH; AudD ingests continuously and notifies you on each match via webhook callbacks or longpoll.
- **Build a private catalog** — fingerprint your own tracks so future recognition calls match against them (special access via [api@audd.io](mailto:api@audd.io)).

Full reference at [docs.audd.io](https://docs.audd.io).

## OpenAPI spec

- [**audd-openapi**](https://github.com/AudDMusic/audd-openapi) — canonical OpenAPI 3.1 spec for the AudD HTTP API. Generate your own client, or use it as a reference.

## Other open-source projects

- [**audd-chrome-extension**](https://github.com/AudDMusic/audd-chrome-extension) — recognize music in any browser tab.
- [**discord-bot**](https://github.com/AudDMusic/discord-bot) — identify music in Discord text & voice channels (Go).

## Support

- **API tokens & billing:** [dashboard.audd.io](https://dashboard.audd.io)
- **Documentation:** [docs.audd.io](https://docs.audd.io)
- **Email:** [api@audd.io](mailto:api@audd.io)
- **Custom terms** — >500k requests/month, >100 simultaneous streams, or enterprise account for ISRC/UPC metadata: [api@audd.io](mailto:api@audd.io).
