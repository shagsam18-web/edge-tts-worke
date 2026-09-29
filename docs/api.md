# API Reference

## Authentication

If the `API_KEY` environment variable is set, all endpoints (except `GET /`) require authentication:
- HTTP endpoints: `Authorization: Bearer <key>` header
- WebSocket: `?token=<key>` query parameter

## Endpoints

### `POST /v1/audio/speech`

Synthesize text to speech. Returns streaming `audio/mpeg` data.

**Request body** (JSON):

```json
{
  "model": "tts-1",
  "input": "Text to synthesize",
  "voice": "en-US-EmmaMultilingualNeural",
  "speed": 1.0,
  "response_format": "mp3"
}
```

- `input` (required): The text to synthesize.
- `voice` (optional): Edge TTS voice short name. If omitted, auto-detected from text using `gender`. Get the full list from `GET /v1/voices`.
- `gender` (optional): `"female"` or `"male"`. Only used when `voice` is omitted. Default: `"female"`. Auto-detected voices:

  | Language | Female | Male |
  |---|---|---|
  | Chinese | `zh-CN-XiaoxiaoNeural` | `zh-CN-YunxiNeural` |
  | Japanese | `ja-JP-NanamiNeural` | `ja-JP-KeitaNeural` |
  | Korean | `ko-KR-SunHiNeural` | `ko-KR-InJoonNeural` |
  | English / other | `en-US-EmmaMultilingualNeural` | `en-US-AndrewMultilingualNeural` |

- `speed` (optional): Float between 0.5 and 2.0. Default: `1.0`. Converted internally to Edge TTS rate format (e.g. 1.5 becomes `+50%`).
- `model` (optional): Accepted for OpenAI compatibility but ignored.
- `response_format` (optional): Only `mp3` is supported.

**Response**: Streaming `audio/mpeg` body.

**Errors**: Returns JSON in OpenAI error format:

```json
{
  "error": {
    "message": "description",
    "type": "invalid_request_error",
    "code": 400
  }
}
```

### `GET /v1/voices`

Returns the full list of available voices from Microsoft Edge TTS.

**Response**: JSON array of voice objects:

```json
[
  {
    "Name": "Microsoft Server Speech Text to Speech Voice (en-US, EmmaMultilingualNeural)",
    "ShortName": "en-US-EmmaMultilingualNeural",
    "Gender": "Female",
    "Locale": "en-US",
    "FriendlyName": "Microsoft Emma Online (Natural) - English (United States)",
    "VoiceTag": {
      "ContentCategories": ["General"],
      "VoicePersonalities": ["Friendly", "Positive"]
    }
  }
]
```

The voice list is cached in memory after the first request.

### `GET /v1/ws`

WebSocket endpoint for persistent connections (e.g. reading apps).

**Connect**:

```
wss://your-worker.workers.dev/v1/ws
wss://your-worker.workers.dev/v1/ws?token=YOUR_API_KEY  # with auth
```

**Send** (JSON text message):

```json
{
  "input": "Text to synthesize",
  "voice": "en-US-EmmaMultilingualNeural",
  "speed": 1.0
}
```

- `input` (required): The text to synthesize.
- `voice` (optional): Auto-detected from text if omitted.
- `gender` (optional): `"female"` or `"male"`. Used when `voice` is omitted. Default: `"female"`.
- `speed` (optional): Float between 0.5 and 2.0. Default: `1.0`.

**Receive**:

- **Binary messages**: MP3 audio chunks. Stream directly to audio player.
- **Text message** `{"event": "done"}`: Synthesis complete for the current request. Send the next text.
- **Text message** `{"event": "error", "message": "..."}`: An error occurred.

**Usage pattern for reading apps**:

1. Connect once via WebSocket
2. Send a paragraph/sentence as JSON
3. Receive binary audio chunks, pipe to audio player
4. On `{"event": "done"}`, send the next paragraph
5. Repeat until finished, then close the connection

The connection stays open between requests — no reconnection overhead per paragraph.

### `GET /`

Serves a minimal HTML test page for trying out TTS in the browser.

### `OPTIONS *`

Returns CORS preflight headers. All origins are allowed (`*`).
