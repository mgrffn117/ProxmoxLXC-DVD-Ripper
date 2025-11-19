# FFmpeg Merge Guide: Lossless Joining with Subtitle Preservation

This guide documents how to merge split video files (e.g., DVD rips, camcorder footage) into a single file without re-encoding. 

**Key Feature:** This method uses the MKV container and the `-map 0` flag to ensure **all** data streams (Video, Audio, DVD Bitmaps, Chapters) are preserved. Standard MP4 merging often discards DVD subtitles.

## 1. Installation

### Windows 10/11
The easiest method is using Winget (Windows Package Manager).

```powershell
winget install Gyan.FFmpeg
```
*Note: You must close and reopen your terminal after installation.*

### Linux (LXC / VM / Native)
Use your distribution's package manager:

**Debian / Ubuntu / Proxmox LXC**
```bash
sudo apt update && sudo apt install ffmpeg
```

**Fedora / RHEL**
```bash
sudo dnf install ffmpeg
```

**Arch Linux**
```bash
sudo pacman -S ffmpeg
```

---

## 2. Preparation: The List File

FFmpeg requires a text file listing the inputs to concatenate. To avoid path issues, place all source files in a single directory.

### File Format Requirements
1.  The file must be named `list.txt` (recommended).
2.  Each line must start with `file`.
3.  Filenames must be wrapped in single quotes `'`.

**Example `list.txt` content:**
```text
file 'part1.mp4'
file 'part2.mp4'
file 'part3.mp4'
```

### Quick Generation (Linux/Bash)
Instead of typing manually, navigate to your video folder and run:
```bash
printf "file '%s'\n" *.mp4 > list.txt
```
*(Ensure files are named alphabetically so they sort correctly)*

---

## 3. The Merge Command

Run this command in the folder containing your videos and `list.txt`. 

```bash
ffmpeg -f concat -safe 0 -i list.txt -map 0 -c copy output.mkv
```

### Command Breakdown
| Flag | Function |
| :--- | :--- |
| `-f concat` | Activates the concatenation demuxer. |
| `-safe 0` | Disables strict path checking (prevents errors with relative paths). |
| `-i list.txt` | Input source is our list file, not a video file. |
| **`-map 0`** | **Critical.** Maps ALL streams (Video, Audio, Subtitles, Data) to the output. Without this, FFmpeg drops DVD subtitles. |
| `-c copy` | **Lossless.** Copies the raw bitstream. No re-encoding, no quality loss, instant speed. |
| `output.mkv` | **MKV Container.** Required to hold `dvd_subtitle` (bitmap) tracks. MP4 cannot support these tracks natively. |

---

## Troubleshooting

*   **Stuttering/Glitches:** If the output video freezes or audio desyncs, your source files likely have different properties (resolution, frame rate, or codecs). You cannot use `-c copy` for mismatched files. You must re-encode:
    ```bash
    ffmpeg -f concat -safe 0 -i list.txt -c:v libx264 -crf 23 -c:a aac output_fixed.mkv
    ```
*   **Missing Subtitles:** Ensure you used the `.mkv` extension and included `-map 0`. If you force `.mp4`, bitmap subtitles will be dropped.
