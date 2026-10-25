
# Purify

A bash script for streamlining the process of removing music from video files, YouTube videos, and audio files.

## Why?

For the people who choose to stop listening to music, for the various reasons such as disliking their emotional effect, improving their focus, improving the quality of sleep, and others you can find by doing some research.

In today's age, music is almost present everywhere, the majority of videos include background music, many shops play music, and so on.

And unfortunately, many informative and useful videos have background music, stopping watching them at all would be costly.

Thus, an automated tool, for cleaning the hardly escapable media from music, would help alleviate the problem and reduce the exposure to music.

## How?

There is interest in computer science in the task of vocal an instrument separation, which is to separate an audio signal into the audio of vocals (containing human speech), and the audio of the instruments (containing music).

At the current days, the task is solved using [deep learning](https://en.wikipedia.org/wiki/Deep_learning).

One great consumable solution is [Universal Vocal Remover (UVR5)](https://github.com/anjok07/ultimatevocalremovergui), and it's cli counterpart [audio-separator](https://github.com/nomadkaraoke/python-audio-separator) that uses the same AI models.

This project provides a bash script that combines [yt-dlp](https://github.com/yt-dlp/yt-dlp), [ffmpeg](https://en.wikipedia.org/wiki/FFmpeg), and audio-separator, for streamlining the process of removing music from videos and audio.

## Why a bash script?

Given the limited resources I currently have, a bash script is the easiest [MVP](https://en.wikipedia.org/wiki/Minimum_viable_product) I can provide. I would love to see more cross-platform friendly, and non tech-savvy users friend solutions being made and inspired from this.

## Example Usages

```sh
# Local video file
purify ./a-video-file.mkv
purify ./a-video-file.webm

# Local audio file
purify ./an-audio-file.mp3
purify ./an-audio-file.mp4

# A directory of local video and/or audio files
purify ./video-series
# All the videos and/or audio files in the current working directory
purify .

# A YouTube video
purify https://www.youtube.com/watch?v=XfeuCfOUuXg

# A YouTube playlist
purify https://www.youtube.com/playlist?list=PLZHQObOWTQDMV832OM2s0zRjsgphR5UJR
```

## Requirements

- `yt-dlp`, `ffmpeg`, `ffprobe` and `audio-separator` must be available on path.

## Processing Pipeline

### 1. Fetching the media file

If the input address is a web url, then `yt-dlp` is used for downloading it locally, into the current working directory.

### 2. Extracting the audio track

`ffmpeg` is used for extracting the audio track, and encoding it as a `.wav` file in preparation for consumption by `audio-separator`.

If the media file contains multiple audio tracks, then the user is prompted to select which audio track to process.

A temporary directory is created for storing the extracted track.

### 3. Vocals separation

`audio-separator` is used for extracting the vocals stem. The model `Kim_Vocal_2.onnx` is elected, and `batch_size` is set to 4 for improving the performance. Logging is set to only errors to avoid cluttering the terminal. And only the `Vocals` stem is extracted.

The extracted stem is stored within the temporary directory created previously.

### 4. Merging the audio track back

If it was a video file, the audio is added as an additional track to the video, encoding it with a reasonable lossy compression that keeps balance between quality and size. The added audio track will be made the default for the video.

If it was an audio file, then the `Vocals` stem would only be re-encoded.

The result file would be given the suffix ` (Purified)` in it's filename, and be stored in the same directory of the original media file.

### 5. Cleanup

The temporarily created directory is deleted, along the temporary files it created within it.

## Extra Features

- The script shall output status messages at each stage start.
- ANSI escape sequences are used for colorizing the script status messages.
