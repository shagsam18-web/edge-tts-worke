# edge-tts-worker

Deploy Microsoft Edge TTS as an OpenAI-compatible API on your own Cloudflare account. Free.

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/sofish/edge-tts-worker)

## Deploy

```bash
npm install
npx wrangler deploy
```

Optionally set an API key to protect your endpoint:

```bash
npx wrangler secret put API_KEY
```

## API

### `POST /v1/audio/speech`

OpenAI-compatible text-to-speech endpoint.

```bash
curl -X POST https://your-worker.workers.dev/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{
    "model": "tts-1",
    "input": "Hello, world!",
    "voice": "en-US-EmmaMultilingualNeural",
    "speed": 1.0,
    "response_format": "mp3"
  }' \
  --output speech.mp3
```

| Parameter | Description | Default |
|---|---|---|
| `model` | Ignored (accepted for compatibility) | — |
| `input` | Text to synthesize (required) | — |
| `voice` | Edge TTS voice name (auto-detected if omitted) | auto |
| `gender` | `"female"` or `"male"` (used when `voice` is omitted) | `female` |
| `speed` | Speech rate, 0.5–2.0 | `1.0` |
| `response_format` | Only `mp3` supported | `mp3` |

If `voice` is omitted, it auto-detects the language from the input text:

| Language | Female | Male |
|---|---|---|
| Chinese | `zh-CN-XiaoxiaoNeural` | `zh-CN-YunxiNeural` |
| Japanese | `ja-JP-NanamiNeural` | `ja-JP-KeitaNeural` |
| Korean | `ko-KR-SunHiNeural` | `ko-KR-InJoonNeural` |
| English / other | `en-US-EmmaMultilingualNeural` | `en-US-AndrewMultilingualNeural` |

If `API_KEY` is set, include `Authorization: Bearer <key>` header.

### `GET /v1/ws`

WebSocket endpoint for reading apps. Keeps a persistent connection for sequential TTS requests.

```
wss://your-worker.workers.dev/v1/ws
```

If `API_KEY` is set, pass it as a query parameter: `wss://your-worker.workers.dev/v1/ws?token=<key>`

**Send** (JSON text message):

```json
{"input": "Text to read", "voice": "en-US-EmmaMultilingualNeural", "gender": "female", "speed": 1.0}
```

**Receive**:
- Binary messages — MP3 audio chunks, stream directly to your audio player
- `{"event": "done"}` — synthesis complete, safe to send next text
- `{"event": "error", "message": "..."}` — error occurred

**Reading app flow**: connect once → send paragraph → receive audio chunks → play → on `done`, send next paragraph → repeat.

### `GET /v1/voices`

Returns the list of available voices as JSON.

```bash
curl https://your-worker.workers.dev/v1/voices
```

### `GET /`

A minimal HTML test page with text input, voice selector, speed slider, and play button.

## Development

```bash
npm install
npm run dev
```

## Disclaimer

This project uses Microsoft's unofficial Edge TTS endpoint. There is no SLA or uptime guarantee. Use at your own risk.

## License

AGPL-3.0
