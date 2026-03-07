---
title: "FFmpeg Has a Mandelbrot Generator Built In"
date: 2026-01-20
description: "Most people use FFmpeg to convert video files. Few know it ships with a virtual device system called lavfi that generates video from pure math — no input file, no external tools, nothing to install. It's already on your system."
tags: ["linux", "ffmpeg", "video"]
---

Most people use FFmpeg to convert video files. Few know it ships with a virtual device system called **lavfi** that generates video from pure math — no input file, no external tools, nothing to install. It's already on your system.

---

## Check Your FFmpeg Install

```bash
ffmpeg -version
```

If FFmpeg is installed, lavfi is included. That's all you need.

---

## Generate a Mandelbrot and Save It

```bash
ffmpeg -f lavfi -i "mandelbrot=size=640x360:rate=24:maxiter=100" -t 30 -c:v libx264 -crf 23 mandelbrot.mp4
```

What each parameter does:

- `size=640x360` — resolution, keep it low for fast renders
- `rate=24` — 24fps
- `maxiter=100` — iteration depth per pixel, this is your biggest performance lever
- `-t 30` — 30 seconds of output then stop
- `-crf 23` — standard x264 quality

Watch the fps counter in the terminal. On a CPU-only machine `maxiter=100` keeps it moving. Drop to `50` if it's still slow. Raise to `500+` for more fractal detail at the cost of render time.

`maxiter` is the iteration ceiling per pixel — how hard it tries to determine whether a point escapes to infinity. At low zoom levels it barely matters. As the animation zooms deeper into the fractal boundary, low `maxiter` causes the edges to go flat and blocky. If you're rendering a zoom sequence, raise it proportionally.

---

## Play It Directly — No File Written

```bash
ffplay -f lavfi "mandelbrot=size=640x360:rate=24:maxiter=100"
```

`ffplay` is FFmpeg's built-in player. Live output, no mp4, no VLC needed. Kill with `q`.

---

## Add Color Rotation

```bash
ffmpeg -f lavfi -i "mandelbrot=size=640x360:rate=24:maxiter=100" \
  -vf "hue=h=t*20" -t 30 -c:v libx264 -crf 23 mandelbrot_color.mp4
```

`hue=h=t*20` rotates the color spectrum over time as a filtergraph applied on top of the raw lavfi output. `t` is elapsed time in seconds.

---

## Zoom Into the Boundary

The default animation zooms toward a fixed point. You can control where it zooms and how fast:

```bash
ffplay -f lavfi "mandelbrot=size=640x360:rate=24:maxiter=500:start_x=-0.7435:start_y=0.1314:start_scale=3.0:end_scale=0.001"
```

`start_x` and `start_y` set the target coordinate in the complex plane. `start_scale` and `end_scale` control zoom range — smaller `end_scale` means deeper zoom. The boundary region around `-0.7435, 0.1314` is a classic target: high detail, long zoom runway.

---

## The Bigger Point

FFmpeg's lavfi system includes other sources worth exploring:

```bash
ffmpeg -f lavfi -i testsrc2 -t 10 test.mp4        # test pattern
ffmpeg -f lavfi -i "sine=frequency=440" -t 5 tone.wav  # pure sine wave
```

The tools that do the actual work have been on your system the whole time. The GUIs were just hiding them.

---

*Tested on Debian, CrunchBang++. FFmpeg 6.x. CPU-only, no GPU required.*
