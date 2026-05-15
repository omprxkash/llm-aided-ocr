# LLM-Aided OCR Project

## Introduction

The LLM-Aided OCR Project is an advanced system designed to significantly enhance the quality of Optical Character Recognition (OCR) output. By leveraging cutting-edge natural language processing techniques and large language models (LLMs), this project transforms raw OCR text into highly accurate, well-formatted, and readable documents.

## Advanced PDF-to-Text/Markdown Converter with LLM Refinement

This Python script provides a powerful pipeline for converting PDF documents into cleaned-up text or well-formatted Markdown. It leverages OCR technology to extract initial text and then utilizes Large Language Models (LLMs) – supporting OpenAI, Anthropic Claude, and local GGUF models – to correct OCR errors and reformat the content. The script is designed to handle large documents efficiently through intelligent chunking and asynchronous processing.

## Features

* **PDF to Image Conversion:** Converts PDF pages into images for OCR processing.
* **OCR Text Extraction:** Uses Tesseract OCR to extract raw text from images.
* **LLM-Powered OCR Correction:** Employs LLMs to identify and fix errors commonly introduced during OCR.
* **Markdown Reformatting:** Optionally reformats the corrected text into structured Markdown, preserving headings, lists, and other semantic elements.
* **Multi-Provider LLM Support:**
    * **OpenAI:** GPT models (e.g., `gpt-4o-mini`).
    * **Anthropic:** Claude models (e.g., `claude-3-haiku-20240307`).
    * **Local LLMs:** Supports GGUF format models (e.g., Llama 3.1) via `llama-cpp-python`, with automatic model download for a default model.
* **Intelligent Text Chunking:** Breaks down large documents into manageable pieces for LLM processing, respecting paragraph and sentence boundaries, and handling context across chunks.
* **GPU Acceleration:** Supports GPU acceleration for local LLMs if compatible hardware and `llama-cpp-python` with CUDA support are available.
* **Asynchronous Processing:** Utilizes `asyncio` for concurrent API calls when using OpenAI or Claude, speeding up processing for large documents.
* **Customizable Processing:** Options to suppress headers/footers, control page ranges, and choose between text or Markdown output.
* **Automated Quality Assessment:** Includes a feature to assess the quality of the LLM-processed output against the raw OCR text using an LLM.
* **Configuration Driven:** Easily configured via a `.env` file.

## Workflow

1.  **Configuration:** Loads settings from the `.env` file.
2.  **PDF Conversion:** The input PDF is converted page by page into images.
3.  **OCR Extraction:** Tesseract OCR processes these images to extract raw text. This raw text is saved.
4.  **Text Aggregation & Chunking:** The raw text from all pages is combined and then intelligently split into overlapping chunks suitable for LLM context windows.
5.  **LLM Refinement (Per Chunk):**
    * **OCR Correction:** Each chunk is sent to the configured LLM with a prompt to correct OCR errors, considering the context from the previous chunk.
    * **Markdown Formatting (Optional):** If enabled, the corrected chunk is further processed by the LLM to reformat it into Markdown.
6.  **Reassembly:** The processed chunks are stitched back together to form the complete, refined document.
7.  **Output Generation:** The final text or Markdown is saved to a new file.
8.  **Quality Assessment (Optional):** Samples of the original OCR text and the final processed text are sent to an LLM for a quality score and explanatory feedback.

## Technologies Used

* **Core:** Python 3.8+
* **PDF Processing:** `pdf2image` (requires Poppler)
* **OCR:** `pytesseract` (requires Tesseract OCR engine)
* **Image Manipulation:** `Pillow`, `opencv-python`
* **LLM APIs:**
    * `openai` (for OpenAI models)
    * `anthropic` (for Claude models)
* **Local LLMs:** `llama-cpp-python` (inferred - ensure you install it if `USE_LOCAL_LLM=True`)
* **Configuration:** `python-decouple`
* **Tokenization:** `tiktoken` (for OpenAI), `transformers` (for Claude, Llama tokenizers)
* **Concurrency:** `asyncio`
* **Utilities:** `numpy`, `filelock`, `logging`

## Prerequisites

