# USF Admissions Chatbot

This project is a lightweight **USF admissions chatbot** built for an Advanced Generative AI Development lab. It:

- Scrapes a few USF webpages (minimal and respectful)
- Builds a compact “USF INFORMATION CONTEXT” block from the scraped pages
- Queries multiple **local LLMs via Ollama** (`/api/generate`)
- Computes quick automatic metrics and an LLM-as-judge score
- Saves everything to a CSV for easy comparison across models

## Files

- `USF_Lab1_Assignment_nishat.ipynb` — main notebook (pipeline + evaluation)
- `usf_chatbot_results_nishat.csv` — output results from running the notebook
- `README.md` — this file

## What the notebook does (pipeline)

1. **Scrape a few USF pages**
   - Uses `requests` + `BeautifulSoup` to fetch pages.
   - Removes boilerplate (scripts, styles, nav, footer, header) and extracts main text.
   - Includes a small delay between requests to avoid stressing servers.

2. **Build a context block**
   - Constructs a text block:
     - `USF INFORMATION CONTEXT`
     - Per-source snippets with numbered sources like `[1]`, `[2]`, `[3]`
   - Truncates the context to stay within a character budget.

3. **Ask multiple models through Ollama**
   - Sends a prompt that includes:
     - The context block
     - The user question (“TASK”)
     - Formatting instructions (bullets + citations + References section)
   - Records:
     - latency
     - response length
     - approximate tokens (simple chars/4 approximation)

4. **Score each response**
   - **Relevance:** TF-IDF cosine similarity between the question and the response
   - **Coherence:** heuristic based on sentence/paragraph structure
   - **Factual accuracy (heuristic):** mostly based on the number of in-text citations
   - **Citation counting:** extracts `[n]` style citations (ignores the References section)

5. **LLM-as-judge evaluation**
   - Uses a chosen model (default: `llama3.1:8b`) to score each answer (1–5) on:
     - groundedness
     - specificity
     - citations
     - clarity

6. **Save output**
   - Writes results to `usf_chatbot_results_nishat.csv`.

## Requirements

### 1) Ollama
- Install Ollama (macOS/Windows/Linux) and make sure the server is running.
- Pull the models used in the notebook (defaults shown below).

Example:
```bash
ollama pull llama3.2:3b
ollama pull mistral:7b
ollama pull llama3.1:8b
```

Ollama should be reachable at:
- `http://localhost:11434/api/generate`

### 2) Python packages
The notebook uses common Python libraries:
- `numpy`
- `pandas`
- `requests`
- `beautifulsoup4`
- `scikit-learn`

Install:
```bash
pip install numpy pandas requests beautifulsoup4 scikit-learn
```

## How to run

### Option A (recommended): Run the notebook
1. Start Ollama.
2. Open `USF_Lab1_Assignment_nishat.ipynb` in Jupyter/Colab/VS Code.
3. Run all cells top-to-bottom.
4. The notebook will generate `usf_chatbot_results_nishat.csv`.

### Option B: Run from the command line (convert notebook to script)
```bash
jupyter nbconvert --to python USF_Lab1_Assignment_nishat.ipynb
python USF_Lab1_Assignment_nishat.py
```

## Configuration

In the notebook, you can change:

- **Models to compare**
  ```python
  MODELS = [
      "llama3.2:3b",
      "mistral:7b",
      "llama3.1:8b",
  ]
  ```

- **USF URLs to scrape**
  ```python
  USF_URLS = [
      "https://www.usf.edu/admissions/freshmen/admission-information/academic-requirements.aspx",
      "https://www.usf.edu/about-usf/",
      "https://www.usf.edu/facilities/service-center/index.aspx",
  ]
  ```

- **Test prompts** (the questions evaluated)
  - Admission requirements
  - Campus facilities
  - Research opportunities
  - Mental health / well-being
  - Career services

## Output: `usf_chatbot_results_nishat.csv`

Each row is one *(model, prompt)* result with the following columns:

- `model` — Ollama model name (e.g., `llama3.2:3b`)
- `prompt` — user question
- `response` — model’s answer (should include `[n]` citations and a References section)
- `latency_seconds` — request latency
- `response_length_chars` — number of characters in the response
- `response_tokens_approx` — rough token estimate (chars/4)
- `success` — whether the call returned a response
- `context` — the full context block used for that run
- `relevance` — TF-IDF cosine similarity (0–1)
- `coherence` — structure-based heuristic score (0–1)
- `factual_accuracy` — citation-based heuristic score (0–1)
- `num_of_citations_found` — number of in-text `[n]` citations in the answer body
- `llm_judged_scores` — a dictionary-like field with 1–5 scores:
  - `groundedness`, `specificity`, `citations`, `clarity`

## Notes / limitations

- The “factual_accuracy” field is **heuristic** (mainly citation counting). It is not a full fact-checking system.
- Citation mapping assumes `[1]` corresponds to the first URL in `USF_URLS`, `[2]` to the second, etc.
- Scraping is intentionally small-scale (few pages + delays). If you expand URLs, keep it responsible and consider caching.

## Credits

Created by Nishat Nayla Labiba for the USF Advanced Generative AI Development lab assignment.
