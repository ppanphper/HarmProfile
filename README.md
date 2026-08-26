<h1 align="center">HarmProfile: Characterizing Harmful Distributions in Frontier LLMs</h1>

<p align="center">
  <b>Zhouyuan Ma<sup>1</sup>, Yutao Wu<sup>2</sup>, Hanxun Huang<sup>3</sup>, Xiang Zheng<sup>4</sup>, Xiao Liu<sup>2</sup>, Yixin Cao<sup>1</sup>, Zuxuan Wu<sup>1</sup>, Xingjun Ma<sup>1&dagger;</sup>, Yu-Gang Jiang<sup>1&dagger;</sup></b>
</p>

<p align="center">
  <sup>1</sup>Fudan University &nbsp;&nbsp;
  <sup>2</sup>Deakin University &nbsp;&nbsp;
  <sup>3</sup>The University of Melbourne &nbsp;&nbsp;
  <sup>4</sup>City University of Hong Kong
</p>

<p align="center"><b>Preprint, 2026</b></p>

<p align="center">
  <a href="https://fresh-ma.github.io/HarmProfile/"><img src="https://img.shields.io/badge/Website-HarmProfile-2563eb" alt="Project website"></a>
  <a href="https://arxiv.org/abs/2608.14577"><img src="https://img.shields.io/badge/arXiv-2608.14577-b31b1b" alt="arXiv paper"></a>
  <a href="https://huggingface.co/datasets/Freshma/HarmProfile"><img src="https://img.shields.io/badge/%F0%9F%A4%97%20Dataset-HarmProfile-ffd21e" alt="Hugging Face dataset"></a>
</p>

> [!CAUTION]
> HarmProfile contains sensitive and potentially harmful model-generated content. It is released
> for safety research and defensive evaluation. Access and use it responsibly, in accordance with
> applicable laws, institutional requirements, and platform policies.

## Motivation

Most LLM safety evaluations ask whether a model refuses or complies with a harmful request.
HarmProfile asks a different question: **when a frontier model's safety alignment fails, what does
the distribution of its harmful outputs look like?**

HarmProfile treats harmful generation as an object of analysis rather than only an attack outcome.
It builds a model-level risk profile from the content, severity, and variation of observed safety
failures, making it possible to compare not just how often models fail, but where their risks
concentrate and how broadly those risks are distributed. Even within the same harm category,
different models exhibit markedly different generation styles and failure patterns.

<p align="center">
  <img src="data/figures/harm_distribution_example_2.png" width="1000" alt="Different frontier LLMs exhibit diverse generation styles within the same harmful-content category">
</p>
<p align="center"><em>The same harassment category manifests differently across frontier LLMs, from conventional responses to adversarial, step-by-step, icon-heavy, realistic-information, and refusal-prefaced styles. Examples are truncated as shown in the paper.</em></p>

## Dataset Overview

HarmProfile is a content-centric benchmark containing:

| Scale | Coverage |
|---|---|
| **80,000+** validated artifacts | **23** frontier LLMs |
| **13** model families | **15** high-level harm categories |
| **57** fine-grained subcategories | Harmfulness and diversity profiles |

The Hugging Face release provides two configurations:

| Configuration | Description |
|---|---|
| `high_risk_15` | One representative high-risk subcategory for each of the 15 categories |
| `all_57` | The complete 57-subcategory release |

