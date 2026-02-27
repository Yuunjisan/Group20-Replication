# Energy Consumption: Google Meet vs Microsoft Teams

Comparing client-side energy consumption of video calls on Google Meet and Microsoft Teams using automated browser bots and hardware-level energy measurement.

**Course:** Sustainable Software Engineering (SSE) — TU Delft 2026

---

## Prerequisites

You need these installed on your machine before starting:

| Tool | Version | Check command | Install |
|------|---------|---------------|---------|
| **Python** | 3.10+ | `python3 --version` | [python.org](https://www.python.org/downloads/) |
| **Google Chrome** | Latest | Check in Chrome menu → About | [google.com/chrome](https://www.google.com/chrome/) |
| **Rust** | Latest | `cargo --version` | [rustup.rs](https://rustup.rs/) |
| **pip** | Latest | `pip3 --version` | Comes with Python |

> **Note:** ChromeDriver is installed automatically by `webdriver-manager` — you don't need to install it manually.

---

## Setup (step by step)

### 1. Create a Python virtual environment

```bash
python3 -m venv venv
source venv/bin/activate      # macOS / Linux
# venv\Scripts\activate       # Windows (PowerShell)
```

### 2. Install Python dependencies

```bash
pip install -r requirements.txt
```

This installs: `selenium`, `webdriver-manager`, `psutil`, `pandas`, `matplotlib`, `seaborn`, `numpy`, `python-dotenv`.

### 3. Build EnergiBridge (energy measurement tool)

[EnergiBridge](https://github.com/tdurieux/EnergiBridge) is a cross-platform tool that measures **real system power** (Watts) during experiments.

```bash
# Install Rust if you don't have it yet
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"

# Clone and build EnergiBridge
git clone https://github.com/tdurieux/EnergiBridge.git
cd EnergiBridge
cargo build -r
cd ..
```

Verify it works:

```bash
EnergiBridge/target/release/energibridge --summary -m 2 sleep 2
# Should print: "Energy consumption in joules: XX.XX for 2.XX sec of execution."
```

### 4. Create meeting links

**Create your meetings:**
- **Google Meet:** Go to [meet.google.com](https://meet.google.com), click "New meeting" → "Start an instant meeting". Copy the URL (looks like `https://meet.google.com/abc-defg-hij`).
- **Microsoft Teams:** Go to [teams.live.com](https://teams.live.com), click "Meet now" → "Get a link to share". Copy the URL (looks like `https://teams.live.com/meet/123456789?p=xxxxx`).

## Running Experiments

### For setting up the environment for adding bot participants. 

> This script is for windows users, please adapt it to have command instead of ctrl if you use a mac.
```bash
cd src 

python set_up_participant_bots.py --platform meet --url "PASTE_YOUR_MEET_LINK_HERE" --bots 10

python set_up_participant_bots.py --platform teams --url "PASTE_YOUR_TEAMS_LINK_HERE" --bots 10
### Full experiment (30 runs per platform)
```
### For running the experiments for each platform
> This script is for mac users, please adapt it to have ctrl instead of command if you use windows.
>Copy your Meet and Teams link to the respective fields. Configure Teams and Meet so that all participants have permission to join and access everything they need.

> The results will be saved to the /data folder, you can see it from there.

```bash
cd src

python run_experiment.py \
  --meet-url "PASTE_YOUR_MEET_LINK_HERE" \
  --teams-url "PASTE_YOUR_TEAMS_LINK_HERE" \
  --repeats 30 \
  --duration 120 \
  --visible
```
### For the test statistics

```bash
cd data
# for outlier detection
python outlier.py
# for running the statistics
python stats.py
# for the plots
python avg_cpu_percentage_plot.py
python energy_cpu_joules_plot.py
python total_net_recv_mb_plot.py.py
python total_net_sent_mb_plot.py
python avg_power_watts_plot.py
```

## Project Structure

```
SSE-26/
├── requirements.txt          # Python dependencies
├── README.md                 # This file
├── EnergiBridge/             # Energy measurement tool (built from source)
│   └── target/release/energibridge
├── src/
│   ├── bot_manager.py        # Selenium bots that join Meet/Teams calls
│   ├── energy_measure.py     # Energy measurement wrapper (EnergiBridge)
│   ├── run_experiment.py     # Runs full experiment  statistics from data
├── data/                     # Raw experiment results (CSV per run)
└── figures/                  # Generated charts (PNG)
```