# Build & Setup

How to prepare the environment before running the notebooks in this repository. For what each notebook does, see [README.md](README.md).

## API keys

Every notebook reads its API key from an environment variable. Set the key for the provider you plan to use before running any other cell.

| Environment variable | Provider | Used by |
|---|---|---|
| `OPENAI_API_KEY` | OpenAI | All `EthicsPrinciples` notebooks, `ETHICS ChatGPT` |
| `ANTHROPIC_API_KEY` | Anthropic | `ETHICS Claude` |
| `DEEPSEEK_API_KEY` | DeepSeek | `ETHICS Deepseek` |
| `XAI_API_KEY` | xAI | `ETHICS Grok` |
| `GROQ_API_KEY` | Groq | `ETHICS Llama` |
| `GOOGLE_API_KEY`, `GEMINI_API_KEY` | Google AI Studio | `ETHICS_Gemini` (set both to the same key; different cells read different names) |

### Option A: Colab Secrets (recommended)

1. In Colab, open the **Secrets** panel (key icon in the left sidebar).
2. Add a secret whose name matches the environment variable above (for example, `OPENAI_API_KEY`) and paste your key as the value.
3. Turn on **Notebook access** for that secret.

The `EthicsPrinciples` notebooks and `ETHICS Deepseek` load the key from Colab Secrets automatically and stop with an error if it is missing. For the other notebooks, load it yourself in the first cell:

```python
import os
from google.colab import userdata

os.environ["ANTHROPIC_API_KEY"] = userdata.get("ANTHROPIC_API_KEY")
```

### Option B: set it in the first cell

Most `ValueAlignmentEval` notebooks start with a placeholder cell like this:

```python
import os
os.environ["ANTHROPIC_API_KEY"] = "API_KEY"
```

Replace `"API_KEY"` with your key for the current session only. **Change it back to the placeholder before saving or committing the notebook**, so the key does not end up in the repository.

The local key file `apikey` is excluded via `.gitignore`; keep any other key files out of the repository as well.
