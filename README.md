# MedScan AI

MedScan AI is a full-stack application that scans medical reports (PDFs, images), extracts text using OCR, and returns a plain-language medical summary and insights in **English** and **Hindi**. It also provides structured medical highlights (e.g., key findings, suggested follow-ups, important values) to help patients and clinicians quickly understand reports.

> Built with **Node.js + Express** for the backend and **React** for the frontend.

---

## Features

* Upload medical reports (PDF, JPG, PNG).
* OCR extraction (supports multi-page PDFs).
* AI-based summarization in **simple English** and **simple Hindi**.
* Medical insights: key findings, important numbers, possible next steps, and cautions.
* Progress/processing status while backend processes files.
* REST API for easy integration.
* Extensible to support other languages and deployment environments.

---

## Tech Stack

* Backend: Node.js, Express, Multer (file upload), pdf-parse, Tesseract.js (OCR), optional image processing libs
* Frontend: React (create-react-app or Vite), fetch/axios for API calls
* AI: any LLM / summarization API of your choice (OpenAI, Anthropic, Perplexity, or a locally hosted model)
* Storage: local disk (dev) or cloud object storage (S3, DigitalOcean Spaces)
* Optional: Redis for job queue/status, PostgreSQL or MongoDB for metadata

---

## Architecture Overview

1. Frontend uploads a report to the backend.
2. Backend stores the file and starts processing (OCR → text extraction → AI summarization).
3. Backend returns a result containing: plain-language summaries (EN + HI), structured insights, and original extracted text.
4. Frontend displays results with a clean UI and language toggle.

---

## Security & Privacy

* MedScan may process sensitive health data. In production, ensure:

  * HTTPS everywhere
  * Secure storage (encrypted at rest)
  * Access controls & authentication
  * Logs scrubbed of PHI
  * If required, follow regional regulations (e.g., HIPAA, GDPR)

* For demo/dev: avoid uploading real patient-identifiable data.

---

## Getting Started (Developer)

### Prerequisites

* Node.js (v16+ recommended)
* npm or yarn
* (Optional) tesseract binaries if you choose native Tesseract instead of pure JS

### Repo structure (suggested)

```
/medscan-ai
  /backend
    server.js
    package.json
    /routes
    /controllers
    /services
    /uploads
  /frontend
    package.json
    src/
  README.md
```

### Environment (.env)

Create a `.env` in `backend/` with your API keys and config:

```
PORT=5000
STORAGE_PATH=./uploads
AI_API_KEY=your_api_key_here
AI_PROVIDER=openai # or anthopic/perplexity etc.
OCR_ENGINE=tesseractjs # or tesseract
MAX_FILE_SIZE=10485760
```

**Important:** Do not commit `.env` or real keys to source control.

---

## Installation

### Backend

```bash
cd backend
npm install
# or
# yarn
```

### Frontend

```bash
cd frontend
npm install
# start dev server
npm run dev
```

---

## Running Locally (example)

Start backend:

```bash
cd backend
npm run start
```

Start frontend (React):

```bash
cd frontend
npm start
```

Open the frontend in your browser (usually `http://localhost:3000`) — it will talk to the backend on the port in `.env` (e.g. `http://localhost:5000`).

---

## API Reference

### POST `/api/summarize`

Upload a file (multipart/form-data). Field name: `file`.

**Request** (curl example):

```bash
curl -X POST "http://localhost:5000/api/summarize" \
  -H "Accept: application/json" \
  -F "file=@/path/to/report.pdf"
```

**Response** (200):

```json
{
  "status": "success",
  "data": {
    "extractedText": "...full extracted OCR text...",
    "summary": {
      "english": "Simple English summary...",
      "hindi": "सरल हिंदी सारांश..."
    },
    "insights": {
      "keyFindings": ["Finding A", "Finding B"],
      "importantValues": {"Hb": "13.2 g/dL", "Sugar": "120 mg/dL"},
      "suggestedNextSteps": ["Consult ENT", "Repeat test in 1 month"]
    }
  }
}
```

### POST `/api/translate-summary`

(Optional) Translate or rephrase an existing summary. Accepts JSON with `text` and `targetLang` (`en` or `hi`).

### GET `/api/status/:jobId`

(Optional) If you run processing asynchronously (recommended for large files), use a jobId to poll for status.

---

## Implementation Notes / Suggested Flow

1. **File upload** – use `multer` to accept files and store temporarily.
2. **Preprocessing** – if PDF, use `pdf-parse` to extract embedded text first. If text is low-quality, run OCR with `tesseract.js` on images or PDF pages converted to images.
3. **Text normalization** – fix encoding, remove headers/footers, split sections.
4. **Summarization** – call your LLM API with a clear system prompt:

   * Provide `extractedText` as input.
   * Ask for: short plain-language summary (2–3 sentences), a 1-line TL;DR, key findings, and recommended next steps.
   * Request both English and Hindi outputs (or do two calls / translate).
5. **Postprocess** – detect and highlight numeric values and lab result tables. Return structured JSON.

**Prompt hygiene tip:** limit the input chunk size; summarize in stages (section -> aggregate) for long reports.

---

## Frontend UI Suggestions

* Simple upload form with drag-and-drop and preview of uploaded pages.
* Processing modal with progress indicator.
* Result view: two tabs (English / Hindi) and a collapsible panel for raw extracted text.
* Highlighted key values and copy-to-clipboard buttons.

---

## Testing

* Create test reports (images + PDFs) with known text to validate OCR accuracy.
* Unit tests for: upload controller, OCR service, AI client wrapper.
* Integration tests for end-to-end flow (upload → summary).

---

## Deployment

* Use environment variables for secrets.
* Store uploads in cloud object storage and keep only short-lived local copies.
* Consider serverless functions for the AI calls and a job-queue worker for OCR if you expect high concurrency.

---

## Privacy & Legal

* Clearly document in your project README or in-app about data retention and deletion policies.
* Add a consent step if you plan to process identifiable patient data.

---

## Contributing

Contributions welcome! Suggested workflow:

1. Fork the repo
2. Create a feature branch
3. Open a pull request with description & screenshots

Please follow the coding style in the project and add tests where applicable.

---

## Roadmap / Future Ideas

* Add user accounts and secure report history
* Add more languages beyond English & Hindi
* Add a clinician review mode and annotations
* Integrate with EHR systems via HL7/FHIR
* On-device OCR for improved privacy

---

## License

Specify your license (e.g., MIT) or replace with your preferred license.

---

## Acknowledgements

* Tesseract.js / pdf-parse / multer
* Any LLM provider you use for summarization

---

*If you want, I can also:

* generate example `.env` and `Procfile` for deployment,
* add Postman collection or OpenAPI spec for the API,
* create a short pitch README for a hackathon presentation.*
