# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **nvr-frigate-demo** repository — a documentation project providing guidance on simulating common IP camera and Frigate NVR diagnostic scenarios for automated testing.

The project documents how to configure Frigate NVR + go2rtc + FFmpeg to generate synthetic RTSP streams that mimic real camera failures (green screen, frozen frames, obstruction, bitrate issues, etc.), which can then be consumed by diagnostic tools like AlertaHub Vision for testing.

## Repository Structure

- **README.md** — High-level project description and purpose
- **docs/HOWTO-SIMULATE-FRIGATE-ISSUES.md** — Complete technical guide with YAML configuration examples for each failure scenario:
  - Green screen simulation
  - Frozen/stuck frame
  - Incorrect bitrate (too low/too high)
  - Partial and total obstruction
  - Validation table for expected detection signals

## No Build/Test/Lint Commands

This is a **documentation-only repository**. There are no:
- Build scripts or commands
- Tests to run
- Linters to configure
- Code to compile

## When Contributing

When updating or extending the documentation:

1. **Follow the structure** — Each failure scenario has a clear format: description, YAML examples, and explanation of effects
2. **Include YAML examples** — All new scenarios should include concrete, copyable go2rtc + FFmpeg configuration
3. **Reference validation** — Add to the validation table in the HOWTO how diagnostic tools should detect the failure
4. **Link to sources** — Maintain references to Frigate docs, go2rtc, and FFmpeg filter documentation

## Domain Context

- **Frigate NVR**: Open-source NVR with IP camera management; uses go2rtc for stream handling
- **go2rtc**: RTSP restreamer embedded in Frigate that can source from FFmpeg
- **AlertaHub Vision**: Technical inspection component that analyzes camera streams for faults (hue histograms, inter-frame differences, luminance, etc.)
- **Key FFmpeg filters used**: `color`, `drawbox`, `colorchannelmixer` for visual faults; bitrate flags for quality faults
