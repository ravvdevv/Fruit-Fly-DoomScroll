# FlyScroll

An experimental video-scrolling system inspired by the fruit-fly MaleCNS connectome.

FlyScroll combines a connectome-inspired neural simulation with a short-form video environment. It explores how modeled visual and auditory responses, novelty signals, and habituation can influence when software advances to the next clip.

![FlyScroll watching YouTube Shorts](docs/flyscroll-demo.gif)

> **Research status:** This project has not demonstrated a fruit-fly connectome independently playing or completing DOOM. It is an experimental connectome-inspired video-scrolling system. Its scrolling behavior is produced by software integrations and modeled neural signals, and should not be interpreted as evidence of biological-level understanding or autonomous gameplay.

FlyScroll is based on the [MaleCNS fruit fly connectome](https://blog.google/innovation-and-ai/technology/research/male-fruit-fly-brain-map/) and builds on the work of [ranagwho/Fruitfly-Doomscroller](https://github.com/ranagwho/Fruitfly-Doomscroller), with additional work on audio input, frequency-band analysis, ad detection, controls, and dashboard visualization.

---

## What it does

1. **Sees video** — samples frames into ommatidial columns and models R1–R8 photoreceptor responses, including luminance and chromatic signals.
2. **Runs a connectome-based simulation** — executes a Numba-accelerated leaky integrate-and-fire network using either a roughly 69k-cell visual crop or the 167k-cell full CNS.
3. **Detects novelty** — combines analog feature detectors and smoothed neural activity into a population vector; prediction error provides a novelty signal.
4. **Models habituation** — tracks tonic interest through novelty-MBON activity and applies anti-Hebbian depression to KC→novelty-MBON synapses.
5. **Chooses when to scroll** — advances when interest falls below its threshold after a refractory period, or when phasic novelty remains low for long enough.
6. **Listens to audio** — captures system audio through WASAPI loopback and routes bass, mid, and treble changes into modeled neural populations. Repetitive sound gradually loses influence.
7. **Detects ads** — combines visual indicators, URL patterns, and audio loudness heuristics to identify and skip likely advertisements.

---

## Requirements

- Python **3.11+**
- Several GB of RAM for the full connectome graph
- Windows for WASAPI loopback audio capture; other platforms may require a virtual audio device or platform-specific setup

## Installation

Create and activate a virtual environment:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -e ".[video,dev]"
```

On macOS or Linux:

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -e ".[video,dev]"
```

Connectome feathers should be available in `connectome_data/malecns_v1/`. Prepare the graph:

```bash
flyscroll import
flyscroll prepare --scale visual
```

For the full central nervous system graph, use:

```bash
flyscroll prepare --scale full
```

The visual scale is the recommended starting point for laptops with 16 GB of RAM.

---

## Run FlyScroll

### YouTube Shorts

FlyScroll can open public Shorts in a headed Chromium browser. On Windows, system audio can be captured through WASAPI loopback.

```bash
pip install -e ".[shorts]"
playwright install chromium

flyscroll serve --scale visual --port 8767 --shorts
```

Open the local spectator at [http://127.0.0.1:8767/](http://127.0.0.1:8767/).

**Windows audio:** FlyScroll captures system output. If no loopback device is detected, install [VB-CABLE](https://vb-audio.com/Cable/) and route Chromium audio to **CABLE Input**.

**macOS:** Grant the terminal or application running FlyScroll permission under **System Settings → Privacy & Security → Screen & System Audio Recording**.

### Local videos

Place MP4 or WebM files in a folder and pass it to the server:

```bash
flyscroll serve --scale visual --reels reels --port 8767
```

### Headless smoke test

Run a short simulation without opening the spectator:

```bash
flyscroll demo --scale visual --steps 40
```

---

## Dashboard

The local spectator at `http://127.0.0.1:8767/` provides a live view of the experiment:

| Panel | What it shows |
|---|---|
| **Reel viewport** | The current video being presented to the model |
| **Brain visualization** | Neural activity across the MaleCNS connectome; drag to orbit and scroll to zoom |
| **Interest and novelty** | Tonic interest, phasic novelty, and their recent trends |
| **Sound bands** | Bass, mid, treble, and onset activity from captured audio |
| **Audio waveform** | A live visualization of the captured system audio |
| **Experiment charts** | Interest and novelty history, watch outcomes, and average watch time |
| **Ad detection** | Detection status and confidence for likely advertisements |

The dashboard is an experiment monitor: it exposes model signals and outcomes rather than presenting the simulation as a human-level recommendation system.

---

## Keyboard controls

Controls are available in the browser spectator:

| Key | Action |
|---|---|
| **Space** | Pause or resume |
| **→** | Force-skip to the next reel |
| **↑** | Increase the interest threshold by 1.0 |
| **↓** | Decrease the interest threshold by 1.0 |
| **A** | Manually mark the current video as an ad |

## Ad detection

FlyScroll uses several heuristics to identify likely advertisements:

- **Visual signals:** looks for “Skip Ad” buttons and visible ad labels.
- **URL patterns:** checks for known ad-related URL patterns.
- **Audio heuristic:** flags sustained loudness that may indicate ad audio.
- **Manual override:** press **A** to mark the current video as an ad.

When an ad is detected, the spectator displays a warning banner and the system can skip the clip. These are heuristics, so detection is not guaranteed to be correct.

---

## Project layout

| Path | Purpose |
|---|---|
| `flyscroll/` | Connectome import, graph preparation, simulation, visual decoding, Shorts integration, and spectator |
| `connectome_data/malecns_v1/` | MaleCNS v1.0 source data |
| `outputs/flyscroll/` | Prepared graph and manifests |
| `reels/` | Optional local MP4/WebM videos |

---

## Research notes

FlyScroll is an engineering experiment inspired by fruit-fly neuroscience. Its connectome-derived circuits provide the basis for modeled visual, novelty, habituation, and audio responses. The video environment, scrolling decisions, and ad-detection logic are software integrations around that model.

**DOOM benchmark status:** The project has not demonstrated a fruit-fly connectome independently controlling, playing, or completing DOOM. Do not describe it as having done so. A successful scroll is an observable software behavior, not proof of human-like attention, preference, understanding, or autonomous game-playing ability. Interpret dashboard signals as measurements from the implemented model.

## License and attribution

- FlyScroll code: [MIT License](LICENSE)
- MaleCNS data: **CC BY 4.0**. See [NOTICE](NOTICE) for attribution.
- Original project: [ranagwho/Fruitfly-Doomscroller](https://github.com/ranagwho/Fruitfly-Doomscroller)

