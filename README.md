# HABS Neuro-AI Hackathon

**Build advanced AI models on real EEG data and contribute to the next generation of neurotechnology.**

Université Paris-Saclay · 2–30 November 2026 · [hackathon.habs.ai](https://hackathon.habs.ai/)

You write `process.py`. HABS runs it on an H100 against the full dataset, and your
results come back to this repository.

---

## 1 · Setup, once

|   |   |
| - | - |
| **Copy this repo** | **Use this template** → *Create a new repository*. Call it anything. |
| **Join Slack** | [**habshackathon.slack.com**](https://join.slack.com/t/habshackathon-pmd8697/shared_invite/zt-4ey1iblfn-pGBe0yROKQY0oAKBvCcn3w) — mentors, announcements and everyone else are there. |
| **DM us your repo** | Send `your-name/your-repo`, exactly as it appears in the URL. Your **team key** and **results link** come back in that DM. |
| **Paste the key** | In [`.github/workflows/submit.yml`](.github/workflows/submit.yml), replace `PASTE-YOUR-TEAM-KEY-HERE`, keeping the quotes. |

## 2 · The data

📖 **[DATASET.md](DATASET.md)** — 384 participants, three tasks, EEG with ECG, skin
conductance, respiration, gaze and pupil size, plus trial labels and questionnaires.
**Read it before you write anything.**

|   |   |
| - | - |
| [`sample/`](sample) | P001, day 1 — already here. Enough to write code against. |
| [🤗 the full dataset](https://huggingface.co/datasets/HABSteam/multimodal-eeg-emotion) | Pull what you want onto your own machine: one file, one folder, or all 385 GB. |
| `/data` | The whole dataset, mounted read-only on the GPU. Nothing to download. |

Access is per person. Open the dataset page with a Hugging Face account, accept
the terms, and HABS approves your request — so ask early, not the night before.

```bash
pip install huggingface_hub && hf auth login

hf download HABSteam/multimodal-eeg-emotion --repo-type dataset --local-dir ./data \
    --include "CURATED/P001/*"                 # one participant
#   --include "CURATED/*/*resting-state*"      # one task, everyone
#   --include "*.csv" "scripts/*"              # the tables and the loader only
#   (no --include)                             # everything, 385 GB
```

You do not need any of this to submit a job — the GPU already has the dataset.
It is for writing the code in the first place.

## 3 · Your code

[`process.py`](process.py) is the only file you have to change.

```python
def process(data_dir: Path, out_dir: Path) -> None:
```

We call it once. `data_dir` is the dataset root, read-only and identical for every
team. Everything you write into `out_dir` is kept and is yours.

Put dependencies in [`requirements.txt`](requirements.txt) — CUDA and PyTorch are
already in the image. For examples and setup instructions, see the
[starting notebooks](starting_notebooks/README.md).

## 4 · Run it on your machine

```bash
pip install -r requirements.txt
pip install "habs-hack[storage] @ git+https://github.com/HABS-BRAIN/habs-hack@main"

habs run --data ./sample --out ./out
```

Same contract, no queue. Worth doing before every submission.

## 5 · Run it on a GPU

**Actions** → **HABS** → **Run workflow**. That is the submission — pushing does not
spend GPU time, so commit as often as you like.

|   |   |   |
| - | - | - |
| **build your image** | red here means `requirements.txt` | seconds |
| **run on a GPU** | red here means your code; the log is your own output | minutes to an hour |


## 6 · Your results

Open them with your **results link**:

- **[Azure Storage Explorer](https://azure.microsoft.com/products/storage/storage-explorer)**
  → *Connect to a resource* → *Blob container* → *SAS URL* → paste.
  `out/` holds a folder per job, `logs/` the full output.
- or `export HABS_SAS='<your link>'`, then `habs status` · `habs watch` · `habs ls` ·
  `habs logs` · `habs get`.

---

**Stuck?** Contact your mentor on
[Slack](https://join.slack.com/t/habshackathon-pmd8697/shared_invite/zt-4ey1iblfn-pGBe0yROKQY0oAKBvCcn3w).

Good luck!
