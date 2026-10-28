# AL.AI / Blu Visual Teacher

This folder combines the frontend with the Express backend from the uploaded ZIP.

## Run locally

1. Open a terminal in `backend/`.
2. Run `npm ci` once to install the dependencies listed in the lock file.
3. Run `npm start`.
4. Open `http://127.0.0.1:3000` in a browser to see the welcome page. Click **Let’s learn** to open the lesson app at `/lesson`.

Use Node.js 18 or newer. The server only listens on `127.0.0.1`.

The Express app serves `backend/public/welcome.html` at `/` and `backend/public/index.html` at `GET /lesson`. The lesson page posts to the same path using `POST /lesson`; the backend maps methods separately. The frontend translates the API's `schema_version: "1.0"` scenes into its visual lesson format and keeps the ten-question quiz in the browser. The lesson workspace has Learn, Tutor, and Notes & slides sections. Play opens a focused browser player at `/lesson?player=1`, where video playback and export controls live. Use the AI.AL logo in the lesson header to return to the welcome page.

Tutor sends lesson questions to the local Ollama model at `/api/chat`. Notes and PowerPoint files are created by `/api/study-materials` from the current lesson. If Ollama is unavailable or rejects a lesson, the app labels its localized study-guide fallback instead of presenting an unrelated preset as topic-specific content.

The backend accepts `en`, `hi`, `ta`, `te`, `kn`, `bn`, and `ml` (English, Hindi, Tamil, Telugu, Kannada, Bengali, and Malayalam), matching the language selector. It validates lessons against `backend/schemas/lesson-v1.schema.json` and loads a localized fallback preset for each language. The backend response does not contain quiz data, so the frontend continues to provide its ten-question quiz.

## Local lesson planner

The server tries the local Ollama model `qwen2.5:1.5b` by default for faster lesson generation on modest hardware. It sends the lesson schema to Ollama as a JSON constraint, validates the response again with Ajv, retries drafts with repeated narration, and uses a localized study-guide fallback if Ollama is stopped, missing the model, times out, or returns invalid data. Generated content is treated as data and is never executed.

Install Ollama and download a small model once while connected to the internet, then run Ollama locally:

```sh
ollama pull qwen2.5:1.5b
```

The app makes requests only to `127.0.0.1:11434`; it does not call an external AI API. `qwen2.5:1.5b` and `qwen2.5:3b` are installed in the current development environment. Set `LOCAL_MODEL` to use another model, or set `LOCAL_MODEL_TIMEOUT_MS` to adjust the default 90-second limit. The app remains usable with the localized study-guide fallback if Ollama is unavailable.

## Narration, captions, and video

- **Listen** uses a local Hugging Face MMS VITS voice for the lesson language. In Settings, choose a language, search Hugging Face, then download its voice. One voice is about 145 MB and runs on CPU; install the speech runtime once from the `vidyaai/` project directory with `powershell -ExecutionPolicy Bypass -File backend/install-tts.ps1`. The seven supported voices are English, Hindi, Tamil, Telugu, Kannada, Bengali, and Malayalam. The official MMS checkpoints are licensed CC BY-NC 4.0, so review the [model card](https://huggingface.co/facebook/mms-tts-hin) before commercial use.
- **Captions SRT** downloads the current lesson captions with timings based on the selected lesson length.
- **Export video** draws and animates each diagram on a 1280×720 canvas, synchronizes the selected-language audio, and downloads MP4 only. Browsers that record WebM use the local FFmpeg H.264/AAC converter. If narration, the recorder, or FFmpeg is unavailable, the app reports the missing requirement instead of downloading a WebM/SRT substitute.
- The system scan checks Ollama, installed MMS voices, the local Python speech runtime, FFmpeg, and the browser runtime.
- **Local model manager** searches Hugging Face by model name and shows repository results. It lets you install a selected model when that repository includes a GGUF text model compatible with Ollama; it shows file size and current RAM/disk availability before a download. Other Hugging Face formats remain searchable but cannot be installed by this Ollama-based loader.
- For conversion, install FFmpeg with H.264 (`libx264`) and AAC encoder support. If it is not on a standard path, set `FFMPEG_PATH` to its full path before starting the server.

The current Windows development environment includes FFmpeg on `PATH`. The speech model weights are downloaded separately per language from Settings and are not committed to the project.

## Offline limits and submission checklist

The app code does not send lesson content to cloud AI or translation services. Its current HTML still loads Tailwind, Lucide icons, and fonts from CDNs, so the first page load is **not fully offline**. Vendor those assets locally before claiming an offline demonstration. The project does not yet include model/voice license records, laptop performance measurements, or two submitted sample MP4s; add those to this README and `outputs/` after testing with the final local models and voices.

Dependencies are not bundled. Use `npm ci` from `backend/` before starting the server.
