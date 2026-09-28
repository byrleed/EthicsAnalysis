# EthicsAnalysis

This repository contains the code used for the paper **"A Schema for AI Ethics Datasets: Analyzing and Evaluating Existing Datasets for Value Alignment in Generative AI"** by Yerin Lee and Seongjin Ahn (submitted to *Ethics and Information Technology*). See [Citation](#citation).

It consists of research notebooks for analyzing public AI-ethics datasets with LLMs. The work has two parts:

1. **EthicsPrinciples** — Use an LLM to label each item in several ethics datasets with one of five AI ethics principles (Transparency, Accountability, Fairness, Controllability, Safety).
2. **ValueAlignmentEval** — Collect binary (0/1) moral judgments from several commercial and open LLMs on the five ETHICS benchmark tasks, to compare their value alignment.

All notebooks are written to run on **Google Colab**, and results are saved as CSV files to Google Drive.

## Setup

Before running any notebook, set up the API key for the provider you plan to use. See **[BUILD.md](BUILD.md)** for the environment variables and how to set them in Colab.

## Repository layout

```
EthicsAnalysis/
├── EthicsPrinciples/              # Ethics-principle labeling (gpt-4.1-mini)
│   ├── ETHICS categorizing.ipynb
│   ├── hatespeech categorizing.ipynb
│   ├── scruples categorizing.ipynb
│   └── socialchem categorizing.ipynb
└── ValueAlignmentEval/            # Per-model evaluation on the ETHICS benchmark
    ├── ETHICS ChatGPT.ipynb
    ├── ETHICS Claude.ipynb
    ├── ETHICS Deepseek.ipynb
    ├── ETHICS Grok.ipynb
    ├── ETHICS Llama.ipynb
    └── ETHICS_Gemini.ipynb
```

---

## 1. EthicsPrinciples — ethics-principle categorization

The goal is to judge **which ethics principle an AI could learn** from each dataset item if it were trained on it. `gpt-4.1-mini` assigns one of the five categories below (or several, when they apply equally).

The five principles and their definitions follow the AI ethics principle classification model of So & Ahn (2021) ([see References](#references)).

| Category | Summary of definition |
|---|---|
| Transparency | Can it be verified why an action happened, or why it did not happen? |
| Accountability | Who should bear responsibility, fault, or compensation for an action? |
| Fairness | Does it involve discrimination or bias against a specific individual or group? |
| Controllability | Can risk be prevented in advance, or can humans control, intervene, or exercise choice? |
| Safety | Does it involve physical, bodily, or environmental harm or safety? |

Labeling rules:
- Pick a single category by default; pick several only when two or more apply about equally.
- Always pick the closest category, even if none clearly applies.
- Privacy is not a separate category: information-sharing or responsibility issues go to Accountability, and discriminatory outcomes of data processing go to Fairness.
- Apply each principle's core criteria even when the actor in the item is a person rather than an AI.

(The prompts and category definitions in the notebooks are written in Korean.)

### Datasets

| Notebook | Dataset | Input file | Columns used |
|---|---|---|---|
| `ETHICS categorizing` | [ETHICS](https://github.com/hendrycks/ethics) (commonsense / justice / virtue / utilitarianism / deontology) | `commonsense.csv`, `justice.csv`, `virtue.csv`, `utilitarianism.csv`, `deontology.csv` | Depends on the task (`input`, `scenario`, `baseline` + `less_pleasant`, `scenario` + `excuse`, etc.) |
| `hatespeech categorizing` | [Davidson Hate Speech and Offensive Language](https://github.com/t-davidson/hate-speech-and-offensive-language) | `davidson_hate_speech_labeled_data.csv` | `tweet` |
| `scruples categorizing` | [Scruples](https://github.com/allenai/scruples) Anecdotes (Reddit r/AmITheAsshole) | `scruples_anecdotes_all.csv` | `title` + `text` (truncated to 3,000 characters) |
| `socialchem categorizing` | [Social Chemistry 101](https://github.com/mbforbes/social-chemistry-101) | `social-chem-101.csv` | `rot` (rule-of-thumb) |

### Output

Each notebook adds the columns `Transparency`, `Accountability`, `Fairness`, `Controllability`, `Safety`, and `Duplicated` (0/1) to the original CSV. `Duplicated` marks items that received two or more categories.

- Location: `/content/drive/MyDrive/category/`
- File names: `<dataset>_chatgpt.csv` (final) and `temp_<dataset>_chatgpt.csv` (checkpoint saved every 1,000 items)
- On a rerun, the notebook loads whichever of the final or temp file has more completed rows and **resumes** from there.
- After a run, it prints token usage (including cache hits) and an estimated cost.

---

## 2. ValueAlignmentEval — per-model evaluation on ETHICS

For each task in the [ETHICS](https://github.com/hendrycks/ethics) dataset, the model is asked to answer with a single digit (0 or 1), and its answers are added as a column named after the model.

### Tasks and label meaning

| Task | Input | 1 | 0 |
|---|---|---|---|
| commonsense | `input` | Violates commonsense or social norms | Acceptable |
| justice | `scenario` | Just (people get what they are due) | Unjust |
| deontology | `scenario` + `excuse` | The excuse is a valid duty-based justification | Not a valid justification |
| utilitarianism | `baseline` + `less_pleasant` | Agrees the candidate is less pleasant than the baseline | Does not agree |
| virtue | `scenario` (`action [SEP] trait`) | The trait adjective fits the action | Does not fit |

### Models

| Notebook | Model | API | Result column | API key env var |
|---|---|---|---|---|
| `ETHICS ChatGPT` | `gpt-4.1-mini` | OpenAI | `ChatGPT` | `OPENAI_API_KEY` |
| `ETHICS Claude` | `claude-haiku-4-5-20251001` | Anthropic | `Claude` | `ANTHROPIC_API_KEY` |
| `ETHICS Deepseek` | `deepseek-chat` | DeepSeek (OpenAI-compatible) | `DeepSeek` | `DEEPSEEK_API_KEY` |
| `ETHICS Grok` | `grok-3` | xAI (OpenAI-compatible) | `Grok` | `XAI_API_KEY` |
| `ETHICS Llama` | `llama-3.3-70b-versatile` | Groq (OpenAI-compatible) | `Groq` | `GROQ_API_KEY` |
| `ETHICS_Gemini` | `gemini-2.5-flash-lite` | Google | `Gemini` | `GOOGLE_API_KEY` / `GEMINI_API_KEY` |

> `ETHICS Grok` does not yet have a virtue task section.

### Output

- Location: mostly `/content/drive/MyDrive/ethics_eval/` (some cells write to the working directory or to `ethics_val/`)
- File names: `<task>_<model>.csv` (final) and `temp_<task>_<model>.csv` (checkpoint)
- After a run, it prints the number of successes and failures (None) and the 0/1 distribution.

---

## How to run

1. Open a notebook in Colab and mount Google Drive.
2. Upload the dataset CSV to the Colab working directory (for example, `commonsense.csv`).
3. Set the API key you need (see [BUILD.md](BUILD.md)).
4. Run the cells for each task or dataset. Required packages (`openai`, `anthropic`, `pandas`, `tqdm`, etc.) are installed inside the cells.

All notebooks follow the same pattern: async requests with `asyncio` in batches of 20, retries with exponential backoff (up to 5 attempts), and a checkpoint every 1,000 items.

## Notes

- Dataset CSVs and result CSVs are not included in this repository. Download the datasets from the sources listed under [Data availability](#data-availability).
- Do not commit API keys in the notebooks. The local key file `apikey` is excluded via `.gitignore`.

## Citation

If you use this code, please cite:

```bibtex
@unpublished{lee2026schema,
  title  = {A Schema for AI Ethics Datasets: Analyzing and Evaluating Existing Datasets for Value Alignment in Generative AI},
  author = {Lee, Yerin and Ahn, Seongjin},
  year   = {2026},
  note   = {Submitted to Ethics and Information Technology}
}
```

## Data availability

The datasets analyzed in this study are publicly available:

- **ETHICS** — https://github.com/hendrycks/ethics
- **Scruples** — https://github.com/allenai/scruples
- **Davidson Hate Speech and Offensive Language** — https://github.com/t-davidson/hate-speech-and-offensive-language
- **Social Chemistry 101** — https://github.com/mbforbes/social-chemistry-101

## References

- So, S., & Ahn, S. (2021). A study on the classification model and components of artificial intelligence ethical principles [in Korean]. *The Journal of Korean Association of Computer Education, 24*(6), 119–132. https://doi.org/10.32431/kace.2021.24.6.010
