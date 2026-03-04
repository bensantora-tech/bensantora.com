---
title: "Go and the Lazarus Engine"
date: 2026-03-04
---

## What Go Does in the Lazarus Engine

Imagine you have an old car that still runs perfectly, but every new gas station only sells a special new fuel it can't use. The car isn't broken — it's just been abandoned by the system around it. That's exactly what happens to old computers.

Go is the solution to that problem. Here's how.

---

### The Dependency Problem (Why Old Computers Get Stuck)

Most programs don't actually work alone. They're like a chef who refuses to bring their own knives — they show up and expect the kitchen to already have everything they need. These "knives" are called libraries, and modern programs need dozens of them.

On an old computer, those libraries are outdated or missing entirely. The new program shows up, looks around, and just quits. This is called a dependency failure, and it's why a perfectly good 2010 laptop often can't run 2026 software.

---

### What Go Does Differently

Go is unusual because it lets you pack everything the program needs inside the binary itself. It's like a chef who shows up with their own knives, cutting board, pots, and ingredients — they don't need anything from the kitchen.

This is called static linking. The binary we build carries its own "heart and lungs" and only needs one thing from the computer: the Linux kernel, which even very old machines have.

The magic command that makes this happen is:

    CGO_ENABLED=0 go build -ldflags="-s -w" -trimpath -o lazarus-audit

Breaking that down in plain English:
- CGO_ENABLED=0      — "don't borrow anything from the system, pack it all yourself"
- -s -w              — "throw away the notes and labels we don't need, shrink the package"
- -trimpath          — "clean up leftover file path info from your computer"
- -o lazarus-audit   — "name the finished package lazarus-audit"

The result is a single file, about 1.2 MB, that runs on almost any Linux machine ever made.

---

### How the Audit Code Actually Works

The lazarus-audit program does three things: it reads some special files the Linux kernel keeps,
figures out what the hardware can do, and prints a verdict.

Reading /proc/cpuinfo — Linux keeps a constantly updated text file at this address that describes
your CPU in plain text. Go opens it like any normal text file and reads it line by line, looking
for two things: the CPU's name, and a list of its special abilities called flags.

What are flags? Think of them like merit badges on a scout's uniform. Each badge means the CPU
knows a special trick. The two badges we care about are:
- aes — the CPU can scramble and unscramble data really fast (useful for encryption and security)
- avx — the CPU can do many math calculations at the same time instead of one at a time
         (useful for video, audio, and modern apps)

The ARM problem we fixed — older phones, routers, and small devices use a different type of CPU
called ARM. ARM uses a different word for its merit badge list: "Features" instead of "flags".
The original code only looked for "flags", so it would always tell an ARM device it was BRONZE
even if it was perfectly capable. We fixed that.

Reading /proc/meminfo — same idea, different file. Linux keeps a running count of how much RAM
the machine has here. Go reads the first relevant line, grabs the number, and converts it from
kilobytes to gigabytes so it's human-readable.

---

### The Verdict System

Once Go has collected all the badges, it makes a decision:

    Both aes AND avx  →  GOLD   — this machine can handle anything
    aes but no avx    →  SILVER — capable but has limits
    neither           →  BRONZE — stick to lightweight tools

This is the classification system — the whole point of the audit. Instead of just dumping raw
data at you, it tells you in plain terms what the machine is good for.

---

### The Cross-Compilation Trick

Here's one of Go's most powerful features. The program was written and compiled on one machine,
but it can be built to run on a completely different type of machine — without ever touching it.

    GOOS=linux GOARCH=arm go build ...

This tells Go: "build this program as if you were on an ARM device." Go handles all the
translation internally. This is how one developer on a modern laptop can produce binaries for
a 2008 router sitting in someone's closet across the world.

---

### Why This Matters

Every step of this project — the static binary, the /proc file reading, the cross-compilation —
was chosen specifically because old hardware is fragile and unforgiving. Go is one of the very
few modern languages where all of these pieces work together cleanly, without needing a
complicated build system or a team of engineers.

