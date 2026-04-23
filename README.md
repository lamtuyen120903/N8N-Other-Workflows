# N8N Workflows

Collection of n8n automation workflows.

## Workflows

### 1. Loveable - Webhook
**File:** `Loveable - Webhook.json`

Webhook handler to sync workflows between n8n and Loveable. Listens for HTTP POST requests and performs actions:

- **`/cap-nhat-flow`** - Update workflow info to Loveable via Supabase function
- **`/xoa-flow`** - Delete workflow from Loveable

**Flow:**
1. Webhook receives request
2. Switch checks action (`add` or `cập nhật`)
3. Send HTTP request to Supabase endpoints for sync

---

### 2. Supabase RAG AI Agent PDFs & More
**File:** `Supabase RAG AI Agent PDFs & More.json`

RAG (Retrieval Augmented Generation) chatbot using Supabase as vector database and OpenAI/Mistral for AI processing.

**Two main parts:**

#### A. Document Ingestion (add documents to Vector DB)
1. Download file from Google Drive
2. Extract text using Mistral
3. Split text into chunks (5000 chars)
4. Generate embeddings with OpenAI
5. Insert into Supabase vectorstore

#### B. Chat Interface (RAG chat)
1. Chat trigger receives message
2. Generate embeddings from query
3. Retrieve relevant documents from Supabase
4. RAG AI Agent answers based on context

**Tools:**
- Supabase Vector Store (documents table)
- Postgres Chat Memory
- Google Drive integration

---

### 3. Translate Video to Vietnamese Speech
**File:** `translate video thành speech tiếng việt.json`

Convert video/audio to Vietnamese speech.

**Flow:**
1. Download file from Google Drive
2. Transcribe audio/video using ElevenLabs (Vietnamese)
3. Translate text to Vietnamese using Google Gemini
4. Convert text to speech using ElevenLabs (Sarah voice, Vietnamese)

**APIs:**
- ElevenLabs (transcribe + TTS)
- Google Gemini (translation)
- Google Drive (file source)

---

## Setup

Workflows include saved credentials in each JSON file:
- `n8nApi` - n8n account
- `googleDriveOAuth2Api` - Google Drive
- `elevenLabsApi` - ElevenLabs
- `googlePalmApi` - Google Gemini
- `supabaseApi` - Supabase
- `openAiApi` - OpenAI
- `postgres` - PostgreSQL

Import into n8n and update credentials to match your instance.
