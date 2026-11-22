# Automated Video Maker

Automated Video Maker turns scripts or topics into narrated videos with images, subtitles, and FFmpeg rendering. It supports documentary narration and multi-presenter discussion videos in vertical and landscape formats.

## Features

- Generate a script from a topic or provide your own.
- Create documentary narration or multi-presenter discussions.
- Produce vertical 9:16 Shorts or landscape 16:9 videos.
- Convert text to speech locally with Piper.
- Plan scenes with Groq AI or use rule-based scene splitting.
- Search for images and use fallback images when search fails.
- Generate subtitles and render videos with FFmpeg.
- Follow generation progress and logs in the browser.

## Project structure

```
.
├── backend/    # API, WebSocket server, and video generation pipeline
└── frontend/   # Next.js user interface
```

Each app has its own `README.md` with app-specific setup instructions.

## Requirements

- Node.js 20 or newer
- pnpm (recommended) or npm
- FFmpeg and `ffprobe` available on your PATH
- Piper executable and compatible voice models
- Groq API key for topic scripts and AI features
- Internet access for Groq and image search

Piper speech synthesis and FFmpeg rendering run locally. Groq and image search use external services, so the complete application is not fully offline.

## Installation

Install dependencies in both applications:

```
cd backend
pnpm install

cd ../frontend
pnpm install
```

Install FFmpeg and `ffprobe` using your operating system’s package manager. Install Piper for your operating system and place compatible `.onnx` voice models in `backend/models/`.

Voice model IDs must match the configured filenames. The default presenter voice IDs are:

- `en_US-ryan-high`
- `en_US-lessac-high`
- `en_US-joe-medium`
- `en_US-amy-high`

## Configuration

Create `backend/.env`:

```
PORT=3002
GROQ_API_KEY=your_groq_api_key
PIPER_BIN=/usr/local/bin/piper
PIPER_LIB_PATH=
USE_AI_SCENE_PLANNING=true
SCENES_PER_GROUP=2

VOICE_STEVE=en_US-ryan-high
VOICE_STORM=en_US-lessac-high
VOICE_STONE=en_US-joe-medium
VOICE_ATLAS=en_US-amy-high
```

If the backend is hosted at a different browser-accessible URL, create `frontend/.env.local`:

```
NEXT_PUBLIC_API_URL=http://localhost:3002
```

Use the backend’s HTTPS URL when the frontend is served over HTTPS. The frontend selects secure WebSockets automatically for HTTPS URLs. Restart the frontend after changing this setting.

Do not commit `.env` files, API keys, voice models, generated videos, caches, or temporary files.

## Run locally

Start the backend in one terminal:

```
cd backend
pnpm dev
```

Start the frontend in another terminal:

```
cd frontend
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000). The backend listens on port `3002` by default.

To build and start the backend for production:

```
cd backend
pnpm build
pnpm start
```

To build and start the frontend for production:

```
cd frontend
pnpm build
pnpm start
```

## Use the app

1. Choose **Script Mode** to enter narration, or **Topic Mode** to generate a draft script.
2. Choose **Documentary** for a single narrator or **Discussion Panel** for multiple presenters.
3. Choose **Shorts** for vertical video or **Long-form** for landscape video.
4. Start generation and keep the browser connection open while progress is reported.
5. Preview and download the finished video.

Topic duration is a script-generation target, not a guarantee of final video runtime.

## How generation works

1. The frontend sends the script, format, and video type to the backend over WebSocket.
2. Discussion scripts are parsed or converted into speaker-tagged dialogue. Documentary scripts use one narrator.
3. Piper generates speech audio locally.
4. The backend plans or splits scenes and searches for images.
5. The backend generates subtitles and uses FFmpeg to render the video.
6. The finished video is served from the backend’s `/output` route.

Generation currently runs over the connected WebSocket session. If the browser disconnects, it may no longer receive progress updates.

## Technology

- **Frontend:** Next.js, React, TypeScript, Tailwind CSS
- **Backend:** Node.js, Express, TypeScript, WebSocket
- **Speech synthesis:** Piper TTS
- **AI services:** Groq
- **Image search:** DuckDuckGo image search
- **Video rendering:** FFmpeg

## Troubleshooting

- **Port already in use:** Stop the process using port `3000` or `3002`. If you change the backend port, update the frontend API URL and the rewrite in `frontend/next.config.js`.
- **Frontend cannot connect:** Check that the backend is running, `NEXT_PUBLIC_API_URL` is reachable from the browser, and HTTP/HTTPS matches WS/WSS.
- **Topic generation or AI scene planning fails:** Check `GROQ_API_KEY` and network access. Set `USE_AI_SCENE_PLANNING=false` to use rule-based scene splitting.
- **Piper model not found:** Check that the model exists in `backend/models/` and that its filename matches the configured voice ID.
- **Piper executable or library errors:** Check `PIPER_BIN` and, if required, `PIPER_LIB_PATH`.
- **FFmpeg errors:** Confirm that both `ffmpeg` and `ffprobe` are installed and available to the backend process.
- **Images are generic or unavailable:** Results depend on search queries, internet access, and provider availability. The pipeline may reuse a cached image or use a placeholder.
- **Rendering is slow or uses too much memory:** Use shorter scripts or reduce the number of generated scenes with `SCENES_PER_GROUP`.

## Contributing

Contributions are welcome.

1. Fork the repository and create a focused branch.
2. Keep frontend changes in `frontend/` and backend changes in `backend/`.
3. When changing an API route or WebSocket message, update both apps and document the change.
4. Run `pnpm build` in both app directories before opening a pull request.
5. Describe what changed and how you checked it. Include screenshots for UI changes when useful.

Do not commit credentials, `.env` files, Piper models, generated videos, cached downloads, or temporary files. For bug reports, include reproduction steps, operating system, Node.js version, and relevant logs with secrets removed.

## Security and privacy

Topic prompts and AI features send data to Groq. Image search queries are sent to the image search provider. Speech synthesis and video rendering run locally.

The backend generation WebSocket does not currently require authentication. Do not expose the backend publicly without adding authentication, request limits, and appropriate network controls.

## License and third-party services

Piper, Groq, FFmpeg, and image search are third-party tools or services with their own terms and availability. Review image licensing before publishing videos that use search results.
