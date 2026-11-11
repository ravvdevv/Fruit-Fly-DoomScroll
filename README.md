# FlyScroll

A fruit-fly-inspired **doomscroller** built on the MaleCNS controller. The fly watches short-form video through modeled photoreceptors, stays on a clip until novelty-compartment MBON activity habituates, and scrolls when interest drops — now with live audio capture, ad detection, keyboard controls, and waveform visualization.

![FlyScroll watching YouTube Shorts](docs/flyscroll-demo.gif)

Based on the recent fruit fly connectome published by [Google](https://blog.google/innovation-and-ai/technology/research/male-fruit-fly-brain-map/).

---

## What it does

1. **Captures video** — frames sampled into ommatidial columns, fed into modeled R1–R8 photoreceptors (luminance + chromatic)
2. **Simulates** — compiled numba LIF spiking network on the MaleCNS connectome (69k visual crop or 167k full CNS)
3. **Computes novelty** — analog feature detectors + smoothed spike rates form a population vector; prediction error = novelty
4. **Tracks interest** — tonic interest = EMA of novelty-MBON rate; anti-Hebbian depression on KC→novelty-MBON synapses causes habituation
5. **Scrolls** — when interest falls below threshold after refractory period, or phasic novelty stays low too long, the fly swipes to the next clip
6. **Hears audio** — WASAPI loopback captures system audio, split into bass/mid/treble bands driving different neural populations; only *changes* in sound drive interest (habituation)

---

## Setup

Python **3.11+**. Several GB of RAM for the full graph.

```sh
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1   # PowerShell
# or: source .venv/bin/activate  # macOS/Linux

pip install -e ".[video,dev]"
```

Connectome feathers should live in `connectome_data/malecns_v1/` (already present). Then prepare the graph:

```sh
flyscroll import                              # Import + normalize MaleCNS connectome
flyscroll prepare --scale visual              # ~70k cells, good for 16GB laptops
flyscroll prepare --scale full                # ~167k cells, intact CNS, more RAM
```

---

## Run

### Live YouTube Shorts (no login required)

Playwright opens public Shorts in headed Chromium. System audio captured via WASAPI loopback.

```sh
pip install -e ".[shorts]"
PLAYWRIGHT_BROWSERS_PATH=0 playwright install chromium

flyscroll serve --scale visual --port 8767 --shorts
# open http://127.0.0.1:8767/
```

**Windows audio:** The fly captures whatever your speakers output. If no loopback device found, install [VB-Cable](https://vb-audio.com/Cable/) and set Chromium output to "Cable Input".

**macOS:** Grant Screen Recording to the app running flyscroll (System Settings → Privacy & Security → Screen & System Audio Recording).

### Local videos

Drop MP4/WebM files into a folder:

```sh
flyscroll serve --scale visual --reels reels --port 8767
```

### Headless smoke test

```sh
flyscroll demo --scale visual --steps 40
```

---

## Keyboard controls (in browser spectator)

| Key | Action |
|-----|--------|
| **Space** | Pause / unpause |
| **→** | Force skip to next reel |
| **↑** | Increase interest threshold (+1.0) |
| **↓** | Decrease interest threshold (−1.0) |
| **A** | Manually mark current video as an ad |

---

## Ad detection

The fly detects ads automatically and skips them:

- **Visual:** Looks for "Skip Ad" buttons, "Ad" labels, ad-related URL patterns
- **Audio:** Flags sustained loudness typical of ad audio
- **Confidence:** Shown as a percentage in the dashboard banner
- **Manual:** Press **A** to mark current video as an ad

When an ad is detected, a red banner appears and the fly scrolls past it immediately.

---

## Dashboard

The spectator at `http://127.0.0.1:8767/` shows:

- **Reel viewport** — current video playing
- **Brain sparks** — phasic spike activity on the MaleCNS connectome (drag to orbit, scroll to zoom)
- **Interest level** — tonic interest meter and phasic novelty
- **Sound bands** — bass / mid / treble / onset levels
- **Audio waveform** — live audio capture visualization
- **Charts** — interest + novelty time series, watch outcomes, average watch time

---

## Layout

| Path | Role |
|------|------|
| `flyscroll/` | Import, preparation, simulation, visual decoder, Shorts integration, spectator |
| `connectome_data/malecns_v1/` | MaleCNS v1.0 source feathers |
| `outputs/flyscroll/` | Prepared `graph.npz` + manifests |
| `reels/` | Drop your own MP4/WebM files here |

---

## License / attribution

FlyScroll code is licensed under the [MIT License](LICENSE). MaleCNS data is **CC BY 4.0**; see [NOTICE](NOTICE) for attribution.

Special thanks to **ranagwho/Fruitfly-Doomscroller** — this project is based on their work, with added improvements including live audio capture, frequency-band analysis, ad detection and auto-skip, keyboard controls, an audio waveform dashboard, and keyboard shortcuts.