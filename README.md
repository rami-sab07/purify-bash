# Purify

Purify is a Bash command-line tool for reducing background music in local
media files, YouTube videos, and YouTube playlists. It extracts the speech or
vocal stem with `audio-separator`, then writes a purified copy beside the
source file.

Purify is intended for Linux users who want to make spoken-word media easier
to listen to without manually editing each file.

## How it works

Purify combines these tools:

- [yt-dlp](https://github.com/yt-dlp/yt-dlp) downloads media from supported
  websites.
- [FFmpeg](https://ffmpeg.org/) and `ffprobe` extract and inspect media
  streams, then merge the purified audio back into video.
- [audio-separator](https://github.com/nomadkaraoke/python-audio-separator)
  separates vocals from music using an AI model.

The default model is `Kim_Vocal_2.onnx`. `audio-separator` downloads it when it
is needed for the first time.

## Installation

Purify requires Bash 4 or newer. Clone the repository and install the script
somewhere on your `PATH`:

```sh
git clone https://github.com/rami-sabbagh/purify-bash.git
cd purify-bash
chmod +x purify
sudo install -m 0755 purify /usr/local/bin/purify
```

### Install yt-dlp and audio-separator with pipx

`pipx` installs each Python application in an isolated environment while still
making its command available on your `PATH`.

On Fedora:

```sh
sudo dnf install pipx
pipx ensurepath
```

On Ubuntu:

```sh
sudo apt update
sudo apt install pipx
pipx ensurepath
```

Restart your shell, or open a new terminal, then install the applications:

```sh
pipx install yt-dlp
pipx install audio-separator
```

### Install FFmpeg

On Ubuntu:

```sh
sudo apt update
sudo apt install ffmpeg
```

On Fedora, enable RPM Fusion if FFmpeg is not available from your configured
repositories, then install it:

```sh
sudo dnf install \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install ffmpeg
```

The commands above provide `ffmpeg` and `ffprobe`, both of which Purify needs.

## Usage

```sh
# Local video file
purify ./a-video-file.mkv
purify ./a-video-file.webm

# Local audio file
purify ./an-audio-file.mp3
purify ./an-audio-file.mp4

# A directory of local media files
purify ./video-series
purify .

# Include subdirectories
purify --recursive ./video-series

# Replace a local file after successful processing
purify --overwrite ./a-video-file.mkv

# Open each result in MPV after successful processing
purify --mpv ./a-video-file.mkv

# Show detailed output from each processing stage
purify --verbose ./a-video-file.mkv

# A YouTube video
purify https://www.youtube.com/watch?v=XfeuCfOUuXg

# A YouTube playlist
purify https://www.youtube.com/playlist?list=PLZHQObOWTQDMV832OM2s0zRjsgphR5UJR
```

You can pass multiple files, directories, and URLs in one invocation.
Directories are searched only at their top level by default. Use `--recursive`
or `-r` to include subdirectories.

Local files are written beside their sources with a ` (Purified)` suffix.
Existing purified files are skipped, and an available numbered suffix is used
if the output name already exists. YouTube downloads are always replaced by
their purified files. For local inputs, use `--overwrite` or `-o` to replace
the original after successful processing. `--override` is accepted as an alias
for `--overwrite`.

When a source contains multiple audio tracks, Purify displays the available
tracks and prompts you to choose one.

## Options

```text
-r, --recursive  Search directories and their subdirectories.
-o, --overwrite  Replace local input files after successful processing.
    --override   Alias for --overwrite.
-q, --quality    Maximum video height to download (default: 720p).
-v, --verbose    Show detailed output from each processing stage.
    --mpv        Open each result in MPV after successful processing.
-h, --help       Show the help message.
```

## Requirements

- Bash 4 or newer
- `yt-dlp`
- `ffmpeg`
- `ffprobe`
- `audio-separator`
- `mpv` (only required when using `--mpv`)

The AI model can require substantial disk space, memory, and processing time.
Processing speed depends on the input duration and your hardware.

## Verify the installation

Run these commands without processing a media file:

```sh
bash -n purify
purify --help
command -v yt-dlp ffmpeg ffprobe audio-separator
```

## Processing pipeline

1. If the input is a URL, `yt-dlp` downloads it to the current directory.
2. `ffprobe` identifies the available audio tracks. If there is more than one,
   Purify asks which track to process.
3. `ffmpeg` extracts the selected track to a temporary WAV file.
4. `audio-separator` extracts the `Vocals` stem with `Kim_Vocal_2.onnx`.
5. For video, `ffmpeg` adds the purified stem as the default audio track while
    preserving the video and subtitle streams. For audio-only files, it encodes
    the stem in the source format when supported.
6. Temporary files are removed when processing finishes.

## License

Purify is released under the [MIT License](LICENSE).

The dependencies and AI model may have their own licenses and terms. Review
those terms before redistributing them or media processed with them.
