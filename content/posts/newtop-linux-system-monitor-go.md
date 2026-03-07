---
title: "newtop — A Lean System Monitor Written in Go"
date: 2026-03-07
description: "newtop is a terminal-based Linux system monitor written in Go with zero external dependencies. It reads directly from /proc and /sys, renders per-thread CPU bars, memory, disk I/O, network rates, and top processes — all in a color-coded, terminal-width-aware display that updates every second."
tags: ["linux", "go", "tools", "terminal"]
---

`top` ships on every Linux system. `htop` is better. But both carry more history than features — ncurses dependencies, config files, compiled C that assumes things about your system. I wanted something simpler: a single binary, no external packages, reads `/proc` and `/sys` directly, renders to the terminal and gets out of the way.

That's newtop.

---

## Install

```bash
git clone https://github.com/bensantora-tech/newtop
cd newtop
go build -o newtop .
./newtop
```

Or if you have Go installed and want it in your path:

```bash
go install github.com/bensantora-tech/newtop@latest
```

`go install` downloads, compiles, and drops the binary into your `$GOPATH/bin` — usually `~/go/bin`, which is typically on your `$PATH`. The `@latest` tells Go's module system which version to fetch; it's required syntax, not optional. After that, `newtop` works from anywhere.

No `apt install`, no shared libraries, no daemon running in the background. The binary is self-contained.

---

## What You See

newtop shows six things, refreshed every second:

**CPU** — per-thread utilization bars, per-thread frequency in MHz, and package temperature. Each bar is color-coded: green below 50%, yellow 50–80%, red above 80%. The display automatically adjusts to one, two, or four columns depending on your thread count and terminal width.

**MEM** — used, total, available, and cached — with the same bar and color thresholds.

**DISK** — read and write rates in KB/s for every physical disk. Partitions are filtered out; only whole disks show.

**NET** — RX/TX rates per interface. Rates above 1 MB/s are displayed as MB/s automatically. Loopback is excluded.

**PROCESSES** — top processes by CPU usage, sorted descending. The list scales to your terminal height — it fills exactly the rows available after the fixed sections. On wide terminals (≥120 columns) it switches to a two-column layout.

**Header and footer** — CPU model, thread count, total RAM, and the current time. All of it fits to whatever width your terminal reports.

---

## No Hardcoded Anything

At startup, newtop detects your hardware rather than assuming it:

- CPU model and thread count from `/proc/cpuinfo` and `/proc/stat`
- Terminal dimensions via `TIOCGWINSZ` ioctl, re-queried every tick so resize works
- CPU temperature from the `x86_pkg_temp` thermal zone, with a fallback to `thermal_zone0` if that's not present
- Per-core frequency from `/sys/devices/system/cpu/cpu*/cpufreq/scaling_cur_freq`

There are no hardcoded paths to specific hardware, no assumptions about core count, no expected screen width. A 4-core machine and a 32-core machine both get a sensible layout.

---

## How CPU% Is Calculated

Per-process CPU percentage is:

```
cpu% = (delta_ticks / (elapsed_seconds × 100)) × 100
```

`delta_ticks` is the change in `utime + stime` from `/proc/[pid]/stat` between two samples taken one second apart. `100` is `CLK_TCK` — jiffies per second, standard on x86 Linux. This is the same method `top` and `htop` use.

Per-thread aggregate CPU percentage comes from `/proc/stat` — the same delta approach, comparing idle vs. active jiffies across the sample interval.

---

## Zero Dependencies

The `go.mod` lists one module: the standard library. Nothing from pkg.go.dev, no vendored packages. The whole thing is four files:

```
main.go    — startup, main loop, signal handling
sys.go     — hardware detection, temperature, frequency, disk and network I/O
proc.go    — process sampling and CPU% calculation
render.go  — terminal rendering: bars, colors, layout
```

If you can read Go, you can read all of newtop in an afternoon. Every number on screen traces back to a specific `/proc` or `/sys` read.

---

## Why Not Just Use htop?

htop is fine. If it's already on your system and does what you need, use it. The point of newtop isn't to replace it — it's to have something I fully understand, that I can modify without touching ncurses, and that compiles to a single binary I can drop on any Linux box. Sometimes the right tool is the one you built yourself.

---

*Tested on Debian/CrunchBang++. Go 1.21+. Linux only — reads /proc and /sys directly.*
