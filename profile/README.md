<img src="https://audd.io/images/audd_cover_image.png" alt="AudD" height="60">

# AudD — Music Recognition API

Identify songs from short audio clips, hours-long broadcast recordings, and live audio streams.

[**Get an API token**](https://dashboard.audd.io/) · [**Full docs**](https://docs.audd.io/) · [Support](mailto:api@audd.io)

---

## Quick Start

```bash
curl https://api.audd.io/ \
  -F url='https://audd.tech/example.mp3' \
  -F api_token='your-api-token'
```

Get your API token at [dashboard.audd.io](https://dashboard.audd.io/). The string `"test"` works for hello-worlds (capped at 10 requests/day).

The API is one HTTP POST and works from any HTTP client. Official SDKs are available for the 11 languages below — they add typed results, retries, and longpoll consumers, but they aren't required.

**Available SDKs:** [Python](https://github.com/AudDMusic/audd-python) · [Node / TypeScript](https://github.com/AudDMusic/audd-node) · [Go](https://github.com/AudDMusic/audd-go) · [Rust](https://github.com/AudDMusic/audd-rust) · [PHP](https://github.com/AudDMusic/audd-php) · [Swift](https://github.com/AudDMusic/audd-swift) · [Kotlin](https://github.com/AudDMusic/audd-kotlin) · [C# / .NET](https://github.com/AudDMusic/audd-dotnet) · [Java](https://github.com/AudDMusic/audd-java) · [C](https://github.com/AudDMusic/audd-c) · [C++](https://github.com/AudDMusic/audd-cpp)

---

## API Endpoints

| Endpoint | Best for | Limits | Response time |
|---|---|---|---|
| [`api.audd.io/`](#recognize-a-song) | Single short clip (Shazam-style) | Up to 12s of audio | ~0.1–1.5s |
| [`enterprise.audd.io/`](#process-long-files-enterprise) | Hours-long mixes, broadcast recordings, podcasts | Unlimited length | Seconds to minutes |
| [`api.audd.io/addStream/`](#monitor-live-streams) | Live broadcasts, radio, Twitch, YouTube live | Continuous | Real-time |

---

## Recognize a Song

Identify a single song from a short audio clip — works like Shazam.

**Parameters:**

| Parameter | Required | Description |
|---|---|---|
| `api_token` | ✅ | Your auth token from the [Dashboard](https://dashboard.audd.io/) |
| `url` | one of `url` / `file` | URL of the audio file to recognize |
| `file` | one of `url` / `file` | Audio file via `multipart/form-data` |
| `return` | | Comma-separated metadata providers: `apple_music`, `spotify`, `deezer`, `napster`, `musicbrainz`. AudD identifies the song from your audio and, if matched, attaches metadata from each requested provider. |
| `market` | | Country code for Apple Music / Spotify links (default: `us`) |

### From a URL

Each language tab shows the raw HTTP form (no SDK), then the official SDK form for the same call.

<details>
<summary><strong>cURL</strong></summary>

```bash
curl https://api.audd.io/ \
  -F url='https://audd.tech/example.mp3' \
  -F return='apple_music,spotify' \
  -F api_token='your-api-token'
```

</details>

<details>
<summary><strong>Python</strong></summary>

HTTP, no SDK:

```python
import requests

r = requests.post("https://api.audd.io/", data={
    "url": "https://audd.tech/example.mp3",
    "return": "apple_music,spotify",
    "api_token": "your-api-token",
}).json()

if r["status"] == "success" and r["result"]:
    s = r["result"]
    print(f"{s['artist']} — {s['title']}")
    print(s["apple_music"]["url"])
```

With the SDK ([github](https://github.com/AudDMusic/audd-python), [docs](https://docs.audd.io/sdks/python)):

```bash
pip install audd
```

```python
from audd import AudD

audd = AudD("your-api-token")
result = audd.recognize(
    "https://audd.tech/example.mp3",
    return_metadata=["apple_music", "spotify"],
)
print(f"{result.artist} — {result.title}")
print(result.apple_music.url)         # direct Apple Music link
print(result.spotify.uri)              # spotify:track:...
```

</details>

<details>
<summary><strong>Node / TypeScript</strong></summary>

HTTP, no SDK:

```typescript
const res = await fetch("https://api.audd.io/", {
  method: "POST",
  body: new URLSearchParams({
    url: "https://audd.tech/example.mp3",
    return: "apple_music,spotify",
    api_token: "your-api-token",
  }),
});
const { status, result } = await res.json();
if (status === "success" && result) {
  console.log(`${result.artist} — ${result.title}`);
  console.log(result.apple_music.url);
}
```

With the SDK ([github](https://github.com/AudDMusic/audd-node), [docs](https://docs.audd.io/sdks/node)):

```bash
npm install @audd/sdk
```

```typescript
import { AudD } from "@audd/sdk";

const audd = new AudD("your-api-token");
const song = await audd.recognize("https://audd.tech/example.mp3", {
  return: ["apple_music", "spotify"],
});
if (song) {
  console.log(`${song.artist} — ${song.title}`);
  console.log(song.appleMusic?.url);
  console.log(song.spotify?.uri);
}
```

</details>

<details>
<summary><strong>Go</strong></summary>

HTTP, no SDK:

```go
import (
    "encoding/json"
    "net/http"
    "net/url"
    "strings"
)

resp, _ := http.Post(
    "https://api.audd.io/",
    "application/x-www-form-urlencoded",
    strings.NewReader(url.Values{
        "url":       {"https://audd.tech/example.mp3"},
        "return":    {"apple_music,spotify"},
        "api_token": {"your-api-token"},
    }.Encode()),
)
defer resp.Body.Close()

var body struct {
    Result *struct {
        Artist, Title string
        AppleMusic    struct{ URL string } `json:"apple_music"`
    }
}
json.NewDecoder(resp.Body).Decode(&body)
if body.Result != nil {
    fmt.Printf("%s — %s\n%s\n", body.Result.Artist, body.Result.Title, body.Result.AppleMusic.URL)
}
```

With the SDK ([github](https://github.com/AudDMusic/audd-go), [docs](https://docs.audd.io/sdks/go)):

```bash
go get github.com/AudDMusic/audd-go
```

```go
import audd "github.com/AudDMusic/audd-go"

client := audd.NewClient("your-api-token")
defer client.Close()

result, err := client.Recognize("https://audd.tech/example.mp3", &audd.RecognizeOptions{
    ReturnMetadata: "apple_music,spotify",
})
if err != nil { log.Fatal(err) }
fmt.Printf("%s — %s\n", result.Artist, result.Title)
fmt.Println("Apple Music:", result.AppleMusic.URL)
fmt.Println("Spotify URI:", result.Spotify.URI)
```

</details>

<details>
<summary><strong>Rust</strong></summary>

HTTP, no SDK (using `reqwest`):

```rust
let r: serde_json::Value = reqwest::Client::new()
    .post("https://api.audd.io/")
    .form(&[
        ("url", "https://audd.tech/example.mp3"),
        ("return", "apple_music,spotify"),
        ("api_token", "your-api-token"),
    ])
    .send().await?
    .json().await?;

if let Some(s) = r["result"].as_object() {
    println!("{} — {}", s["artist"], s["title"]);
    println!("{}", s["apple_music"]["url"]);
}
```

With the SDK ([github](https://github.com/AudDMusic/audd-rust), [docs](https://docs.audd.io/sdks/rust)):

```bash
cargo add audd
```

```rust
use audd::{AudD, RecognizeOptions};

let audd = AudD::new("your-api-token");
let return_metadata = ["apple_music".into(), "spotify".into()];
let Some(r) = audd
    .recognize_with(
        "https://audd.tech/example.mp3",
        RecognizeOptions { return_metadata: Some(&return_metadata), ..Default::default() },
    )
    .await? else { return Ok(()) };

println!("{} — {}", r.artist, r.title);
if let Some(am) = r.apple_music.as_ref() {
    println!("Apple Music: {}", am.url.as_deref().unwrap_or(""));
}
```

</details>

<details>
<summary><strong>PHP</strong></summary>

HTTP, no SDK:

```php
$r = json_decode(file_get_contents(
    'https://api.audd.io/',
    false,
    stream_context_create(['http' => [
        'method' => 'POST',
        'header' => 'Content-Type: application/x-www-form-urlencoded',
        'content' => http_build_query([
            'url' => 'https://audd.tech/example.mp3',
            'return' => 'apple_music,spotify',
            'api_token' => 'your-api-token',
        ]),
    ]])
), true);

if ($r['status'] === 'success' && $r['result']) {
    $s = $r['result'];
    echo "{$s['artist']} — {$s['title']}\n";
    echo $s['apple_music']['url'] . "\n";
}
```

With the SDK ([github](https://github.com/AudDMusic/audd-php), [docs](https://docs.audd.io/sdks/php)):

```bash
composer require audd/audd
```

```php
use AudD\AudD;

$audd = new AudD('your-api-token');
$result = $audd->recognize(
    'https://audd.tech/example.mp3',
    returnMetadata: ['apple_music', 'spotify'],
);
echo "{$result->artist} — {$result->title}\n";
echo $result->apple_music->url, "\n";
echo $result->spotify->uri, "\n";
```

</details>

<details>
<summary><strong>Swift</strong></summary>

HTTP, no SDK:

```swift
var req = URLRequest(url: URL(string: "https://api.audd.io/")!)
req.httpMethod = "POST"
req.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
req.httpBody = "url=https://audd.tech/example.mp3&return=apple_music,spotify&api_token=your-api-token"
    .data(using: .utf8)

let (data, _) = try await URLSession.shared.data(for: req)
let json = try JSONSerialization.jsonObject(with: data) as? [String: Any]
if let result = json?["result"] as? [String: Any] {
    print("\(result["artist"]!) — \(result["title"]!)")
}
```

With the SDK ([github](https://github.com/AudDMusic/audd-swift), [docs](https://docs.audd.io/sdks/swift)):

```swift
// Package.swift: .package(url: "https://github.com/AudDMusic/audd-swift", from: "1.0.0")
import AudD

let audd = try AudD(apiToken: "your-api-token")
guard let result = try await audd.recognize(
    "https://audd.tech/example.mp3",
    return: ["apple_music", "spotify"]
) else { return }

print("\(result.artist ?? "?") — \(result.title ?? "?")")
print(result.appleMusic?.url ?? "")
print(result.spotify?.uri ?? "")
```

</details>

<details>
<summary><strong>Kotlin</strong></summary>

HTTP, no SDK (using OkHttp):

```kotlin
val body = FormBody.Builder()
    .add("url", "https://audd.tech/example.mp3")
    .add("return", "apple_music,spotify")
    .add("api_token", "your-api-token")
    .build()

val response = OkHttpClient().newCall(
    Request.Builder().url("https://api.audd.io/").post(body).build()
).execute()
println(response.body?.string())
```

With the SDK ([github](https://github.com/AudDMusic/audd-kotlin), [docs](https://docs.audd.io/sdks/kotlin)):

```kotlin
// Gradle: implementation("io.audd:audd-kotlin:1.0.0")
import io.audd.AudD

val audd = AudD("your-api-token")
val result = audd.recognize(
    "https://audd.tech/example.mp3",
    returnExtras = listOf("apple_music", "spotify"),
)
result?.let {
    println("${it.artist} — ${it.title}")
    println(it.appleMusic?.get("url"))
    println(it.spotify?.get("uri"))
}
```

</details>

<details>
<summary><strong>C# / .NET</strong></summary>

HTTP, no SDK:

```csharp
using var http = new HttpClient();
var content = new FormUrlEncodedContent(new Dictionary<string, string> {
    ["url"] = "https://audd.tech/example.mp3",
    ["return"] = "apple_music,spotify",
    ["api_token"] = "your-api-token",
});
var response = await http.PostAsync("https://api.audd.io/", content);
Console.WriteLine(await response.Content.ReadAsStringAsync());
```

With the SDK ([github](https://github.com/AudDMusic/audd-dotnet), [docs](https://docs.audd.io/sdks/dotnet)):

```bash
dotnet add package AudD
```

```csharp
using AudD;

await using var audd = new AudD.AudD("your-api-token");
var result = await audd.RecognizeAsync(
    "https://audd.tech/example.mp3",
    @return: new[] { "apple_music", "spotify" });

Console.WriteLine($"{result?.Artist} — {result?.Title}");
Console.WriteLine(result?.AppleMusic?.Url);
Console.WriteLine(result?.Spotify?.Uri);
```

</details>

<details>
<summary><strong>Java</strong></summary>

HTTP, no SDK (`java.net.http`):

```java
var client = HttpClient.newHttpClient();
var body = "url=https://audd.tech/example.mp3&return=apple_music,spotify&api_token=your-api-token";
var req = HttpRequest.newBuilder(URI.create("https://api.audd.io/"))
    .header("Content-Type", "application/x-www-form-urlencoded")
    .POST(BodyPublishers.ofString(body))
    .build();
System.out.println(client.send(req, BodyHandlers.ofString()).body());
```

With the SDK ([github](https://github.com/AudDMusic/audd-java), [docs](https://docs.audd.io/sdks/java)):

```xml
<!-- Maven -->
<dependency><groupId>io.audd</groupId><artifactId>audd</artifactId><version>1.0.0</version></dependency>
```

```java
import io.audd.AudD;
import io.audd.RecognizeOptions;

var audd = new AudD("your-api-token");
var r = audd.recognize(
    "https://audd.tech/example.mp3",
    RecognizeOptions.builder()
        .returnMetadata("apple_music", "spotify")
        .build()
);
System.out.println(r.artist() + " — " + r.title());
if (r.appleMusic() != null) System.out.println(r.appleMusic().url());
if (r.spotify()    != null) System.out.println(r.spotify().uri());
```

</details>

<details>
<summary><strong>C</strong></summary>

HTTP, no SDK (using libcurl):

```c
CURL *c = curl_easy_init();
curl_easy_setopt(c, CURLOPT_URL, "https://api.audd.io/");
curl_easy_setopt(c, CURLOPT_POSTFIELDS,
    "url=https://audd.tech/example.mp3&return=apple_music,spotify&api_token=your-api-token");
curl_easy_perform(c);  /* response goes to stdout by default */
curl_easy_cleanup(c);
```

With the SDK ([github](https://github.com/AudDMusic/audd-c), [docs](https://docs.audd.io/sdks/c)):

```cmake
# CMake
add_subdirectory(audd-c)
target_link_libraries(your_app PRIVATE audd)
```

```c
#include <audd.h>

audd_client_t *client = audd_client_new("your-api-token", NULL);
const char *want[] = { "apple_music", "spotify", NULL };
audd_recognize_options_t opts = { .return_metadata = want };

audd_recognition_t *r = NULL;
if (audd_recognize(client, "https://audd.tech/example.mp3", &opts, &r) == AUDD_OK && r) {
    printf("%s — %s\n", audd_recognition_artist(r), audd_recognition_title(r));
    const audd_apple_music_t *am = audd_recognition_apple_music(r);
    const audd_spotify_t     *sp = audd_recognition_spotify(r);
    if (am) printf("Apple Music: %s\n", audd_apple_music_get_url(am));
    if (sp) printf("Spotify URI: %s\n", audd_spotify_get_uri(sp));
    audd_recognition_free(r);
}
audd_client_free(client);
```

</details>

<details>
<summary><strong>C++</strong></summary>

HTTP, no SDK (using libcurl):

```cpp
CURL *c = curl_easy_init();
curl_easy_setopt(c, CURLOPT_URL, "https://api.audd.io/");
curl_easy_setopt(c, CURLOPT_POSTFIELDS,
    "url=https://audd.tech/example.mp3&return=apple_music,spotify&api_token=your-api-token");
curl_easy_perform(c);
curl_easy_cleanup(c);
```

With the SDK ([github](https://github.com/AudDMusic/audd-cpp), [docs](https://docs.audd.io/sdks/cpp)):

```cmake
# CMake
add_subdirectory(audd-cpp)
target_link_libraries(your_app PRIVATE audd::audd)
```

```cpp
#include <audd/audd.hpp>

audd::AudD client("your-api-token");
audd::RecognizeOptions opts;
opts.return_metadata = {"apple_music", "spotify"};

if (auto result = client.recognize("https://audd.tech/example.mp3", opts)) {
    std::cout << result->artist << " — " << result->title << "\n";
    if (result->apple_music) std::cout << "Apple Music: " << result->apple_music->url << "\n";
    if (result->spotify)     std::cout << "Spotify URI: " << result->spotify->uri << "\n";
}
```

</details>

### From a local file

Same call, just point at a local path. SDKs that auto-detect treat strings as URL-or-path; SDKs that take an explicit `Source` accept a file variant.

<details>
<summary><strong>cURL</strong></summary>

```bash
curl https://api.audd.io/ \
  -F file=@/path/to/audio.mp3 \
  -F return='apple_music,spotify' \
  -F api_token='your-api-token'
```

</details>

<details>
<summary><strong>Python</strong></summary>

HTTP, no SDK:

```python
import requests

with open("audio.mp3", "rb") as f:
    r = requests.post("https://api.audd.io/",
        data={"return": "apple_music,spotify", "api_token": "your-api-token"},
        files={"file": f},
    ).json()

if r["status"] == "success" and r["result"]:
    s = r["result"]
    print(f"{s['artist']} — {s['title']}")
    print(s["apple_music"]["url"])
```

With the SDK ([github](https://github.com/AudDMusic/audd-python), [docs](https://docs.audd.io/sdks/python)):

```python
result = audd.recognize("audio.mp3", return_metadata=["apple_music", "spotify"])
print(f"{result.artist} — {result.title}")
print(result.apple_music.url)
```

</details>

<details>
<summary><strong>Node / TypeScript</strong></summary>

HTTP, no SDK:

```typescript
import { readFileSync } from "node:fs";

const form = new FormData();
form.append("file", new Blob([readFileSync("audio.mp3")]));
form.append("return", "apple_music,spotify");
form.append("api_token", "your-api-token");

const res = await fetch("https://api.audd.io/", { method: "POST", body: form });
const { status, result } = await res.json();
if (status === "success" && result) {
  console.log(`${result.artist} — ${result.title}`);
  console.log(result.apple_music.url);
}
```

With the SDK ([github](https://github.com/AudDMusic/audd-node), [docs](https://docs.audd.io/sdks/node)):

```typescript
const song = await audd.recognize("./audio.mp3", {
  return: ["apple_music", "spotify"],
});
if (song) console.log(`${song.artist} — ${song.title}`, song.appleMusic?.url);
```

</details>

<details>
<summary><strong>Go</strong></summary>

HTTP, no SDK:

```go
import (
    "bytes"
    "io"
    "mime/multipart"
    "net/http"
    "os"
)

f, _ := os.Open("audio.mp3")
defer f.Close()

var buf bytes.Buffer
w := multipart.NewWriter(&buf)
fw, _ := w.CreateFormFile("file", "audio.mp3")
io.Copy(fw, f)
w.WriteField("return", "apple_music,spotify")
w.WriteField("api_token", "your-api-token")
w.Close()

resp, _ := http.Post("https://api.audd.io/", w.FormDataContentType(), &buf)
defer resp.Body.Close()
// parse resp.Body as JSON, same shape as the URL example
```

With the SDK ([github](https://github.com/AudDMusic/audd-go), [docs](https://docs.audd.io/sdks/go)):

```go
result, _ := client.Recognize("/path/to/audio.mp3", &audd.RecognizeOptions{
    ReturnMetadata: "apple_music,spotify",
})
fmt.Printf("%s — %s\n%s\n", result.Artist, result.Title, result.AppleMusic.URL)
```

</details>

<details>
<summary><strong>Rust</strong></summary>

HTTP, no SDK (`reqwest` multipart):

```rust
let part = reqwest::multipart::Part::file("audio.mp3").await?;
let form = reqwest::multipart::Form::new()
    .part("file", part)
    .text("return", "apple_music,spotify")
    .text("api_token", "your-api-token");

let r: serde_json::Value = reqwest::Client::new()
    .post("https://api.audd.io/")
    .multipart(form)
    .send().await?
    .json().await?;
```

With the SDK ([github](https://github.com/AudDMusic/audd-rust), [docs](https://docs.audd.io/sdks/rust)):

```rust
use audd::RecognizeOptions;

let return_metadata = ["apple_music".into(), "spotify".into()];
if let Some(r) = audd
    .recognize_with(
        "audio.mp3",
        RecognizeOptions { return_metadata: Some(&return_metadata), ..Default::default() },
    )
    .await?
{
    println!("{} — {}", r.artist, r.title);
}
```

</details>

<details>
<summary><strong>PHP</strong></summary>

HTTP, no SDK (using cURL):

```php
$ch = curl_init('https://api.audd.io/');
curl_setopt_array($ch, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_POST => true,
    CURLOPT_POSTFIELDS => [
        'file' => new CURLFile('audio.mp3'),
        'return' => 'apple_music,spotify',
        'api_token' => 'your-api-token',
    ],
]);
$r = json_decode(curl_exec($ch), true);
curl_close($ch);

if ($r['status'] === 'success' && $r['result']) {
    echo "{$r['result']['artist']} — {$r['result']['title']}\n";
    echo $r['result']['apple_music']['url'] . "\n";
}
```

With the SDK ([github](https://github.com/AudDMusic/audd-php), [docs](https://docs.audd.io/sdks/php)):

```php
$result = $audd->recognize('audio.mp3', return_metadata: ['apple_music', 'spotify']);
echo "{$result->artist} — {$result->title}\n";
echo $result->apple_music->url, "\n";
```

</details>

<details>
<summary><strong>Swift</strong></summary>

HTTP, no SDK — manual multipart with `URLSession`:

```swift
let boundary = UUID().uuidString
let crlf = "\r\n"
let fileData = try Data(contentsOf: URL(fileURLWithPath: "audio.mp3"))

var body = Data()
body.append("--\(boundary)\(crlf)Content-Disposition: form-data; name=\"file\"; filename=\"audio.mp3\"\(crlf)Content-Type: audio/mpeg\(crlf)\(crlf)".data(using: .utf8)!)
body.append(fileData)
body.append("\(crlf)--\(boundary)\(crlf)Content-Disposition: form-data; name=\"return\"\(crlf)\(crlf)apple_music,spotify\(crlf)--\(boundary)\(crlf)Content-Disposition: form-data; name=\"api_token\"\(crlf)\(crlf)your-api-token\(crlf)--\(boundary)--\(crlf)".data(using: .utf8)!)

var req = URLRequest(url: URL(string: "https://api.audd.io/")!)
req.httpMethod = "POST"
req.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")

let (data, _) = try await URLSession.shared.upload(for: req, from: body)
```

With the SDK ([github](https://github.com/AudDMusic/audd-swift), [docs](https://docs.audd.io/sdks/swift)):

```swift
let file = URL(fileURLWithPath: "audio.mp3")
guard let result = try await audd.recognize(
    .file(file),
    return: ["apple_music", "spotify"]
) else { return }

print("\(result.artist ?? "?") — \(result.title ?? "?")")
print(result.appleMusic?.url ?? "")
```

</details>

<details>
<summary><strong>Kotlin</strong></summary>

HTTP, no SDK (OkHttp):

```kotlin
val body = MultipartBody.Builder()
    .setType(MultipartBody.FORM)
    .addFormDataPart(
        "file", "audio.mp3",
        File("audio.mp3").asRequestBody("audio/mpeg".toMediaType())
    )
    .addFormDataPart("return", "apple_music,spotify")
    .addFormDataPart("api_token", "your-api-token")
    .build()

val response = OkHttpClient().newCall(
    Request.Builder().url("https://api.audd.io/").post(body).build()
).execute()
println(response.body?.string())
```

With the SDK ([github](https://github.com/AudDMusic/audd-kotlin), [docs](https://docs.audd.io/sdks/kotlin)):

```kotlin
import io.audd.Source

val result = audd.recognize(
    Source.FilePath(File("audio.mp3")),
    returnExtras = listOf("apple_music", "spotify"),
)
result?.let { println("${it.artist} — ${it.title} ${it.appleMusic?.get("url")}") }
```

</details>

<details>
<summary><strong>C# / .NET</strong></summary>

HTTP, no SDK:

```csharp
using var http = new HttpClient();
using var form = new MultipartFormDataContent();
form.Add(new StreamContent(File.OpenRead("audio.mp3")), "file", "audio.mp3");
form.Add(new StringContent("apple_music,spotify"), "return");
form.Add(new StringContent("your-api-token"), "api_token");

var response = await http.PostAsync("https://api.audd.io/", form);
Console.WriteLine(await response.Content.ReadAsStringAsync());
```

With the SDK ([github](https://github.com/AudDMusic/audd-dotnet), [docs](https://docs.audd.io/sdks/dotnet)):

```csharp
var result = await audd.RecognizeAsync(
    "/path/to/audio.mp3",
    @return: new[] { "apple_music", "spotify" });
Console.WriteLine($"{result?.Artist} — {result?.Title} {result?.AppleMusic?.Url}");
```

</details>

<details>
<summary><strong>Java</strong></summary>

HTTP, no SDK (manual multipart with `java.net.http`):

```java
var boundary = "----" + UUID.randomUUID();
var crlf = "\r\n";
var fileBytes = Files.readAllBytes(Path.of("audio.mp3"));

var head = ("--" + boundary + crlf
    + "Content-Disposition: form-data; name=\"file\"; filename=\"audio.mp3\"" + crlf
    + "Content-Type: audio/mpeg" + crlf + crlf).getBytes();
var tail = (crlf + "--" + boundary + crlf
    + "Content-Disposition: form-data; name=\"return\"" + crlf + crlf
    + "apple_music,spotify" + crlf
    + "--" + boundary + crlf
    + "Content-Disposition: form-data; name=\"api_token\"" + crlf + crlf
    + "your-api-token" + crlf
    + "--" + boundary + "--" + crlf).getBytes();

var body = new ByteArrayOutputStream();
body.write(head); body.write(fileBytes); body.write(tail);

var req = HttpRequest.newBuilder(URI.create("https://api.audd.io/"))
    .header("Content-Type", "multipart/form-data; boundary=" + boundary)
    .POST(BodyPublishers.ofByteArray(body.toByteArray()))
    .build();
System.out.println(HttpClient.newHttpClient().send(req, BodyHandlers.ofString()).body());
```

With the SDK ([github](https://github.com/AudDMusic/audd-java), [docs](https://docs.audd.io/sdks/java)):

```java
import java.nio.file.Path;

var r = audd.recognize(
    Path.of("audio.mp3"),
    RecognizeOptions.builder()
        .returnMetadata("apple_music", "spotify")
        .build()
);
System.out.println(r.artist() + " — " + r.title());
```

</details>

<details>
<summary><strong>C</strong></summary>

HTTP, no SDK (libcurl mime):

```c
CURL *c = curl_easy_init();
curl_easy_setopt(c, CURLOPT_URL, "https://api.audd.io/");

curl_mime *form = curl_mime_init(c);
curl_mimepart *p;
p = curl_mime_addpart(form); curl_mime_name(p, "file"); curl_mime_filedata(p, "audio.mp3");
p = curl_mime_addpart(form); curl_mime_name(p, "return"); curl_mime_data(p, "apple_music,spotify", CURL_ZERO_TERMINATED);
p = curl_mime_addpart(form); curl_mime_name(p, "api_token"); curl_mime_data(p, "your-api-token", CURL_ZERO_TERMINATED);

curl_easy_setopt(c, CURLOPT_MIMEPOST, form);
curl_easy_perform(c);
curl_mime_free(form);
curl_easy_cleanup(c);
```

With the SDK ([github](https://github.com/AudDMusic/audd-c), [docs](https://docs.audd.io/sdks/c)):

```c
const char *want[] = { "apple_music", "spotify", NULL };
audd_recognize_options_t opts = { .return_metadata = want };
audd_recognition_t *r = NULL;
audd_recognize(client, "/path/to/audio.mp3", &opts, &r);
```

</details>

<details>
<summary><strong>C++</strong></summary>

HTTP, no SDK (libcurl mime):

```cpp
CURL *c = curl_easy_init();
curl_easy_setopt(c, CURLOPT_URL, "https://api.audd.io/");

curl_mime *form = curl_mime_init(c);
auto *p = curl_mime_addpart(form);
curl_mime_name(p, "file");
curl_mime_filedata(p, "audio.mp3");

for (auto [name, value] : {
    std::pair{"return", "apple_music,spotify"},
    std::pair{"api_token", "your-api-token"},
}) {
    p = curl_mime_addpart(form);
    curl_mime_name(p, name);
    curl_mime_data(p, value, CURL_ZERO_TERMINATED);
}

curl_easy_setopt(c, CURLOPT_MIMEPOST, form);
curl_easy_perform(c);
curl_mime_free(form);
curl_easy_cleanup(c);
```

With the SDK ([github](https://github.com/AudDMusic/audd-cpp), [docs](https://docs.audd.io/sdks/cpp)):

```cpp
audd::RecognizeOptions opts;
opts.return_metadata = {"apple_music", "spotify"};
auto result = client.recognize("/path/to/audio.mp3", opts);
```

</details>

### Example response

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
    "apple_music": { "url": "https://music.apple.com/...", "previews": [...], "...": "..." },
    "spotify": { "uri": "spotify:track:...", "external_urls": { "spotify": "https://open.spotify.com/..." }, "...": "..." }
  }
}
```

When `result` is `null`, no match was found. The `timecode` field is the position within the *original* song where your clip was playing. The `apple_music` and `spotify` blocks are present only when included in `return=`.

> **WebSocket variant:** Connect to `wss://api.audd.io/ws/?api_token=[token]` and stream multiple files (binary) without waiting for responses — useful for high-throughput pipelines.

---

## Process Long Files (Enterprise)

The enterprise endpoint accepts files of **any length** — hours-long DJ mixes, full radio recordings, video files — and returns every recognized track with timestamps.

Requests are counted as **1 per 12 seconds of audio**. Use `skip` and `every` to control cost.

**Parameters:**

| Parameter | Required | Description |
|---|---|---|
| `api_token` | ✅ | Your auth token |
| `url` | one of `url` / `file` | URL of the file (also accepts web pages containing audio/video) |
| `file` | one of `url` / `file` | File via `multipart/form-data` |
| `accurate_offsets` | | `"true"` for precise start/end offsets |
| `skip` | | Number of 12s chunks to skip after each scanned chunk |
| `every` | | Number of consecutive chunks to scan |
| `skip_first_seconds` | | Skip N seconds at the beginning |
| `limit` | | Max number of matches per chunk (recommended: `1` for cost control) |

> **Cost example:** `skip=4&every=1` scans 12s then skips 48s → 1 request per minute of audio. `skip=9&every=1` → 1 request per 2 minutes.

<details>
<summary><strong>cURL</strong></summary>

```bash
curl https://enterprise.audd.io/ \
  -F url='https://audd.tech/djatwork_example.mp3' \
  -F accurate_offsets='true' \
  -F limit='1' \
  -F api_token='your-api-token'
```

</details>

<details>
<summary><strong>Python</strong></summary>

```python
r = requests.post("https://enterprise.audd.io/", data={
    "url": "https://audd.tech/djatwork_example.mp3",
    "accurate_offsets": "true",
    "limit": "1",
    "api_token": "your-api-token",
}).json()

for chunk in r.get("result", []):
    for song in chunk["songs"]:
        print(f"[{chunk['offset']}] {song['artist']} — {song['title']} (score: {song['score']})")
```

With the SDK:

```python
for match in audd.recognize_enterprise(
    "https://audd.tech/djatwork_example.mp3",
    accurate_offsets=True, limit=1,
):
    print(f"[{match.offset}] {match.artist} — {match.title} ({match.score})")
```

</details>

<details>
<summary><strong>Node / TypeScript</strong></summary>

```typescript
const r = await fetch("https://enterprise.audd.io/", {
  method: "POST",
  body: new URLSearchParams({
    url: "https://audd.tech/djatwork_example.mp3",
    accurate_offsets: "true",
    limit: "1",
    api_token: "your-api-token",
  }),
}).then(r => r.json());

for (const chunk of r.result ?? []) {
  for (const s of chunk.songs) {
    console.log(`[${chunk.offset}] ${s.artist} — ${s.title} (${s.score})`);
  }
}
```

With the SDK:

```typescript
const matches = await audd.recognizeEnterprise(
  "https://audd.tech/djatwork_example.mp3",
  { accurateOffsets: true, limit: 1 },
);
for (const m of matches) {
  console.log(`[${m.offset}] ${m.artist} — ${m.title}`);
}
```

</details>

Every SDK exposes the same enterprise recognition under each language's naming conventions (`recognize_enterprise`, `recognizeEnterprise`, `RecognizeEnterprise`, etc.) — see the [per-language docs](https://docs.audd.io/sdks/).

### Response shape

| Field | Meaning |
|---|---|
| `offset` | Position in *your file* where the 12s chunk starts (e.g. `"04:48"`) |
| `songs[].timecode` | Position in the *original song* being played |
| `songs[].score` | Confidence score (0–100) |
| `songs[].start_offset` / `end_offset` | Millisecond positions within the 12s chunk |
| `songs[].isrc`, `upc` | Track / album identifiers (Startup plan and above) |

---

## Monitor Live Streams

Monitor radio stations, Twitch broadcasts, YouTube live streams, and any audio stream in real time. You provide stream URLs, AudD monitors them continuously, and delivers recognition results via webhook callbacks or longpoll.

**Pricing:** $45/stream/month with AudD's catalog, or $25/stream/month with your own catalog only.

### Setup

**1. Set your callback URL:**

```bash
curl https://api.audd.io/setCallbackUrl/ \
  -F url='https://yourserver.com/audd-webhook' \
  -F api_token='your-api-token'
```

If you don't have a server, use `https://audd.tech/empty/` as the callback and pull results via longpoll instead.

**2. Add streams:**

```bash
# Radio / Icecast / HLS / DASH
curl https://api.audd.io/addStream/ \
  -F url='https://npr-ice.streamguys1.com/live.mp3' \
  -F radio_id='3249' \
  -F api_token='your-api-token'

# Twitch channel
curl https://api.audd.io/addStream/ \
  -F url='twitch:monstercat' \
  -F radio_id='5513' \
  -F api_token='your-api-token'

# YouTube live (video or channel)
curl https://api.audd.io/addStream/ \
  -F url='youtube:5qap5aO4i9A' \
  -F api_token='your-api-token'
```

**3. Receive results at your callback URL:**

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

By default callbacks fire when a song *finishes* playing. Pass `callbacks=before` on `addStream` to fire when a song *starts* instead.

### Stream URL formats

| Platform | Format |
|---|---|
| Radio / Icecast / HLS / DASH | `https://stream-url.com/live.mp3` |
| Twitch | `twitch:channelname` |
| YouTube (video) | `youtube:videoId` |
| YouTube (channel) | `youtube-ch:channelId` |

### Stream management

| Endpoint | Purpose |
|---|---|
| `POST /getStreams/` | List active streams |
| `POST /setStreamUrl/` | Update a stream's URL |
| `POST /deleteStream/` | Remove a stream |
| `POST /getCallbackUrl/` | Read current callback URL |

### Longpoll & widget

As an alternative to callbacks, pull results via longpoll:

```
https://api.audd.io/longpoll/?category=[longpoll_category]&timeout=50&since_time=[timestamp]
```

Get `longpoll_category` from `/getStreams/`. Or embed a live now-playing widget:

```
https://widget.audd.tech/?ch=-[longpoll_category]&background&history&shadow
```

Every official SDK exposes a longpoll consumer that handles the timestamp bookkeeping for you — see the [per-SDK docs](https://docs.audd.io/sdks/).

---

## Custom Catalog

Fingerprint your own tracks so future recognition calls match against them. Useful for proprietary catalogs, sound effects, voice samples, or audio you can't share with general-purpose music databases.

Access is granted per-account — email [api@audd.io](mailto:api@audd.io) to request it. Once enabled, every SDK has a `custom_catalog.add()` (or equivalent) method for uploading tracks.

---

## Common Errors

| Code | Description |
|---|---|
| `#901` | No API token, or free quota reached — [get a token](https://dashboard.audd.io/) |
| `#900` | Invalid API token |
| `#700` | No file received — check `Content-Type: multipart/form-data` and use `https://` URLs |
| `#600` | Couldn't download the audio URL |
| `#500` | Invalid audio file format |
| `#400` | File too large for `api.audd.io/` (max 10MB / 25s) — use [enterprise](#process-long-files-enterprise) |
| `#300` | Fingerprinting error — clip is likely too short |

Full error catalog at [docs.audd.io](https://docs.audd.io/).

---

## Tips

- **Audio length:** the standard endpoint works best with 5–12 seconds of audio.
- **Provider links:** specify `return=apple_music,spotify,deezer,napster,musicbrainz` to attach those services' track URLs and IDs to the response.
- **Enterprise cost:** use `skip` and `every` to sample long files instead of scanning every chunk; set `limit=1` if you only need one match per chunk.

---

## Resources

- [**API dashboard**](https://dashboard.audd.io/) — get your token, manage billing, configure streams
- [**Full documentation**](https://docs.audd.io/) — complete API reference
- [**audd-chrome-extension**](https://github.com/AudDMusic/audd-chrome-extension) — recognize music in any browser tab
- [**RedditBot**](https://github.com/AudDMusic/RedditBot) — music recognition bot for Reddit
- [**DiscordBot**](https://github.com/AudDMusic/DiscordBot) — identify music in Discord channels (Go)
- [Code examples on GitHub](https://github.com/search?q=%22api.audd.io%22&type=code) — community uses

---

## Support

- **API tokens & billing:** [dashboard.audd.io](https://dashboard.audd.io)
- **Documentation:** [docs.audd.io](https://docs.audd.io)
- **Email:** [api@audd.io](mailto:api@audd.io)
- **Custom terms** — >500k requests/month, >100 simultaneous streams, or enterprise account for ISRC/UPC metadata: [api@audd.io](mailto:api@audd.io)
