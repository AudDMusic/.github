 <img src="https://audd.io/images/audd_cover_image.png" alt="AudD" height="60">
 
# How to identify music

**Identify songs from audio files, live streams, and large video files with industry-leading accuracy.**

[Get API Token](https://dashboard.audd.io/) · [Full Docs](https://docs.audd.io/) · [Support](mailto:api@audd.io)

---

## Quick Start

```bash
curl https://api.audd.io/ \
  -F url='https://audd.tech/example.mp3' \
  -F api_token='your_api_token'
```

Sign up at [dashboard.audd.io](https://dashboard.audd.io/) to get your API token.

## API Endpoints

| Endpoint | Best For | Limits | Response Time |
|----------|----------|--------|---------------|
| [`api.audd.io/`](#standard-recognition) | Single song ID | Up to 12s of audio | ~0.1–1.5s |
| [`api.audd.io/addStream/`](#live-stream-monitoring) | Real-time monitoring | Continuous streams | Real-time |
| [`enterprise.audd.io/`](#enterprise--large-files) | Large files, mixes | Unlimited length | Seconds to minutes |

## Standard Recognition

Identify a single song from a short audio clip — works like Shazam.

**Parameters:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `api_token` | ✅ | Your auth token from the [Dashboard](https://dashboard.audd.io/) |
| `url` | | URL of the audio file to recognize |
| `file` | | Audio file via multipart/form-data |
| `return` | | Comma-separated: `apple_music`, `spotify`, `deezer`, `napster`, `musicbrainz` |
| `market` | | Country code for Apple Music/Spotify results (default: `us`) |

You must provide either `url` or `file`. We recommend `url` when the file is accessible online.

### Recognize by URL

<details>
<summary><strong>cURL</strong></summary>

```bash
curl https://api.audd.io/ \
  -F url='https://audd.tech/example.mp3' \
  -F return='apple_music,spotify' \
  -F api_token='your_api_token'
```
</details>

<details>
<summary><strong>Python</strong></summary>

```python
import requests

data = {
    'url': 'https://audd.tech/example.mp3',
    'return': 'apple_music,spotify',
    'api_token': 'your_api_token',
}

response = requests.post('https://api.audd.io/', data=data)
result = response.json()

if result['status'] == 'success' and result['result']:
    song = result['result']
    print(f"{song['artist']} — {song['title']}")
    print(f"Album: {song['album']}")
    print(f"Timecode: {song['timecode']}")
else:
    print('No match found')
```
</details>

<details>
<summary><strong>Node.js</strong></summary>

```javascript
const response = await fetch('https://api.audd.io/', {
  method: 'POST',
  body: new URLSearchParams({
    url: 'https://audd.tech/example.mp3',
    return: 'apple_music,spotify',
    api_token: 'your_api_token',
  }),
});

const { status, result } = await response.json();

if (status === 'success' && result) {
  console.log(`${result.artist} — ${result.title}`);
  console.log(`Album: ${result.album}`);
}
```
</details>

<details>
<summary><strong>Go</strong></summary>

```go
import "github.com/AudDMusic/audd-go"

client := audd.NewClient("your_api_token")
song, err := client.RecognizeByUrl(
    "https://audd.tech/example.mp3",
    "apple_music,spotify",
    nil,
)
if err == nil && song != nil {
    fmt.Printf("%s — %s\n", song.Artist, song.Title)
}
```
</details>

### Recognize by File Upload

<details>
<summary><strong>cURL</strong></summary>

```bash
curl https://api.audd.io/ \
  -F file=@/path/to/audio.mp3 \
  -F return='apple_music,spotify' \
  -F api_token='your_api_token'
```
</details>

<details>
<summary><strong>Python</strong></summary>

```python
import requests

with open('audio.mp3', 'rb') as f:
    data = {
        'return': 'apple_music,spotify',
        'api_token': 'your_api_token',
    }
    files = {'file': f}
    response = requests.post('https://api.audd.io/', data=data, files=files)

result = response.json()
if result['status'] == 'success' and result['result']:
    song = result['result']
    print(f"{song['artist']} — {song['title']}")
```
</details>

<details>
<summary><strong>Node.js</strong></summary>

```javascript
import { readFileSync } from 'fs';

const form = new FormData();
form.append('file', new Blob([readFileSync('audio.mp3')]));
form.append('return', 'apple_music,spotify');
form.append('api_token', 'your_api_token');

const response = await fetch('https://api.audd.io/', {
  method: 'POST',
  body: form,
});

const { status, result } = await response.json();
if (status === 'success' && result) {
  console.log(`${result.artist} — ${result.title}`);
}
```
</details>

### Example Response

```json
{
  "status": "success",
  "result": {
    "artist": "Imagine Dragons",
    "title": "Warriors",
    "album": "Warriors",
    "release_date": "2014-09-18",
    "label": "Universal Music",
    "timecode": "02:32",
    "song_link": "https://lis.tn/Warriors",
    "apple_music": { "...": "..." },
    "spotify": { "...": "..." }
  }
}
```

When `result` is `null`, no match was found. The `timecode` field indicates the position within the original song where your clip was playing.

> **WebSocket support:** Connect to `wss://api.audd.io/ws/?api_token=[token]` and send multiple files (binary) without waiting for responses — great for high-throughput pipelines.

## Enterprise — Large Files

The enterprise endpoint accepts files of **any length** — hours-long DJ mixes, full radio recordings, video files — and returns every recognized track with timestamps.

Requests are counted as **1 per 12 seconds of audio**. Use `skip` and `every` to control cost.

**Parameters:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `api_token` | ✅ | Your auth token |
| `url` | | URL of the file (can be a web page containing audio/video) |
| `file` | | File via multipart/form-data |
| `accurate_offsets` | | Set to `"true"` for precise start/end offsets |
| `skip` | | Number of 12s chunks to skip after each scanned chunk |
| `every` | | Number of consecutive chunks to scan |
| `skip_first_seconds` | | Skip N seconds at the beginning of the file |

> **Cost example:** `skip=4&every=1` scans 12s then skips 48s → 1 request per minute of audio. `skip=9&every=1` → 1 request per 2 minutes.

<details>
<summary><strong>cURL</strong></summary>

```bash
curl https://enterprise.audd.io/ \
  -F url='https://audd.tech/djatwork_example.mp3' \
  -F accurate_offsets='true' \
  -F api_token='your_api_token'
```
</details>

<details>
<summary><strong>Python</strong></summary>

```python
import requests

data = {
    'url': 'https://audd.tech/djatwork_example.mp3',
    'accurate_offsets': 'true',
    'api_token': 'your_api_token',
}

response = requests.post('https://enterprise.audd.io/', data=data)
result = response.json()

for chunk in result.get('result', []):
    offset = chunk['offset']
    for song in chunk['songs']:
        print(f"[{offset}] {song['artist']} — {song['title']} (score: {song['score']})")
```
</details>

<details>
<summary><strong>Node.js</strong></summary>

```javascript
const response = await fetch('https://enterprise.audd.io/', {
  method: 'POST',
  body: new URLSearchParams({
    url: 'https://audd.tech/djatwork_example.mp3',
    accurate_offsets: 'true',
    api_token: 'your_api_token',
  }),
});

const { result } = await response.json();
for (const chunk of result) {
  for (const song of chunk.songs) {
    console.log(`[${chunk.offset}] ${song.artist} — ${song.title}`);
  }
}
```
</details>

### Understanding the Response

| Field | Meaning |
|-------|---------|
| `offset` | Position in *your file* where the 12s chunk starts (e.g., `"04:48"`) |
| `songs[].timecode` | Position in the *original song* being played |
| `songs[].score` | Confidence score (0–100) |
| `songs[].start_offset` / `end_offset` | Millisecond positions within the 12s chunk where the match occurs |

## Live Stream Monitoring

Monitor radio stations, Twitch broadcasts, YouTube live streams, and any audio stream in real time. You provide stream URLs, AudD monitors them continuously, and sends recognition results via callbacks or long polling.

**Pricing:** $45/stream/month with AudD's music DB, or $25/stream/month with your own catalog.

### Setup

**Step 1** — Set your callback URL:

```bash
curl https://api.audd.io/setCallbackUrl/ \
  -F url='https://yourserver.com/audd-webhook' \
  -F api_token='your_api_token'
```

If you don't have a server, use `https://audd.tech/empty/` and retrieve results via long polling.

**Step 2** — Add streams:

```bash
# Radio station
curl https://api.audd.io/addStream/ \
  -F url='https://npr-ice.streamguys1.com/live.mp3' \
  -F radio_id='3249' \
  -F api_token='your_api_token'

# Twitch channel
curl https://api.audd.io/addStream/ \
  -F url='twitch:monstercat' \
  -F radio_id='5513' \
  -F api_token='your_api_token'

# YouTube live stream
curl https://api.audd.io/addStream/ \
  -F url='youtube:5qap5aO4i9A' \
  -F radio_id='5512' \
  -F api_token='your_api_token'
```

**Step 3** — Receive results via your callback URL:

```json
{
  "status": "success",
  "result": {
    "radio_id": 7,
    "timestamp": "2020-04-13 10:31:43",
    "play_length": 111,
    "results": [{
      "artist": "Alan Walker, A$AP Rocky",
      "title": "Live Fast (PUBGM)",
      "score": 100,
      "song_link": "https://lis.tn/LiveFastPUBGM"
    }]
  }
}
```

By default, callbacks are sent when a song **finishes** playing. Add `callbacks=before` when calling `addStream` to receive results when songs **start** playing instead.

### Supported Stream URL Formats

| Platform | Format |
|----------|--------|
| Radio / Icecast / HLS / DASH | `https://stream-url.com/live.mp3` |
| Twitch | `twitch:channelname` |
| YouTube (video) | `youtube:videoId` |
| YouTube (channel) | `youtube-ch:channelId` |

### Stream Management Endpoints

| Endpoint | Purpose |
|----------|---------|
| `POST /getStreams/` | List all active streams |
| `POST /setStreamUrl/` | Update a stream's URL |
| `POST /deleteStream/` | Remove a stream |
| `POST /getCallbackUrl/` | Check current callback URL |

### Long Polling & Widgets

As an alternative to callbacks, use long polling:

```
https://api.audd.io/longpoll/?category=[longpoll_category]&timeout=50&since_time=[timestamp]
```

Get `longpoll_category` from the `/getStreams/` response. Or use the embeddable HTML widget:

```
https://widget.audd.tech/?ch=-[longpoll_category]&background&history&shadow
```

## Common Errors

| Code | Description |
|------|-------------|
| `#901` | No API token provided and free limit reached — [get a token](https://dashboard.audd.io/) |
| `#900` | Invalid API token |
| `#700` | No file received — ensure `Content-Type: multipart/form-data` and use `https://` (not `http://`) |
| `#600` | Incorrect audio URL — couldn't download |
| `#500` | Invalid audio file format |
| `#400` | File too large for standard endpoint (max 10MB / 25s) — use [enterprise](#enterprise--large-files) instead |
| `#300` | Fingerprinting error — audio clip is likely too short |

## Tips

- **Audio length:** For the standard endpoint, send 5–12 seconds of audio for best results.
- **Prefer URL over upload:** The `url` parameter is faster and more reliable since AudD downloads the file server-side.
- **Additional metadata:** Specify the `return` parameter to get streaming links — e.g., `return=apple_music,spotify`.
- **Enterprise cost control:** Use `skip` and `every` to sample long files instead of scanning every 12-second chunk.

## Resources

- **[API Dashboard](https://dashboard.audd.io/)** — Get your token, manage billing, configure streams
- **[Full Documentation](https://docs.audd.io/)** — Complete API reference
- **[Chrome Extension](https://github.com/AudDMusic/audd-chrome-extension)** — Recognize music from browser tabs (JS source)
- **[Discord Bot](https://github.com/AudDMusic/discord-bot)** — Identify music in Discord text/voice channels (Go)
- **[Go Library](https://github.com/AudDMusic/audd-go)** — Official Go client with long-polling support
- **[Code Examples on GitHub](https://github.com/search?o=desc&q=%22api.audd.io%22&s=indexed&type=Code)** — Community examples

## Support

Reach out to [api@audd.io](mailto:api@audd.io) for any questions, custom terms (>500k requests/month or >100 streams), or enterprise access for ISRCs/UPCs.
