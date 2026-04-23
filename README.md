# video-captions-burn

> **Burn SRT/VTT subtitles directly into the video stream with configurable font, colour, outline and position. One ffmpeg pass, pure local.**

An Agentino skill that takes a source video and a subtitle file (`.srt`, `.vtt`, `.ass`, `.ssa`) and renders an MP4 with the captions baked into the picture — so they show on every platform without a separate subtitle track, and survive any further re-encoding. Audio is stream-copied to avoid quality loss.

Designed to chain after [`video-slideshow`](https://github.com/dagoSte/agentino-skill-video-slideshow) or any pipeline that produced an MP4 + SRT (e.g. [`video-voiceover`](https://github.com/dagoSte/agentino-skill-video-voiceover) with edge-tts).

## Install

Requires [Agentino](https://github.com/dagoSte/agentino) ≥ `1.2.0-rc.1` and `ffmpeg` compiled with libass (standard for Homebrew and apt builds).

```bash
brew install ffmpeg                                                    # macOS
# sudo apt install ffmpeg                                              # Debian / Ubuntu

agentino marketplace install dagoSte/agentino-skill-video-captions-burn
agentino skill show video-captions-burn
```

## Use

```bash
agentino skill exec video-captions-burn \
  -i video_path=/tmp/talk.mp4 \
  -i subtitles_path=/tmp/talk.srt \
  -i output_path=/tmp/talk.captioned.mp4 \
  -i font_size=32 \
  -i position=bottom
```

Custom styling:

```bash
agentino skill exec video-captions-burn --input-json '{
  "video_path": "/tmp/talk.mp4",
  "subtitles_path": "/tmp/talk.srt",
  "output_path": "/tmp/talk.captioned.mp4",
  "font_name": "Helvetica",
  "font_size": 36,
  "font_color": "#FFEB3B",
  "outline_color": "black",
  "outline_width": 3,
  "position": "bottom",
  "margin_v": 80
}'
```

### Example output

```json
{
  "output_path": "/tmp/talk.captioned.mp4",
  "input_duration_s": 1634.21,
  "output_duration_s": 1634.21,
  "subtitle_count": 218,
  "font_name": "Helvetica",
  "font_size": 36,
  "position": "bottom"
}
```

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `video_path` | string | **required** | Absolute path to the source video. |
| `subtitles_path` | string | **required** | Absolute path to a `.srt`, `.vtt`, `.ass` or `.ssa` file. |
| `output_path` | string | `<video>.captioned.<ext>` | Destination MP4. |
| `font_name` | string | `Arial` | Font family (must be installed on the system). |
| `font_size` | integer | `28` | Point size. 28 for 1080p, 20 for 720p, 36 for 4K. |
| `font_color` | string | `white` | Named (`white`, `yellow`, `cyan`, `red`, `green`, `black`, `blue`, `gray`) or `#RRGGBB`. |
| `outline_color` | string | `black` | Outline colour, same format as `font_color`. |
| `outline_width` | integer | `2` | Outline thickness in pixels. `0` disables the outline. |
| `position` | string | `bottom` | `bottom` \| `top` \| `middle` (always horizontally centred). |
| `margin_v` | integer | `40` | Vertical margin from the chosen edge, in pixels. |

## Outputs

- **`output_path`** — absolute path to the captioned MP4.
- **`input_duration_s`**, **`output_duration_s`** — both from `ffprobe`; they should match.
- **`subtitle_count`** — number of SRT/VTT cues discovered.
- **`font_name`**, **`font_size`**, **`position`** — resolved style summary.

## How it works

1. **Validate inputs** — both paths must exist, and the subtitle file must have a supported extension.
2. **Resolve the ASS style** — names and `#RRGGBB` strings are converted to ffmpeg's `&H00BBGGRR` colour format; position is mapped to `Alignment` 2/8/10.
3. **Build `force_style`** — font, colours, outline, alignment, margin packed into a single subtitles-filter argument.
4. **Escape the subtitles path** for the filter string (`:`, `\`, `'` are special).
5. **Render in one ffmpeg pass** — `-vf subtitles=<path>:force_style=<style>` + `libx264` CRF 20, `-c:a copy` to preserve the audio track byte-for-byte, `+faststart`.
6. **Re-probe** input and output durations; if they diverge by more than a rounding error something went wrong (the skill surfaces both).

## Safety

- `sandbox_level: none` (ffmpeg discovery).
- `network_access: false`.
- `file_access: read-write`.
- `agentino skill exec skill-security-check -i path=skill.yaml` → zero findings.

## License

MIT — see [`LICENSE`](./LICENSE).