1.  **Python:** Version 3.8 or higher.
2.  **Tesseract OCR Engine:**
    * Installation instructions: [Tesseract GitHub](https://github.com/tesseract-ocr/tesseract)
    * Ensure it's added to your system's PATH.
    * Install necessary language data (e.g., for English: `eng.traineddata`).
3.  **Poppler:**
    * `pdf2image` depends on Poppler utilities.
    * **Linux:** `sudo apt-get install poppler-utils`
    * **macOS:** `brew install poppler`
    * **Windows:** Download Poppler binaries, extract them, and add the `bin/` directory to your PATH.
4.  **For Local LLM GPU Support (Optional):**
    * A compatible NVIDIA GPU.
    * CUDA Toolkit.
    * Install `llama-cpp-python` with CUDA support (e.g., `CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install llama-cpp-python`).

## Installation

1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd <repository-directory>
    ```

2.  **Create a virtual environment (recommended):**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows: venv\Scripts\activate
    ```

3.  **Install Python dependencies:**
    Create a `requirements.txt` file (see [Requirements](#requirements) section below) and run:
    ```bash
    pip install -r requirements.txt
    ```
    *If you plan to use local LLMs, install `llama-cpp-python` separately, potentially with GPU support as described in Prerequisites.*
    ```bash
    # Example for llama-cpp-python (CPU by default)
    # pip install llama-cpp-python
    # Example for llama-cpp-python with NVIDIA GPU support
    # CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install llama-cpp-python
    ```

4.  **Set up configuration:**
    * Rename `.env.example` to `.env` (or create a new `.env` file).
    * Fill in the required API keys and model preferences. See [Configuration](#configuration) below.

## Configuration

Create a `.env` file in the root directory of the project with the following variables:






# General Settings
USE_LOCAL_LLM=False # Set to True to use a local GGUF model, False for API
API_PROVIDER="OPENAI" # or "CLAUDE" if USE_LOCAL_LLM=False

# OpenAI Configuration (if API_PROVIDER="OPENAI")
OPENAI_API_KEY="your-openai-api-key"
OPENAI_COMPLETION_MODEL="gpt-4o-mini" # Or any other suitable completion model
OPENAI_EMBEDDING_MODEL="text-embedding-3-small" # Used if embeddings are needed elsewhere, not directly by this script's core flow

# Anthropic Configuration (if API_PROVIDER="CLAUDE")
ANTHROPIC_API_KEY="your-anthropic-api-key"
CLAUDE_MODEL_STRING="claude-3-haiku-20240307" # Or other Claude model

# Local LLM Configuration (if USE_LOCAL_LLM=True)
DEFAULT_LOCAL_MODEL_NAME="Llama-3.1-8B-Lexi-Uncensored_Q5_fixedrope.gguf" # The script will try to download this if not found in ./models
LOCAL_LLM_CONTEXT_SIZE_IN_TOKENS=2048 # Adjust based on your model and VRAM

# Optional: Verbose logging for llama.cpp
USE_VERBOSE=False


## Features

- PDF to image conversion
- OCR using Tesseract
- Advanced error correction using LLMs (local or API-based)
- Smart text chunking for efficient processing
- Markdown formatting option
- Header and page number suppression (optional)
- Quality assessment of the final output
- Support for both local LLMs and cloud-based API providers (OpenAI, Anthropic)
- Asynchronous processing for improved performance
- Detailed logging for process tracking and debugging
- GPU acceleration for local LLM inference

## Detailed Technical Overview

### PDF Processing and OCR

1. **PDF to Image Conversion**
   - Function: `convert_pdf_to_images()`
   - Uses `pdf2image` library to convert PDF pages into images
   - Supports processing a subset of pages with `max_pages` and `skip_first_n_pages` parameters

2. **OCR Processing**
   - Function: `ocr_image()`
   - Utilizes `pytesseract` for text extraction
   - Includes image preprocessing with `preprocess_image()` function:
     - Converts image to grayscale
     - Applies binary thresholding using Otsu's method
     - Performs dilation to enhance text clarity

### Text Processing Pipeline

1. **Chunk Creation**
   - The `process_document()` function splits the full text into manageable chunks
   - Uses sentence boundaries for natural splits
   - Implements an overlap between chunks to maintain context

2. **Error Correction and Formatting**
   - Core function: `process_chunk()`
   - Two-step process:
     a. OCR Correction:
        - Uses LLM to fix OCR-induced errors
        - Maintains original structure and content
     b. Markdown Formatting (optional):
        - Converts text to proper markdown format
        - Handles headings, lists, emphasis, and more

3. **Duplicate Content Removal**
   - Implemented within the markdown formatting step
   - Identifies and removes exact or near-exact repeated paragraphs
   - Preserves unique content and ensures text flow

4. **Header and Page Number Suppression (Optional)**
   - Can be configured to remove or distinctly format headers, footers, and page numbers

### LLM Integration

1. **Flexible LLM Support**
   - Supports both local LLMs and cloud-based API providers (OpenAI, Anthropic)
   - Configurable through environment variables

2. **Local LLM Handling**
   - Function: `generate_completion_from_local_llm()`
   - Uses `llama_cpp` library for local LLM inference
   - Supports custom grammars for structured output

3. **API-based LLM Handling**
   - Functions: `generate_completion_from_claude()` and `generate_completion_from_openai()`
   - Implements proper error handling and retry logic
   - Manages token limits and adjusts request sizes dynamically

4. **Asynchronous Processing**
   - Uses `asyncio` for concurrent processing of chunks when using API-based LLMs
   - Maintains order of processed chunks for coherent final output

### Token Management

1. **Token Estimation**
   - Function: `estimate_tokens()`
   - Uses model-specific tokenizers when available
   - Falls back to `approximate_tokens()` for quick estimation

2. **Dynamic Token Adjustment**
   - Adjusts `max_tokens` parameter based on prompt length and model limits
   - Implements `TOKEN_BUFFER` and `TOKEN_CUSHION` for safe token management

### Quality Assessment

1. **Output Quality Evaluation**
   - Function: `assess_output_quality()`
   - Compares original OCR text with processed output
   - Uses LLM to provide a quality score and explanation

### Logging and Error Handling

- Comprehensive logging throughout the codebase
- Detailed error messages and stack traces for debugging
- Suppresses HTTP request logs to reduce noise

## Configuration and Customization

The project uses a `.env` file for easy configuration. Key settings include:

- LLM selection (local or API-based)
- API provider selection
- Model selection for different providers
- Token limits and buffer sizes
- Markdown formatting options

## Output and File Handling

1. **Raw OCR Output**: Saved as `{base_name}__raw_ocr_output.txt`
2. **LLM Corrected Output**: Saved as `{base_name}_llm_corrected.md` or `.txt`

The script generates detailed logs of the entire process, including timing information and quality assessments.

## Requirements

- Python 3.12+
- Tesseract OCR engine
- PDF2Image library
- PyTesseract
- OpenAI API (optional)
- Anthropic API (optional)
- Local LLM support (optional, requires compatible GGUF model)


## Usage

1. Place your PDF file in the project directory.

2. Update the `input_pdf_file_path` variable in the `main()` function with your PDF filename.

3. Run the script:
   
  `llm_aided_ocr.py`
   

4. The script will generate several output files, including the final post-processed text.

## How It Works

The LLM-Aided OCR project employs a multi-step process to transform raw OCR output into high-quality, readable text:

1. **PDF Conversion**: Converts input PDF into images using `pdf2image`.

2. **OCR**: Applies Tesseract OCR to extract text from images.

3. **Text Chunking**: Splits the raw OCR output into manageable chunks for processing.

4. **Error Correction**: Each chunk undergoes LLM-based processing to correct OCR errors and improve readability.

5. **Markdown Formatting** (Optional): Reformats the corrected text into clean, consistent Markdown.

6. **Quality Assessment**: An LLM-based evaluation compares the final output quality to the original OCR text.

## Code Optimization

- **Concurrent Processing**: When using API-based models, chunks are processed concurrently to improve speed.
- **Context Preservation**: Each chunk includes a small overlap with the previous chunk to maintain context.
- **Adaptive Token Management**: The system dynamically adjusts the number of tokens used for LLM requests based on input size and model constraints.

## Configuration

The project uses a `.env` file for configuration. Key settings include:

- `USE_LOCAL_LLM`: Set to `True` to use a local LLM, `False` for API-based LLMs.
- `API_PROVIDER`: Choose between "OPENAI" or "CLAUDE".
- `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`: API keys for respective services.
- `CLAUDE_MODEL_STRING`, `OPENAI_COMPLETION_MODEL`: Specify the model to use for each provider.
- `LOCAL_LLM_CONTEXT_SIZE_IN_TOKENS`: Set the context size for local LLMs.

## Output Files

The script generates several output files:

1. `{base_name}__raw_ocr_output.txt`: Raw OCR output from Tesseract.
2. `{base_name}_llm_corrected.md`: Final LLM-corrected and formatted text.

## Limitations and Future Improvements

- The system's performance is heavily dependent on the quality of the LLM used.
- Processing very large documents can be time-consuming and may require significant computational resources.

## Contributing

Contributions to this project are welcome! Please fork the repository and submit a pull request with your proposed changes.

## License

This project is licensed under the MIT License.

---




