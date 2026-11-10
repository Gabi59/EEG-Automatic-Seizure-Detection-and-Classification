# Agent Rules: PDF Processing & Text Extraction

## Trigger & Objective
When invoked or requested to process documents, the agent must read all PDF files from the `PDFs/` folder, extract their textual content, and save the extracted information into the `text/` folder as `.txt` files.

---

## Execution Rules & Workflow

### 1. Source Discovery (`PDFs/`)
- Scan the `PDFs/` directory for all files matching `*.pdf` or `*.PDF`.
- Inspect and process each PDF file discovered.
- If the `PDFs/` folder is empty or contains no readable PDF documents, report this status to the user and halt.

### 2. Destination Setup (`text/`)
- Ensure the destination directory `text/` exists at the project root.
- If `text/` does not exist, create it before attempting to write output files.

### 3. Text Extraction
- Extract all readable text from each PDF document across all pages.
- Preserve document reading order, headings, paragraphs, and list formatting as closely as possible.
- If a document contains pages that are scanned images without selectable text, log a warning indicating that OCR may be required for those pages.

### 4. Output Specification
- **File Naming**: For each `PDFs/<filename>.pdf`, write the output to `text/<filename>.txt` using the identical base name.
- **Encoding**: Always save `.txt` files using standard UTF-8 encoding.
- **Header Metadata (Optional)**: Include a brief top comment in the `.txt` file if helpful (e.g., source file name, extraction timestamp).
