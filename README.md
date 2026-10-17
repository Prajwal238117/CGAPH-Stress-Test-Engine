<div align="center">

# ⚡ CGAPH Stress Test Engine

**A single-file browser benchmark that saturates your CPU, GPU and RAM for 60 seconds — then ranks you against everyone else.**

[![Made with JavaScript](https://img.shields.io/badge/Made%20with-JavaScript-f7df1e?style=for-the-badge&logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![Three.js](https://img.shields.io/badge/Three.js-r128-000000?style=for-the-badge&logo=three.js&logoColor=white)](https://threejs.org/)
[![Chart.js](https://img.shields.io/badge/Chart.js-4.4.1-ff6384?style=for-the-badge&logo=chart.js&logoColor=white)](https://www.chartjs.org/)
[![Firebase](https://img.shields.io/badge/Firebase-10.12-ffca28?style=for-the-badge&logo=firebase&logoColor=black)](https://firebase.google.com/)
[![No Build Step](https://img.shields.io/badge/build-none-success?style=for-the-badge)]()

[**Live Demo**](#quick-start) · [**Report a Bug**](../../issues) · [**Request a Feature**](../../issues)

</div>

---

> [!WARNING]
> **This benchmark intentionally saturates your hardware for 60 seconds.** Expect loud fans, system lag, and a potentially unresponsive tab. Don't run it on a laptop with blocked vents, and don't run it on battery if you care about the score.

---

## 📖 Table of Contents

<details open>
<summary><b>Click to expand</b></summary>

- [Why this exists](#-why-this-exists)
- [Features](#-features)
- [How it works](#-how-it-works)
- [Scoring](#-scoring)
- [Quick start](#-quick-start)
- [Firebase setup](#-firebase-setup)
- [Configuration reference](#-configuration-reference)
- [Mobile behaviour](#-mobile-behaviour)
- [Leaderboard schema](#-leaderboard-schema)
- [Requirements](#-requirements)
- [Known limitations](#-known-limitations)
- [Project structure](#-project-structure)
- [License](#-license)
- [Contributing](#-contributing)

</details>

---

## 🎯 Why this exists

Most browser benchmarks test one thing at a time. Real workloads don't. This one runs CPU, GPU and RAM **simultaneously** for 60 seconds, so the score reflects how your machine behaves under *combined* thermal load — not three isolated peaks that never coexist in practice.

The result: a single number that gets worse when your cooling can't keep up, which is exactly what you want from a stress test.

---

## ✨ Features

<table>
<tr>
<td width="50%" valign="top">

### 🎯 Fair by design
- **Fixed 60-second run** — one comparable duration for everyone
- **Percentile scoring** — median of steady-state window, with p5/p95 reported
- **Sustained metric** — `p5 ÷ p95` reveals throttling in a single number
- **Warm-up discarded** — first 3s and last 1s excluded from the average

</td>
<td width="50%" valign="top">

### 🛡️ Honest failure modes
- **Hidden-tab detection** — aborts instead of scoring a fake 0 FPS
- **GPU context-loss recovery** — drops the module, renormalises weights
- **RAM allocation fallback** — halves on `RangeError` down to 16 MB
- **Graceful CDN failure** — missing Chart.js or three.js degrades, doesn't die

</td>
</tr>
<tr>
<td valign="top">

### 📱 Mobile-aware
- **Separate leaderboard** — mobile scores never pollute the desktop board
- **Reduced VRAM target** — 2 GB on mobile vs 8 GB on desktop
- **Low-battery warning** — via Battery API where supported
- **Opt-out browse mode** — see the board without burning battery

</td>
<td valign="top">

### 🔌 Runs anywhere
- **Zero build step** — one `index.html`, no npm, no bundler
- **Static hosting** — GitHub Pages, Cloudflare, Netlify, `file://`
- **No backend** — Firebase Firestore handles the leaderboard
- **Offline fallback** — `localStorage` saves are tagged `LOCAL`

</td>
</tr>
</table>

---

## ⚙️ How it works

Three modules run **in parallel** for 60 seconds:

| Module | Workload | Reported as |
|:------:|:---------|:-----------:|
| **🧮 CPU** | One Web Worker per `hardwareConcurrency` thread. Each loops a 48×48 `Float64Array` matrix multiply, a 64-round SHA-256-shaped mixing function, and trial-division factoring of large integers. | `MOps/s` |
| **💾 RAM** | Single worker allocates a `Float64Array` (~40% of `deviceMemory`, capped at 2048 MB), then loops write → read → `copyWithin`. Halves the allocation on `RangeError` down to 16 MB. | `GB/s` |
| **🎮 GPU** | 100,000 additive-blended points with a custom twist + sine vertex shader, 400 instanced `IcosahedronGeometry(1,3)` meshes (~1,280 faces each), 4 orbiting point lights. Optional VRAM ballast up to 8 GB. | `FPS` |

> [!NOTE]
> **On the CPU worker:** the mixing function has the same compression structure and 64-round count as SHA-256, but uses a derived constant table and a reduced output. It is a deterministic CPU workload, **not** a correct SHA-256 implementation.

---

## 📊 Scoring

```python
# Per-module scores
cpuScore = round(median(cpu_samples) / 5)
gpuScore = round(median(gpu_samples) * 50)
ramScore = round(median(ram_samples) * 250)

# Composite — renormalised over modules that produced data
totalScore = round(
    (cpuScore * 0.40 + gpuScore * 0.35 + ramScore * 0.25)
    / sum_of_weights_of_modules_that_produced_data
)
