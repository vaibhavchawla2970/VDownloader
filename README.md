<div align="center">
  <img src="https://i.postimg.cc/fyz59fLm/icon128.png" alt="VDownloader Logo" width="96">
  <h1>VDownloader</h1>
  <p><strong>YouTube video/audio downloader — Browser Extension + Local Server</strong></p>
  <p>
    <a href="#screenshots">Screenshots</a> •
    <a href="#features">Features</a> •
    <a href="#prerequisites">Prerequisites</a> •
    <a href="#installation">Installation</a> •
    <a href="#how-it-works">How It Works</a> •
    <a href="#usage">Usage</a>
  </p>
  <p>
    <a href="https://vdownloader-yt.netlify.app/" target="_blank">:globe_with_meridians: Website</a> •
    <a href="https://github.com/vaibhavchawla2970/VDownloader" target="_blank">:octocat: GitHub</a>
  </p>
</div>

---

## About

VDownloader is a browser extension that downloads YouTube videos, audio, and playlists directly to your computer. It pairs a **Chrome/Firefox extension** (popup UI) with a **local Python server** that handles the actual downloading via `yt-dlp`.

No cloud services, no third-party websites, no ads — everything runs locally on your machine.

---

## Features

- **4K & HD downloads** — Choose quality from 360p up to 4K
- **Audio extraction** — Download as MP3, M4A, OPUS, FLAC, or WAV
- **Video + Audio combined** — Best quality with automatic merging (requires ffmpeg)
- **Video only / Audio only** — Separate streams for advanced use
- **Playlist support** — View, select, and download individual or all videos in a playlist
- **YouTube Shorts** — Fully compatible
- **Age-restricted videos** — Works with cookie-based authentication
- **Custom filename patterns** — Use `{title}`, `{id}`, `{ext}` and more
- **Live progress** — Real-time download progress in the popup
- **Live server logs** — Built-in log viewer inside the extension
- **Persistent pinned window** — Pin the popup as a standalone window
- **Works offline** — After initial setup, no internet needed for the extension itself (server must be running locally)

---

## Screenshots

<div align="center">

| Format Selection | Playlist Manager | Settings |
|:---:|:---:|:---:|
| <img src="https://i.postimg.cc/RhgdQJ6b/video-only.png" width="220" alt="Video+Audio"> | <img src="https://i.postimg.cc/90m17YM8/playlist.png" width="220" alt="Playlist"> | <img src="https://i.postimg.cc/fy8CjSt2/settings.png" width="220" alt="Settings"> |

| Audio Only | Video Only | Playlist Quality |
|:---:|:---:|:---:|
| <img src="https://i.postimg.cc/Y0r8Wz9h/audio-only.png" width="220" alt="Audio Only"> | <img src="https://i.postimg.cc/RhgdQJ6b/video-only.png" width="220" alt="Video Only"> | <img src="https://i.postimg.cc/cCxTnBHB/playlist-video-quality.png" width="220" alt="Playlist Quality"> |

| Server Logs |
|:---:|
| <img src="https://i.postimg.cc/d356mZkN/sever-Logs.png" width="440" alt="Server Logs"> |

</div>

---

## Prerequisites