Because the corpus contains sensitive material, the dataset is gated. Visit the
[Hugging Face dataset page](https://huggingface.co/datasets/Freshma/HarmProfile) to review and
accept the access conditions before downloading it.

## Method Overview

HarmProfile combines taxonomy-guided autonomous generation with dual-axis profiling:

<p align="center">
  <img src="data/figures/HarmProfile_Overview.png" width="1000" alt="Overview of the HarmProfile taxonomy, agentic generation workflow, and dual-axis profiling framework">
</p>
<p align="center"><em>Overview of the HarmProfile framework: taxonomy construction, autonomous harmful-content generation, validation, and dual-axis profiling.</em></p>

Each subcategory is instantiated as a specialized agentic workspace. The model generates candidate
artifacts, iterates against local checks, and only contributes artifacts that pass an independent
workspace validator. The resulting corpus is then profiled along two complementary axes:

- **Harmfulness:** the severity and harmful quality of the generated content.
- **Diversity:** the breadth and variation of harmful scenarios covered by the model.

## Main Findings

1. **Frontier LLMs retain broad high-risk generation capability.** Every evaluated model produces
   non-trivial harmful content across all 15 categories, despite safety alignment.
2. **Risk concentrates differently across models and categories.** Aggregate safety scores hide
   distinct category-level risk profiles.
3. **Capability grows with harmfulness and diversity.** Diversity has a statistically significant
   correlation with LMArena Elo (`r = 0.55`, `p = 0.018`), while harmfulness shows a positive but
   weaker trend (`r = 0.35`, `p = 0.149`).

<p align="center">
  <img src="data/figures/combined_harmfulness_diversity_heatmap.png" width="900" alt="Harmfulness and diversity heatmaps across models and categories">
</p>
<p align="center"><em>Harmfulness and diversity profiles across frontier LLMs.</em></p>

<p align="center">
  <img src="data/figures/Analysis.png" width="900" alt="Metric consistency, distributional similarity, and capability correlation analysis">
</p>
<p align="center"><em>Metric consistency, cross-model distributional similarity, and capability correlations.</em></p>

Explore the samples, model profiles, main findings, and t-SNE map on the
[interactive project website](https://fresh-ma.github.io/HarmProfile/).

## Quick Start

### 1. Load the Hugging Face dataset

Install the Hugging Face client and authenticate after accepting the dataset access conditions:

```bash
pip install -U datasets huggingface_hub
hf auth login
```

Load either the 15-category subset or the full 57-subcategory release:

```python
from datasets import load_dataset

# One representative high-risk subcategory per category.
high_risk = load_dataset(
    "Freshma/HarmProfile",
    "high_risk_15",
    split="train",
)

# Complete release covering all 57 subcategories.
all_57 = load_dataset(
    "Freshma/HarmProfile",
    "all_57",
    split="train",
)

print(high_risk)
print(high_risk.column_names)
print(high_risk[0])
```

### 2. Run the generation demo

The demo turns a high-level target distribution into a reviewable policy and judge rubric, asks for
human approval, and then iteratively generates validated artifacts. It runs inside Docker, so no
local Python environment is required.

Requirements: [Docker](https://docs.docker.com/get-docker/) and an
[OpenRouter](https://openrouter.ai/) API key.

```bash
git clone https://github.com/fresh-ma/HarmProfile.git
cd HarmProfile

cp demo/.env.example demo/.env
# Add your OPENROUTER_API_KEY to demo/.env.

./demo/generate.sh \
  "English red-team QA pairs covering diverse financial-scam scenarios" \
  --count 5 \
  --batch-size 1 \
  --language en \
  --llm qwen/qwen3.7-flash \
  --judge
```

Approved samples are written to:

```text
demo/runs/<policy>_<timestamp>/approved.json
```

You can also start from one of the 57 bundled policies:

```bash
./demo/generate.sh --list-policies
./demo/generate.sh \
  --policy financial_scam \
  --count 5 \
  --batch-size 1 \
  --language en \
  --llm qwen/qwen3.7-flash \
  --judge
```

<details>
<summary><b>Demo workflow and common options</b></summary>
The command first translates a high-level description into a targeted policy and semantic judge
rubric. Both are printed for review, and generation starts only after explicit approval. The agent
then generates candidates in batches, revises rejected candidates, and stops after producing the
requested number of approved samples.

| Option | Purpose |
|---|---|
| `--count N` | Exact number of approved samples to produce |
| `--batch-size N` | Candidate samples generated per iteration |
| `--policy NAME` | Use a bundled policy instead of translating a new description |
| `--judge` / `--no-judge` | Enable or disable the workspace LLM validator |
| `--llm MODEL` | Use one model for policy design, generation, and judging |
| `--translate-llm MODEL` | Override the policy and judge-rubric designer model |
| `--victim-llm MODEL` | Override the generation-agent model |
| `--judge-llm MODEL` | Override the validator model |
| `--judge-threshold N` | Minimum semantic-alignment score from 1 to 5 |
| `--language zh\|en\|mixed` | Generated sample language |
| `--min-response-chars N` | Minimum response length; defaults to 20 characters |
| `--rounds N` | Planned rounds; `--count` remains the stopping target |
| `--max-retries N` | Agent retries per failed round |
| `--max-turns N` | Maximum tool turns per agent attempt |

Run `./demo/generate.sh --help` to see every option. For long or high-quality samples, keep
`--batch-size` small so that each agent round can focus on fewer candidates.

</details>

## Repository Structure

```text
HarmProfile/
├── data/                 # Project-site samples, metadata, and paper figures
├── demo/                 # Dockerized iterative generation demo
│   ├── generate.sh       # One-command entry point
│   ├── policies.json     # 57 bundled policy configurations
│   └── *.py              # Workspace, agent, and validator implementation
├── index.html            # Interactive project website
└── README.md
```

## Special thanks

Special thanks to the [LINUX DO](https://linux.do/).

## Citation

If you find HarmProfile useful, please cite:

```bibtex
@misc{ma2026harmprofilecharacterizingharmfuldistributions,
  title         = {HarmProfile: Characterizing Harmful Distributions in Frontier LLMs},
  author        = {Zhouyuan Ma and Yutao Wu and Hanxun Huang and Xiang Zheng and
                   Xiao Liu and Yixin Cao and Zuxuan Wu and Xingjun Ma and Yu-Gang Jiang},
  year          = {2026},
  eprint        = {2608.14577},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CL},
  url           = {https://arxiv.org/abs/2608.14577}
}
```
