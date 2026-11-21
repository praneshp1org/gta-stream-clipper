# GTA Stream Clipper

A Google Colab pipeline that turns long Twitch or YouTube stream VODs into short, vertical, YouTube Shorts-ready clips.

The workflow uses audio analysis and transcript scoring to find energetic moments, then renders only the best segments with a 9:16 layout and burned-in captions.

## Features

- Audio-only analysis for faster first-pass processing
- Hype detection using loudness and pitch
- Optional YAMNet laughter, shout, scream, and cheer detection
- Optional Twitch chat activity detection
- Whisper transcription for the highest-scoring moments only
- Automatic or LLM-assisted clip ranking and titles
- Manual review mode
- Split, blurred-background, or cropped 9:16 layouts
- Word-level burned-in captions
- GPU rendering with NVENC or CPU fallback
- Loudness-normalized MP4 exports and metadata JSON

## Requirements

- Google Colab with a T4 GPU runtime recommended
- A public Twitch or YouTube VOD URL, or a local video upload
- An optional Gemini or Anthropic API key for improved ranking and titles

## Quick Start

1. Open `stream_clipper_v2.ipynb` in Google Colab.
2. Select **Runtime → Change runtime type → T4 GPU**.
3. Optionally add a secret in **Colab → Secrets**:
   - `GEMINI_API_KEY`, or
   - `ANTHROPIC_API_KEY`
4. Run every cell from top to bottom.
5. Set `VIDEO_URL` to a Twitch or YouTube VOD URL.
6. Adjust the settings in the **Settings** cell.
7. Download `clips.zip` from the final cell.

The generated `clips/` folder includes the rendered MP4 files and `metadata.json` with titles, hashtags, source timestamps, and scores.

## Local Video Upload

Set:

```python
USE_UPLOAD = True
```

Then run the notebook and upload a video from the file picker. Local uploads disable Twitch chat replay.

## Main Settings

| Setting | Purpose |
|---|---|
| `VIDEO_URL` | Source Twitch or YouTube VOD |
| `USE_UPLOAD` | Upload a local video instead of downloading |
| `NUM_CLIPS` | Number of final clips to render |
| `N_CANDIDATES` | Number of candidate moments analyzed by Whisper |
| `MIN_LEN`, `MAX_LEN` | Minimum and maximum clip length in seconds |
| `HOOK_PAD`, `TAIL` | Seconds added before the rise and after the peak |
| `MIN_GAP` | Minimum separation between candidate peaks |
| `LAYOUT` | `split`, `blur`, or `crop` |
| `FACECAM` | Face-cam position and size as frame fractions |
| `CAPTIONS` | Add burned-in word captions |
| `USE_CHAT` | Use Twitch chat activity spikes |
| `USE_YAMNET` | Detect laughter, shouting, screaming, and cheering |
| `LLM` | Set to `gemini`, `anthropic`, or `none` |
| `REVIEW` | Enable manual selection from the ranked shortlist |

## Layout Options

- **`split`** — gameplay at the top and face cam at the bottom
- **`blur`** — blurred full-screen background with the original video centered
- **`crop`** — direct vertical center crop

For a custom face-cam crop, update `FACECAM` using normalized `(x, y, width, height)` values.

## Output

Each rendered clip is exported as a vertical MP4 with:

- 1080 × 1920 output
- AAC audio
- Loudness normalization
- Optional ASS word captions
- Fast-start metadata

`clips/metadata.json` contains export details for all successful clips.

## Troubleshooting

- **YouTube download is blocked:** use `USE_UPLOAD = True` or download the VOD on a local machine first.
- **Clips feel slow or miss the best moments:** increase `N_CANDIDATES` or adjust `WEIGHTS`.
- **Whisper is slow:** use `WHISPER_MODEL = "base"`.
- **Face cam is cropped incorrectly:** adjust `FACECAM` based on a screenshot of the source frame.
- **GPU encoding is unavailable:** the notebook automatically falls back to `libx264`.
- **No API key configured:** ranking continues locally using transcript and audio signals.

## Tuning Suggestions

- FPS or competitive streams: increase the `loud` weight.
- Comedy or reaction streams: increase the `event` weight.
- Streams with active chat: keep `USE_CHAT=True` for Twitch VOD URLs.
- For more accurate speech boundaries: use the `medium` or `large-v3` Whisper model if runtime allows.
