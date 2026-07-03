# LLM-Aided OCR

**Live site:** **[omprxkash.github.io/llm-aided-ocr](https://omprxkash.github.io/llm-aided-ocr/)**

Turn scanned PDFs into clean, accurate text. This project takes the raw output from Tesseract OCR and passes it through a large language model to fix the usual OCR errors — split words, misread characters, stray page numbers — and reformat the result as either plain text or proper Markdown.

It works with OpenAI or a fully local Llama model. There's a Flask web app for uploads, a CLI for batch jobs, and a Docker setup if you want GPU acceleration.

The project website (`index.html` at the root) walks through the pipeline, shows a live before/after comparison from a 1975 Warren Buffett letter, and links to every artefact in `docs/`.

---

## Why this exists

Tesseract is good, but it isn't perfect. Scanned documents usually come back with errors like `fo` instead of `To`, `hank` instead of `bank`, and broken page breaks scattered throughout. Cleaning that up by hand is painful for anything longer than a few pages.

An LLM is well-suited to fixing exactly this kind of noise. It understands context, recognises that `fo control promises` should be `To control promises`, and can reformat the whole document at the same time.

That's the whole idea.

---

## What it does

- Converts each page of a PDF into an image with `pdf2image` + Poppler.
- Pre-processes the image (greyscale, Otsu threshold, dilation) with OpenCV before OCR.
- Runs Tesseract on every page in parallel via a thread pool.
- Splits the combined OCR output into overlapping chunks at paragraph and sentence boundaries.
- Sends each chunk to the LLM with a structured correction prompt — fixes typos, joins broken words, removes page numbers, keeps headings intact.
- Optionally runs a second pass to reformat the corrected text as Markdown.
- Reassembles the chunks and writes both the raw OCR (`*__raw_ocr_output.txt`) and the cleaned result (`*_llm_corrected.md`) to disk.
- Asks the LLM to score the final output 0–100 against the raw text and gives a short explanation.

The Flask web app also produces a summary, a French translation, a named-entity list, and audio versions of all of these via gTTS.

---

## Repo layout

```
llm-aided-ocr/
├── src/                            # Python source
│   ├── llm_aided_ocr.py            # Pipeline + Flask web app
│   └── llm-aided-ocr-cli.py        # Command-line wrapper
├── notebooks/
│   └── OCR_phase1.ipynb            # Early experiments
├── docs/
│   ├── research/
│   │   ├── Research_paper_OCR_LLM.docx
│   │   └── 21BCE1950_53074.pdf
│   ├── patents/
│   │   └── PATENT_OCR_LLM_17_mar.docx
│   └── presentations/
│       ├── capstone_ppt.pptx
│       └── poster_llm_ocr.pptx
├── samples/                        # A Warren Buffett letter, raw OCR, and LLM-corrected output
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── index.html                      # Project website
└── README.md
```

---

## Requirements

You need three things installed on your machine before the Python deps will be useful.

1. **Python 3.8 or higher.**
2. **Tesseract OCR.** Install from the [Tesseract releases](https://github.com/tesseract-ocr/tesseract) and make sure `tesseract` is on your PATH. Grab the English language pack (`eng.traineddata`) at minimum.
3. **Poppler.**
   - Linux: `sudo apt-get install poppler-utils`
   - macOS: `brew install poppler`
   - Windows: download the Poppler binaries, unzip, and add the `bin/` folder to your PATH.

If you want to run the local Llama option on a GPU, you'll also need CUDA and `llama-cpp-python` built with CUDA support:

```bash
CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install llama-cpp-python
```

---

## Setup

```bash
git clone https://github.com/omprxkash/llm-aided-ocr.git
cd llm-aided-ocr

python -m venv venv
source venv/bin/activate            # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

Then create a `.env` file in the repo root with your provider settings.

```env
# Pick one: API or local
USE_LOCAL_LLM=False
API_PROVIDER="OPENAI"

# OpenAI
OPENAI_API_KEY="sk-..."
OPENAI_COMPLETION_MODEL="gpt-4o-mini"

# Local Llama (only if USE_LOCAL_LLM=True)
DEFAULT_LOCAL_MODEL_NAME="Llama-3.1-8B-Lexi-Uncensored_Q5_fixedrope.gguf"
LOCAL_LLM_CONTEXT_SIZE_IN_TOKENS=2048
USE_VERBOSE=False
```

The local model file will be downloaded automatically on first run if it isn't already in `./models/`.

---

## Running it

### Web app

```bash
python src/llm_aided_ocr.py
```

Open `http://127.0.0.1:5000` in your browser, upload a PDF, and you'll get back the cleaned text, a summary, a French translation, the named entities, and audio playback for each.

### CLI

```bash
# Whole document, default settings
python src/llm-aided-ocr-cli.py document.pdf

# Pages 2–10 only, no Markdown formatting
python src/llm-aided-ocr-cli.py document.pdf --skip-pages 1 --max-pages 9 --no-markdown

# Tune the hallucination filter
python src/llm-aided-ocr-cli.py document.pdf --threshold 0.35 --test-filtering
```

Available flags:

| Flag | Default | What it does |
|---|---|---|
| `--max-pages` | `0` (all) | How many pages to process |
| `--skip-pages` | `0` | How many pages to skip at the start |
| `--threshold` | `0.40` | Starting hallucination similarity threshold |
| `--check-english` | off | Validate that extracted text is English |
| `--no-markdown` | off | Output plain text instead of Markdown |
| `--db-path` | `./sentence_embeddings.sqlite` | Embeddings DB path |
| `--test-filtering` | off | Test hallucination filtering on existing output |

### Docker

`docker-compose.yml` is configured for NVIDIA GPU passthrough and ships with Tesseract, Poppler, Python 3.12, and the Llama dependencies baked in.

```bash
docker compose up -d
```

The container exposes SSH on port `3232` and shares 32 GB of shared memory.

---

## How the pipeline works

1. **Configuration** — reads `.env`.
2. **PDF → images** — `pdf2image` rasterises each page using Poppler.
3. **OCR** — Tesseract runs over every page in a thread pool.
4. **Chunking** — the combined text is split into ~8000-character chunks that respect paragraph and sentence boundaries, with overlap between chunks so the LLM never loses context.
5. **LLM correction** — each chunk is sent to the configured LLM with a prompt that asks it to fix OCR errors without changing the meaning. The OpenAI path runs chunks concurrently with `asyncio.gather()`; the local Llama path runs sequentially.
6. **Markdown pass (optional)** — a second prompt converts the corrected text into properly structured Markdown.
7. **Reassembly + save** — chunks are stitched back together and written to disk.
8. **Quality score** — a final LLM call compares the raw OCR and the cleaned output and returns a score out of 100 with an explanation.

---

## A worked example

The `samples/` folder has a real before/after pair from a 1975 Warren Buffett letter to Katharine Graham about pension fund management.

Raw OCR (a few lines):

```
fo control promises rationally, it is necessary to
understand the basic arithmetic and practical rules
governing pension plans.

-2-

they had $2.2 billion in the hank", whose economic
results would impact Future values for the shareholders.

This requires a very wise and informed
elient ~ and even then is not free from pitfaLLs.
```

After LLM correction:

```markdown
## The Irreversible Nature of Pension Promises

To control promises rationally, it is necessary to understand
the basic arithmetic and practical rules governing pension plans.

they had $2.2 billion in the bank, whose economic results would
impact future values for the shareholders.

This requires a very wise and informed client — and even then
is not free from pitfalls.
```

`fo → To`, `hank → bank`, `elient → client`, `pitfaLLs → pitfalls`, the stray `-2-` page number is gone, and the heading is properly formatted.

---

## Project artefacts

The `docs/` folder contains the academic work that goes with this project:

- **Research paper** — full write-up of the pipeline and results
- **Patent filing** — covers the chunking strategy and quality scoring approach
- **Capstone presentation** — defence slides
- **Conference poster** — visual summary

The `samples/` folder has the sample document, the raw Tesseract output, and the LLM-corrected version side by side.

The project website (`index.html`) ties all of it together with a walk-through of the pipeline, a live before/after comparison, and links to every artefact.

---

## License

See the repository for license details.