| Software | Required | Notes |
|:---|:---:|:---|
| **Python 3.8+** | Yes | [python.org](https://www.python.org/downloads/) — check "Add Python to PATH" |
| **pip** | Yes | Comes with Python 3.4+ |
| **yt-dlp** | Yes | Installed automatically by the server |
| **FastAPI + uvicorn** | Yes | Installed automatically by the server |
| **ffmpeg** | Recommended | Required for merging video+audio (best quality) and format conversions |

### Installing ffmpeg (recommended)

ffmpeg enables **best-quality combined downloads** (video+audio merged) and format conversion. Without it, only video-only or audio-only streams are available.

**Windows:**
- **Option A (chocolatey):** `choco install ffmpeg`
- **Option B (scoop):** `scoop install ffmpeg`
- **Option C (manual):** Download from [ffmpeg.org](https://ffmpeg.org/download.html), extract, and add the `bin` folder to your PATH

**macOS:**
```bash
brew install ffmpeg
```

**Linux:**
```bash
sudo apt install ffmpeg        # Debian/Ubuntu
sudo dnf install ffmpeg        # Fedora
```

---

## Installation

### 1. Start the Server

**Quick start (Windows):**
Double-click `server_v1.3.0/start_server.bat`

**Manual start (any OS):**
```bash
cd server_v1.3.0
pip install -U yt-dlp fastapi uvicorn
python server.py
```

The server will start at `http://localhost:8000`. Keep this terminal window open.

> **Note:** The server handles all the downloading. You must keep it running while using the extension. It logs every action so you can see what's happening.

### 2. Install the Extension

#### Chrome / Brave / Edge

1. Download **`extension-chrome-brave-edge.zip`**
2. Unzip it to a folder (keep the folder, don't delete it)
3. Open `chrome://extensions` (or `brave://extensions`, `edge://extensions`)
4. Enable **Developer mode** (toggle in the top-right)
5. Click **Load unpacked**
6. Select the unzipped folder
7. The VDownloader icon appears in the toolbar

#### Firefox

1. Download **`extension-firefox.zip`**
2. Open `about:debugging#/runtime/this-firefox`
3. Click **Load Temporary Add-on**
4. Select the zip file directly
5. The VDownloader icon appears in the toolbar

> **Firefox note:** Temporary add-ons are removed when Firefox closes. For permanent installation, you would need to sign the add-on with Mozilla. The Firefox version includes extra CSS fixes (`width: 300px`, `max-height: 440px`, and a flexbox scroll fix) to match Chrome's popup behavior.

---

## How It Works

```
┌─────────────────────────────────────────────────────────┐
│                    Your Browser                          │
│  ┌────────────────────────────────────────────────┐     │
│  │  VDownloader Extension                         │     │
│  │  ┌──────────┐    ┌───────────────────────────┐ │     │
│  │  │ popup.js │◄──►│ background.js (relay)     │ │     │
│  │  └──────────┘    └──────────┬────────────────┘ │     │
│  └──────────────────────────────┼──────────────────┘     │
└─────────────────────────────────┼────────────────────────┘
                                  │ HTTP requests (localhost:8000)
                                  ▼
┌─────────────────────────────────────────────────────────┐
│              Local Python Server                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  FastAPI + uvicorn                              │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │    │
│  │  │ /formats │  │/download │  │ /download/   │  │    │
│  │  │          │  │          │  │ status/{id}  │  │    │
│  │  └──────────┘  └────┬─────┘  └──────────────┘  │    │
│  └─────────────────────┼───────────────────────────┘    │
│                        │                                │
│                        ▼                                │
│              ┌──────────────────┐                       │
│              │  yt-dlp process  │                       │
│              │  (downloads to   │                       │
│              │   ~/Downloads/)  │                       │
│              └──────────────────┘                       │
└─────────────────────────────────────────────────────────┘
```

1. You navigate to a YouTube video and click the VDownloader icon
2. The extension fetches available formats from the server (`/formats`)
3. You pick quality and type (video, audio, or both)
4. The extension tells the server to download (`/download`)
5. The server runs `yt-dlp` with your chosen options
6. The file saves directly to your `~/Downloads` folder
7. The popup shows real-time progress and server logs

---

## Usage

### Basic download

1. Go to any YouTube video (or Short)
2. Click the VDownloader icon
3. Wait for formats to load
4. Select **Video+Audio**, **Video only**, or **Audio only**
5. Choose your preferred quality
6. Click **Download**
7. The file appears in your `Downloads` folder

### Playlist download

1. Go to any YouTube playlist
2. Click the VDownloader icon
3. The playlist loads with checkboxes for each video
4. Use **Select All** / **Select None** to choose videos
5. Click **Download Playlist**
6. Optionally use **Playlist Video Quality** to apply a uniform quality to all

### Pin as window

Click the **pin icon** in the header to keep the popup open as a persistent window. This is useful for:
- Long playlist downloads
- Monitoring progress without the popup closing
- Browsing YouTube while the extension stays open

### Server controls

The **Settings** panel shows:
- Server status (online/offline)
- yt-dlp version
- ffmpeg availability
- Start / Stop / Restart buttons
- Custom filename pattern

### Log viewer

Click **View Logs** in the Settings panel to see real-time server logs. Use **Clear** to reset or **Export** to save logs to a file.

### Filename pattern

Customize filenames using `{title}`, `{id}`, `{ext}`, `{uploader}`, `{upload_date}`, and more. Default: `{title}.{ext}`

### Browser cookies (optional)

Enable **Use Browser Cookies** in Settings if you need to download age-restricted or private videos. The extension will pass your browser's YouTube cookies to the server.

---

## File Structure

```
VDownloader/
├── extension_v1.3.0/          # Source files for the extension
│   ├── manifest.json          # Extension manifest (MV3)
│   ├── popup.html             # Extension popup UI
│   ├── popup.js               # Popup logic
│   ├── background.js          # Background service worker (relay)
│   ├── content.js             # Content script (video detection)
│   ├── styles.css             # Popup styles
│   └── icon*.png              # Extension icons
│
├── extension-chrome-brave-edge.zip   # Chrome/Brave/Edge package
├── extension-firefox.zip             # Firefox package
│
├── server_v1.3.0/             # Python download server
│   ├── server.py              # FastAPI server (yt-dlp wrapper)
│   └── start_server.bat       # Windows one-click launcher
│
├── Images and videos/         # Screenshots and demo
│   ├── icon128.png            # Logo
│   ├── video+audio.png        # Screenshot
│   ├── audio only.png         # Screenshot
│   ├── video only.png         # Screenshot
│   ├── playlist.png           # Screenshot
│   ├── playlist video quality.png  # Screenshot
│   ├── settings.png           # Screenshot
│   ├── sever Logs.png         # Screenshot
│   └── demo.mp4               # Demo video
│
├── index.html                 # Landing page
├── styles.css                 # Landing page styles
├── script.js                  # Landing page scripts
└── README.md                  # This file
```

---

## Troubleshooting

| Problem | Solution |
|:---|:---|
| **Server won't start** | Make sure Python 3.8+ is installed and in PATH. Run `pip install -U yt-dlp fastapi uvicorn` manually. |
| **"yt-dlp not found"** | Run `pip install -U yt-dlp` and restart the server. |
| **No formats loading** | Check that the server is running (`http://localhost:8000`). Check the Settings panel for server status. |
| **Download fails silently** | Open the log viewer (Settings → View Logs) to see error details. Common causes: missing ffmpeg, invalid URL, network issues. |
| **Age-restricted video fails** | Enable **Use Browser Cookies** in Settings. Make sure you're logged into YouTube in your browser. |
| **Firefox extension disappears** | Firefox temporary add-ons are removed when Firefox closes. You need to re-add it each session, or sign it via Mozilla. |
| **"FFmpeg not found" in logs** | Install ffmpeg and restart the server. Without it, video+audio combined downloads won't merge properly. |

---

## Technical Details

- **Extension:** Manifest V3, Chrome Extension API (works in Chrome, Brave, Edge, Firefox)
- **Server:** Python 3.8+, FastAPI, uvicorn, yt-dlp
- **Download engine:** [yt-dlp](https://github.com/yt-dlp/yt-dlp) (active fork of youtube-dl)
- **Local only:** All traffic is between the extension and `localhost:8000` — no external requests
- **Platform:** Windows, macOS, Linux (server is cross-platform; extension works in any Chromium browser + Firefox)

---

## License

This project is for personal and educational use. Respect YouTube's Terms of Service. Only download content you have the right to download.

---

<div align="center">
  <sub>Built with yt-dlp, FastAPI, and a lot of caffeine.</sub>
</div>
