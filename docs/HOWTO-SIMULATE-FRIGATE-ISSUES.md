# HOWTO: Simulate Camera Issues in Frigate NVR for Automated Diagnostic Testing

This guide explains how to configure **Frigate NVR** + **go2rtc** + **FFmpeg** to simulate common IP camera problems (green screen, frozen image, incorrect bitrate, partial/total obstruction) for automated diagnostic testing — such as the **AlertaHub Vision** technical inspection component.

---

## Overview

Frigate does not have a built-in "fault injection" mode. Instead, you create synthetic RTSP streams using FFmpeg (via go2rtc) that mimic the visual symptoms a faulty camera or NVR would produce. Frigate then consumes these streams as if they were real cameras.

```
[FFmpeg synthetic stream] → [go2rtc RTSP server] → [Frigate camera input] → [AlertaHub Vision]
```

---

## Prerequisites

- Frigate NVR running (Docker recommended)
- `go2rtc` embedded in Frigate (default since Frigate 0.12+)
- `frigate.yml` accessible for editing
- Optional: separate `ffmpeg` binary if generating streams externally

---

## 1. Green Screen

Simulates a camera producing a fully green frame — typically caused by H.265→H.264 decode mismatch, codec errors, or corrupted I-frames.

### Option A: Pure synthetic green stream

```yaml
go2rtc:
  streams:
    sim_green_screen:
      - "ffmpeg:#input=-f lavfi -i color=size=1280x720:rate=15:color=green#video=h264"

cameras:
  test_green_screen:
    ffmpeg:
      inputs:
        - path: rtsp://127.0.0.1:8554/sim_green_screen
          input_args: preset-rtsp-restream
          roles:
            - detect
            - record
    detect:
      width: 1280
      height: 720
```

### Option B: Overlay green filter on a real stream

```yaml
go2rtc:
  streams:
    sim_green_overlay:
      - "ffmpeg:rtsp://user:pass@camera-ip/stream1#video=h264#raw=-vf colorchannelmixer=0:1:0:0:0:1:0:0:0:1:0:0"
```

---

## 2. Frozen / Stuck Frame

Simulates a stream that is "alive" at the transport level but the image never changes.

```yaml
go2rtc:
  streams:
    sim_frozen_frame:
      - "ffmpeg:#input=-loop 1 -i /media/frigate/test_frame.jpg -t 3600#video=h264"
```

---

## 3. Incorrect / Unstable Bitrate

### Too low bitrate (< 256 kbps)
```yaml
go2rtc:
  streams:
    sim_low_bitrate:
      - "ffmpeg:rtsp://user:pass@camera-ip/stream1#video=h264#raw=-b:v 128k -maxrate 128k -bufsize 64k"
```

### Too high / uncontrolled bitrate
```yaml
go2rtc:
  streams:
    sim_high_bitrate:
      - "ffmpeg:#input=-f lavfi -i mandelbrot=size=1920x1080:rate=30#video=h264#raw=-b:v 20M -maxrate 25M"
```

---

## 4. Partial Image Obstruction

```yaml
go2rtc:
  streams:
    sim_partial_obstruction:
      - "ffmpeg:rtsp://user:pass@camera-ip/stream1#video=h264#raw=-vf drawbox=x=0:y=0:w=640:h=720:color=black@0.85:t=fill"
```

---

## 5. Total Obstruction / Black Screen

```yaml
go2rtc:
  streams:
    sim_black_screen:
      - "ffmpeg:#input=-f lavfi -i color=size=1280x720:rate=15:color=black#video=h264"
```

---

## 6. Validation Table for AlertaHub Vision

| Camera Name      | Expected Fault           | Detection Signal                      |
|------------------|--------------------------|---------------------------------------|
| test_green       | Green screen             | Hue histogram spike near 120°         |
| test_frozen      | Frozen frame             | Zero inter-frame difference           |
| test_black       | Total obstruction        | Mean luminance < threshold            |
| test_partial     | Partial obstruction      | Region-of-interest darkness > 80%     |
| sim_low_bitrate  | Bitrate below spec       | FFmpeg stream info < expected profile |
| sim_high_bitrate | Bitrate above spec       | FFmpeg stream info > expected profile |

---

## References

- [Frigate NVR Docs – go2rtc](https://docs.frigate.video/configuration/camera_specific)
- [go2rtc FFmpeg sources](https://github.com/AlexxIT/go2rtc/blob/master/README.md#source-ffmpeg)
- [FFmpeg lavfi color source](https://ffmpeg.org/ffmpeg-filters.html#color-1)
- [FFmpeg drawbox filter](https://ffmpeg.org/ffmpeg-filters.html#drawbox)
- [Frigate GitHub Issues – Green Screen](https://github.com/blakeblackshear/frigate/issues/1095)
```

***