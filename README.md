# cmdplay-pp

Play a video inside a terminal window as ASCII / ANSI art, with colours, dithering and live resizing.

This is an improved version of [mariiaan/cmdplay-pp](https://github.com/mariiaan/cmdplay-pp), which is itself the
rewrite of [CmdPlay](https://github.com/mariiaan/CmdPlay). The player, the renderer and the original ideas come
from Marian Wölpper's project; this fork adds the changes listed below on top of that work.

## Features

- ASCII playback of any video FFmpeg can decode, straight in the console
- Colour, true-colour and colour / grayscale dithering
- Dynamic resolution: resize the window while playing and the grid follows
- Multi-threaded decoding, frames are decoded and shown while the video plays
- Local files and URLs (YouTube and the sites yt-dlp supports)

## What this fork adds

**URL playback.** Pass a link instead of a path and it is downloaded with `yt-dlp`, with progress, into a
`videos` folder next to the executable.

**Automatic clean-up.** The download is deleted when playback ends, when you quit with `Q`, when you press
Ctrl+C, and when the window is closed. Downloads left behind by a killed run are removed on the next start.
Local files are never touched.

**Better stream selection.** The format picker targets 1080p and the highest frame rate available, prefers
H.264 for decode speed, and asks for AAC audio so the merged MP4 stays conventional:

```
bv*[height<=1080]+ba[ext=m4a]/b[height<=1080]/b   -S res:1080,fps,codec:h264
```

If a site only offers a combined stream, or ffmpeg is missing for merging, it retries with a single-file format
instead of failing. The older hard 720p cap is gone, so 1080p sources arrive as 1080p.

**Exact downscaling.** Frames are scaled with FFmpeg's area filter, so every character is the true average of
the pixels it represents instead of a bilinear sample of four of them.

**Hotkey bar.** A single row under the video lists the keys and the state of every toggle: green when the option
is on, grey when it is off, plain white for plain actions. It redraws with each frame and immediately on a
keypress, and drops entries from the right in narrow windows rather than wrapping.

**Restart.** `R` plays the video again from the first frame.

**Playback ends by itself.** The player tracks the container duration, so the process stops and the download is
removed at the end of the video instead of running until you quit.

**Fixes.** Removed debug output that printed the audio stream count at startup, made the audio clock per
playback (a restart used to inherit the old playback position and jump straight to the end), and gave the
window-size query a sane fallback when stdout is redirected.

## Requirements

- Windows 10 / 11
- Visual Studio 2022 with the Desktop development with C++ workload (toolset v143) to build
- The FFmpeg 6.x runtime DLLs that already sit in `bin` (`avcodec-60`, `avformat-60`, `avutil-58`,
  `swresample-4`, `swscale-7`)
- `yt-dlp` on `PATH` for URLs, plus `ffmpeg` so yt-dlp can merge video and audio

## Build

Open `cmdplaypp.sln` and build `Release | x64`, or from a developer prompt:

```
msbuild cmdplaypp.sln -p:Configuration=Release -p:Platform=x64
```

The executable lands in `x64\Release\cmdplaypp.exe`. Copy it next to the DLLs in `bin` before running, or run it
from `bin` after copying. A macOS makefile is included but the changes in this fork are Windows-focused.

## Usage

```
cmdplay.exe
```

asks for a path or URL. You can also pass it directly, and most terminals accept a dragged file:

```
cmdplay.exe "C:\videos\clip.mp4"
cmdplay.exe https://www.youtube.com/watch?v=...
```

The startup screen prints the version and the controls before the prompt.

## Controls

| Key | Action |
| --- | --- |
| `SPACE` | play / pause |
| `C` | colours |
| `D` | colour dithering |
| `T` | text / grayscale dithering |
| `A` | accurate colours (true-colour) |
| `B` | full pixel (solid colour blocks) |
| `R` | restart from the beginning |
| `Q` | quit, deleting the download |

## About picture quality

The download can be 1080p, the screen cannot. A 1920×1080 frame is about two million pixels, while a 100×30
console is 3,000 cells, roughly 0.14% of them — that gap is what looks like blur, and no download setting
closes it. What actually helps, in order: maximise the window (doubling both dimensions gives four times the
cells), then `B` with `C` and `A` for solid true-colour blocks, then a higher frame rate for smoother motion.

## Performance tips

- Use Windows Terminal rather than conhost; it handles the escape sequences far better
- Colours, true-colour and both dithering modes cost throughput, so use them at lower resolution
- With colours and dithering off, Windows Terminal can hold quite high resolutions

## License

MIT, see [LICENSE](LICENSE).

This fork is released under the MIT license. The code it derives from is copyright (c) 2023 Marian Wölpper
and is MIT licensed as well; that notice stays in the LICENSE file, which is what the license asks for.

Third-party components keep their own licenses: FFmpeg (LGPL 2.1+), yt-dlp (Unlicense), miniaudio (public
domain / MIT-0).
