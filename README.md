# RealNews Listener 100 percent paywall buster 

**Information, Unlocked.** A GPU-accelerated, fully offline-voiced RSS news reader for Windows.

RealNews Listener pulls headlines from 30+ news feeds across the political spectrum — left, center, and right, side by side — and reads them aloud in studio-quality neural voices rendered locally on your own GPU. No subscriptions. No accounts. No cloud TTS. No tracking. Your news, your hardware, your ears.

A product of **N2NHU Labs**. Free software under the **GNU GPL v3**.

---

## Highlights

- **Hear the whole spectrum.** Feeds are organized into Left / Center / Right categories so you can compare coverage of the same story across outlets — or binge just one side, your call.
- **Neural text-to-speech, 100% local.** Powered by the [Kokoro](https://github.com/thewh1teagle/kokoro-onnx) ONNX TTS model running on your NVIDIA GPU via CUDA (with automatic CPU fallback). Eight voices included, from clear newsroom reads to British accents.
- **Instant playback.** The moment a fetch completes, every story is rendered to WAV in the background by a GPU worker pool. By the time you pick a story, the audio is already sitting on disk — playback is instantaneous.
- **Your personal radio archive.** Rendered audio is organized on disk as `NEWS/<FeedName>/<story>.wav` — replay it, copy it to your phone, or pipe it anywhere.
- **Built-in feed discovery.** Type a site name ("politico") or URL into the Config tab and the app scans the site for advertised RSS/Atom feeds, probes common feed paths, validates every candidate, and lets you checkmark the winners straight into your feed list.
- **Zero hardcoding.** Every setting — feeds, voices, speed, theme, worker counts, timeouts, folder names — lives in a plain-text `config.ini` you can edit with Notepad.
- **Single native binary.** Compiled from Python to C with Nuitka. One `.exe`, no Python installation required.

---

## Quick Start

1. Download and unzip the release anywhere (e.g. `C:\newsexe`).
2. Double-click **`RealNewsListener.exe`**.
3. On the **Fetch News** tab, leave the feeds checked (or pick your favorites) and click **Fetch Selected Feeds**.
4. The app jumps to the **Listen** tab and immediately begins rendering audio on your GPU — watch the status bar count up and the `...` markers flip to `OK` beside each story.
5. Select a story, a whole feed, or **ALL stories**, and press **Play Selected**. Done.

### What's in the box

| File | Purpose | Required |
|---|---|---|
| `RealNewsListener.exe` | The application (self-contained native binary) | Yes |
| `config.ini` | All settings and your feed list — edit freely | Yes |
| `kokoro-v1.0.onnx` | The Kokoro neural TTS model (~310 MB) | Yes, for voice |
| `voices-v1.0.bin` | Voice embeddings for all 8 voices (~27 MB) | Yes, for voice |
| `realnews-listener.png` | The banner artwork (auto-scales to your window) | No |
| `kokoro-quant.onnx` | Placeholder for a future quantized model | No (unused) |
| `NEWS\` | Auto-created; holds rendered WAVs, wiped on each re-render | Auto |
| `news_output.txt` | Auto-created; plain-text digest of the last fetch | Auto |

The four required files must sit **in the same folder**. If the two model files are missing, the app still works as a text news reader — you just won't get audio.

### System requirements

- Windows 10/11, 64-bit
- ~1 GB disk space for the app + models, plus a little for rendered WAVs
- Internet connection (for fetching feeds — TTS itself is fully offline)
- **Recommended:** an NVIDIA GPU with current drivers for fast audio rendering. Without one, rendering falls back to CPU — everything still works, it just renders slower.

---

## The Three Tabs

### 1. Fetch News

Your feed picker. Feeds are grouped by category (Left / Center / Right). Check the ones you want, then:

- **Fetch Selected Feeds** — downloads all checked feeds in parallel, parses RSS 2.0 / RSS 1.0 / Atom, and builds the story list. Feeds that error out are logged to the console without disturbing the rest.
- **Select All / Select None** — bulk toggles.
- **Save Output As...** — exports the plain-text digest of the current fetch to a file of your choosing.

The number of stories pulled per feed is controlled by `default_max_items` in `config.ini`.

### 2. Listen

The heart of the app. Stories appear grouped by feed, each with a status marker:

| Marker | Meaning |
|---|---|
| `...` | Audio still rendering |
| `OK` | WAV ready — instant playback |
| `X` | Rendering failed for this story (see console) |

**Selecting what to play.** Pick exactly one radio button:

- **ALL stories (sequential)** — the full broadcast: every story from every fetched feed, in order.
- **Play ALL from this feed** — one outlet's entire block, like tuning to a station.
- Any individual story.

**Playback controls.** **Play Selected**, **Pause/Resume**, and **Stop** do what they say. If you hit Play on a story that's still rendering, the app politely waits for it and then starts.

**Voice & Speed.** Choose any of the eight bundled voices from the dropdown; drag the speed slider from 0.6x to 1.6x. **Changing either one automatically re-renders all audio with the new settings** — no extra clicks. (The **Regenerate Audio** button forces a re-render manually if you ever want one.) The status bar always shows which voice and speed the current audio was rendered with.

| Voice | Character |
|---|---|
| `af_bella` | American female — clear, energetic (default) |
| `af_sarah` | American female — warm, even |
| `af_sky` | American female — bright |
| `af_nicole` | American female — soft whisper / ASMR style |
| `am_adam` | American male — deep |
| `am_michael` | American male — neutral |
| `bm_george` | British male |
| `bf_emma` | British female |

**Preview pane.** The bottom panel shows the full text digest of the current fetch.

### 3. Config

- **Dark Mode** — toggles the full dark/light theme instantly; the choice is saved.
- **Find & Add RSS Feeds** — the feed search engine:
  1. Type a site name (`arstechnica`), a domain (`politico.com`), or a full URL.
  2. Click **Search for Feeds**. The app fetches the site, reads its advertised `<link rel="alternate">` feeds, probes common feed paths (`/feed`, `/rss.xml`, etc.), and validates each candidate by actually parsing it.
  3. Working feeds appear with their real titles and live item counts. A Google News topic feed for your search term is always offered as a fallback.
  4. Checkmark the ones you want, pick a category, and hit **ADD CHECKED FEEDS**. They're written to `config.ini` and appear on the Fetch tab immediately.
- **Add Feed Manually** — paste a name and feed URL directly.
- **Remove Feed** — pick any feed from the dropdown and delete it.
- **Save Config** — persists your current voice, speed, theme, and window size as the new defaults.

---

## The Audio Pipeline (how it stays fast)

When a fetch completes, a pool of GPU worker threads (`wav_workers` in config) starts rendering every story to WAV simultaneously with the playback UI being live. Files land in:

```
NEWS\
  CNN_World\
    000_af_bella_100_Markets_digest_bank_earnings.wav
    001_af_bella_100_Still_haven_t_filed_your_taxes.wav
  Mother_Jones\
    012_af_bella_100_Lawmakers_Demand_Answers.wav
  ...
```

The voice and speed are baked into every filename, so audio from an old render can never be confused with — or block — a new one. The `NEWS` folder is wiped and rebuilt on every fetch or re-render; copy out anything you want to keep first.

How much of each story is voiced is set by `wav_text_limit` (characters of headline + summary).

---

## config.ini Reference

Everything is editable with any text editor. Changes are picked up the next time the app starts (and feed changes made in the GUI are written back instantly).

### `[Settings]`

| Key | Default | What it does |
|---|---|---|
| `output_file` | `news_output.txt` | Where the plain-text digest is written |
| `default_max_items` | `6` | Stories pulled per feed |
| `default_voice` | `af_bella` | Voice used for rendering |
| `default_speed` | `1.00` | Speech rate (0.6–1.6) |
| `voices` | (8 voices) | Voices offered in the dropdown |
| `theme` | `dark` | `dark` or `light` |
| `geometry` | `1200x880` | Startup window size |
| `banner_max_height` | `170` | Max banner height in pixels (aspect ratio always preserved) |
| `news_folder` | `NEWS` | Where rendered WAVs are stored |
| `wav_workers` | `2` | Parallel GPU rendering threads |
| `fetch_workers` | `6` | Parallel feed download threads |
| `fetch_timeout` | `15` | Per-feed HTTP timeout, seconds |
| `wav_text_limit` | `900` | Max characters voiced per story |
| `sample_rate` | `24000` | Audio sample rate (matches Kokoro's output) |
| `category_order` | `left, center, right` | Category display order |
| `user_agent` | (browser UA) | HTTP User-Agent sent to feed servers |
| `feed_probe_paths` | `/feed, /rss, ...` | Paths tried by the feed discovery search |

### `[Feeds]`

One line per feed: `Display Name = feed_url|category`

```ini
[Feeds]
BBC World = http://feeds.bbci.co.uk/news/world/rss.xml|center
The Guardian = https://www.theguardian.com/world/rss|left
Fox News World = https://moxie.foxnews.com/google-publisher/world.xml|right
```

Categories are free-form — invent your own (`tech`, `local`, `sports`) and they appear as new sections automatically. Several outlets whose native feeds are unreliable (Reuters, AP) ship preconfigured through Google News proxy feeds, which is a trick you can reuse for any site:

```ini
Anything = https://news.google.com/rss/search?q=site:example.com|center
```

---

## Troubleshooting

**"Kokoro TTS is not available."**
The two model files aren't beside the exe. Confirm `kokoro-v1.0.onnx` (~310 MB) and `voices-v1.0.bin` (~27 MB) are in the same folder as `RealNewsListener.exe`. Exact filenames matter.

**A feed shows an error / loads no stories.**
Feed URLs rot — outlets move or retire them constantly. Check the console window for the exact error (404 = moved, SSL = certificate problems on their end, syntax error = they're serving HTML, not XML). Fastest fix: replace the URL with a Google News proxy feed as shown above, or use Config → feed search to find the outlet's current feed.

**Audio renders slowly.**
Check the console at startup: `Available ONNX providers` should list `CUDAExecutionProvider`. If only `CPUExecutionProvider` appears, update your NVIDIA drivers. You can also raise `wav_workers` to 3–4 on a beefy GPU.

**The voice didn't change.**
It does now — changing the voice dropdown auto-re-renders everything. Watch the status bar: it names the voice being rendered. If statuses show `X`, check the console.

**Stories play but sound clipped.**
Raise `wav_text_limit` in config.ini to voice more of each summary.

**No banner.**
The banner is optional — drop any wide PNG named `realnews-listener.png` (or `banner.png`) beside the exe and it auto-scales.

---

## Building from Source

The app is a single Python file compiled to a native binary with [Nuitka](https://nuitka.net).

```bat
python -m venv venv && venv\Scripts\activate
pip install nuitka kokoro-onnx onnxruntime-gpu soundfile pygame pillow

python -m nuitka --standalone --onefile --enable-plugin=tk-inter --include-package=kokoro_onnx --include-package-data=kokoro_onnx --include-distribution-metadata=kokoro-onnx --include-package-data=onnxruntime --include-package-data=_soundfile_data --include-package-data=language_tags --include-data-files="%VIRTUAL_ENV%\Lib\site-packages\espeakng_loader\espeak-ng.dll=espeakng_loader/espeak-ng.dll" --include-data-dir="%VIRTUAL_ENV%\Lib\site-packages\espeakng_loader\espeak-ng-data=espeakng_loader/espeak-ng-data" --output-filename=RealNewsListener.exe news_reader.py
```

Install `onnxruntime-gpu` (not plain `onnxruntime`) if you want CUDA acceleration in the build. The Kokoro model files are downloaded separately from the [kokoro-onnx releases](https://github.com/thewh1teagle/kokoro-onnx/releases). First compile takes 10–30 minutes; subsequent builds are cached.

> **Release packaging tip:** the binary plus models exceed GitHub's 100 MB per-file repository limit — attach the zip to a **GitHub Release** rather than committing it to the repo.

---

## Privacy

RealNews Listener phones home to exactly nobody. The only network traffic it generates is fetching the RSS feeds you selected and the sites you explicitly search for feeds on. Speech synthesis runs entirely on your machine. There is no telemetry, no analytics, no account, and no cloud.

---

## Credits

- **TTS:** [Kokoro](https://huggingface.co/hexgrad/Kokoro-82M) neural TTS via [kokoro-onnx](https://github.com/thewh1teagle/kokoro-onnx)
- **Inference:** [ONNX Runtime](https://onnxruntime.ai) (CUDA / CPU)
- **Audio I/O:** [python-soundfile](https://github.com/bastibe/python-soundfile) / libsndfile, playback via [pygame](https://www.pygame.org)
- **Phonemization:** [espeak-ng](https://github.com/espeak-ng/espeak-ng)
- **Compilation:** [Nuitka](https://nuitka.net)
- Built by **N2NHU Labs**

## License

This program is free software: you can redistribute it and/or modify it under the terms of the **GNU General Public License v3.0** as published by the Free Software Foundation. It is distributed in the hope that it will be useful, but **WITHOUT ANY WARRANTY**; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the [LICENSE](LICENSE) file or <https://www.gnu.org/licenses/gpl-3.0.html> for details.

Note: the Kokoro model weights and the bundled third-party libraries carry their own licenses (Kokoro-82M is Apache-2.0); see their respective projects.

*RealNews Listener — because the news should come to you, out loud, from every direction.*
