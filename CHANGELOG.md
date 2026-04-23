# Changelog

All notable changes to the `video-captions-burn` Agentino skill are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and this skill adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-04-23

First public release for the Agentino marketplace.

### Added

- `agentino skill exec video-captions-burn` burns `.srt`, `.vtt`, `.ass` or `.ssa` captions into a source video:
  - Configurable font, font size, fill colour and outline colour (named or `#RRGGBB`).
  - Three positions: `bottom` / `top` / `middle`, with configurable vertical margin.
  - Adjustable outline width (`0` disables).
  - Stream-copies the audio track to avoid any re-encode loss.
  - Single `ffmpeg -vf subtitles=...` pass with `libx264` CRF 20 + `+faststart`.
- Safe filter-path escaping (`:`, `\`, `'`).
- Reports input/output duration and subtitle cue count via `ffprobe`.

### Tested

- 1.51 s 1080p slideshow MP4 + 1-cue SRT → 1.51 s captioned MP4 with burned-in subtitle.
- End-to-end chain in the `slides-to-video` pipeline (10.94 s captioned output, 4-cue SRT).
- Zero findings from `agentino skill exec skill-security-check -i path=skill.yaml` (fail-on = high).

### Requires

- Agentino ≥ 1.2.0-rc.1
- `ffmpeg` with libass (standard on Homebrew / apt builds)