The result is a single file you can copy onto almost any Linux device ever made, and it just runs.
That's the whole mission.

---

# Lazarus Engine

> *Software should serve the machine, not the other way around.*

A zero-dependency hardware audit tool built in Go. Designed to run on machines that modern tooling has abandoned — broken package managers, ancient kernels, ARM routers, discarded NAS boxes. If it runs Linux, this runs on it.

---

## Why This Exists

Modern software creates a **Dependency Death Spiral**: you can't run the app without the library, you can't update the library without the OS, and you can't update the OS without buying new hardware. Billions of functional machines end up in landfills because of software rot, not hardware failure.

The Lazarus Engine breaks that chain. A single static binary, no shared libraries, no runtime dependencies — just direct syscalls to the Linux kernel.

---

## What It Does

`lazarus-audit` probes the system and classifies hardware into one of three tiers:

| Status | Criteria | Recommended Path |
|--------|----------|-----------------|
| 🥇 GOLD | AES + AVX (or ARM equivalents) | Full modern FOSS stack |
| 🥈 SILVER | AES only | Most tasks; AVX workloads may struggle |
| 🥉 BRONZE | Neither | Minimalist tools: ffplay, aplay, busybox |

**Sample output:**
```
--- [LAZARUS ENGINE: HARDWARE AUDIT] ---
[CPU] Intel(R) Core(TM) i5-3320M CPU @ 2.60GHz (amd64)
[RAM] 3.73 GB
[OS ] Linux Kernel 5.15.0-91-generic
----------------------------------------
[STATUS] GOLD: Modern instruction sets detected.
[ACTION] This machine can handle modern FOSS effortlessly.
```

---

## Build

### Standard build
```bash
go build -o lazarus-audit
```

### Extreme Shrink (recommended for constrained hardware)
```bash
# Strips symbol tables, debug info, and dynamic library linkage
CGO_ENABLED=0 go build -ldflags="-s -w" -trimpath -o lazarus-audit

# Optional: compress binary further (requires upx)
upx --brute lazarus-audit
```

The shrink pipeline targets machines with 2GB RAM or mechanical hard drives where binary size and disk I/O genuinely matter. Typical output: **~1.2 MB**.

### Cross-compilation (edge devices)
```bash
# ARM (Raspberry Pi, old routers)
GOOS=linux GOARCH=arm GOARM=6 CGO_ENABLED=0 go build -ldflags="-s -w" -o lazarus-audit-arm

# ARM64 (NAS boxes, newer SBCs)
GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build -ldflags="-s -w" -o lazarus-audit-arm64

# MIPS (many consumer routers)
GOOS=linux GOARCH=mips GOMIPS=softfloat CGO_ENABLED=0 go build -ldflags="-s -w" -o lazarus-audit-mips
```

---

## Architecture Support

The audit correctly handles both x86/x86_64 and ARM/MIPS flag formats:

- **x86/x86_64** — reads `flags` field from `/proc/cpuinfo`
- **ARM/MIPS** — reads `Features` field; maps `aes-ce`/`pmull` → AES tier, `asimdhp`/`sve` → AVX tier

This means the GOLD/SILVER/BRONZE classification is meaningful on the edge devices this tool is designed for, not just on desktops.

---

## The Mission: Resourceful Computing

This tool is part of a larger framework. The three-step restoration process:

1. **Audit** — run `lazarus-audit` to know what you're working with
2. **Strip** — remove the desktop environment; reclaim RAM
3. **Restore** — deploy static binaries to turn "junk" into a dedicated media server, VPN gateway, or network ad-blocker

Full writeup: [Binary Resurrection](https://github.com/bensantora-tech/lazarus-engine)

---

## Requirements

- Linux (any kernel version with `/proc` filesystem)
- Go 1.18+ to build
- No runtime dependencies

---

## License

GPLV-3

## Author

Ben Santora
