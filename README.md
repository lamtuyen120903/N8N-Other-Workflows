# N8N Workflows

Collection of n8n automation workflows for Telegram bot management, multi-language content posting, RAG AI agents, and KPI tracking.

## Table of Contents

- [Workflows](#workflows)
  - [Loveable - Webhook](#1-lovable---webhook)
  - [Supabase RAG AI Agent PDFs \& More](#2-supabase-rag-ai-agent-pdfs--more)
  - [Translate Video to Vietnamese Speech](#3-translate-video-to-vietnamese-speech)
  - [Bot Listen](#4-bot-listen)
  - [Bot group BD_Send](#5-bot-group-bd_send)
  - [Đăng bài (Auto-Post Multi-Channel)](#6-đăng-bài-auto-post-multi-channel)
  - [Đăng bài channel public](#7-đăng-bài-channel-public)
  - [Phân tích (Analysis)](#8-phân-tích-analysis)
- [Setup](#setup)

---

## Workflows

### 1. Loveable - Webhook

**File:** `Loveable - Webhook.json`

Webhook handler to sync workflows between n8n and Loveable. Listens for HTTP POST requests and performs actions:

- **`/cap-nhat-flow`** — Update workflow info to Loveable via Supabase function
- **`/xoa-flow`** — Delete workflow from Loveable

**Flow:**
1. Webhook receives HTTP POST request
2. Switch node checks action type (`add` or `cập nhật`)
3. HTTP Request node sends data to Supabase endpoints for sync

---

### 2. Supabase RAG AI Agent PDFs & More

**File:** `Supabase RAG AI Agent PDFs & More.json`

RAG (Retrieval Augmented Generation) chatbot using Supabase as vector database and OpenAI/Mistral for AI processing.

#### A. Document Ingestion (add documents to Vector DB)

1. Download file from Google Drive
2. Extract text using Mistral OCR
3. Split text into chunks (5000 chars)
4. Generate embeddings with OpenAI `text-embedding-3-small`
5. Insert into Supabase vectorstore (`documents` table)

#### B. Chat Interface (RAG chat)

1. Chat trigger receives user message
2. Generate query embeddings with OpenAI
3. Retrieve relevant documents from Supabase via vector similarity
4. RAG AI Agent answers based on context + query

**Tools:**
- Supabase Vector Store (`documents` table)
- Postgres Chat Memory
- Google Drive integration

---

### 3. Translate Video to Vietnamese Speech

**File:** `translate video thành speech tiếng việt.json`

Convert video/audio content into Vietnamese speech using AI.

**Flow:**
1. Download file from Google Drive
2. Transcribe audio/video using ElevenLabs (Vietnamese voice)
3. Translate text to Vietnamese using Google Gemini 2.0 Flash
4. Convert translated text to speech using ElevenLabs (Sarah voice, Vietnamese)

**APIs:**
- ElevenLabs (transcription + TTS)
- Google Gemini (translation)
- Google Drive (file source)

---

## Private-TG-Tracker (KPI & KOL Tracking)
<img width="1437" height="673" alt="Screenshot 2026-04-23 at 14 51 02" src="https://github.com/user-attachments/assets/f889721e-462c-400a-88a2-9b0a241b3028" />
### 4. Bot Listen

**File:** `Private-TG-Tracker/Bot Listen.json`

Bot listens for KOL (Key Opinion Leader) replies when posts are sent. Tracks KPI Post and Event Salary.

**Flow:**
1. Telegram Trigger receives KOL reply message
2. Verify reply is from BingX bot (KPI Post / Event Salary check)
3. Extract post link from `link_preview_options.url` or message text
4. Calculate duration between BD posting and KOL publishing
5. Update KPI to Google Sheets (`KPI_Post` sheet) and Lark Bitable
6. Log to `Result_KPI_Post` and `Result_Event_Salary` sheets

**Integrations:**
- Telegram Bot (Huyền - KPI Post)
- Google Sheets (`KPI POST EVENT SALARY AUTO POST & REPORT`)
- Lark/Feishu Bitable (`VFtYwvJsmivhoGkdd7glkd66gYd`)

---

### 5. Bot group BD_Send

**File:** `Private-TG-Tracker/Bot group BD_Send.json`

Bot sends KPI posts to KOL and tracks when KOL responds with post links.

**Flow:**
1. Get Unprocessed Rows from DataTable (pending post list)
2. Switch node classifies: KPI Post vs Event Salary
3. Append row to `KPI_Post` or `Event_Salary` sheet
4. Get ref code from `Ref_code` sheet
5. Send media (photo/video/document) via Telegram to each group
6. Send reminder message to Admin group
7. Delete row after sending
8. Update status to Done in sheet

**Features:**
- Album support (`media_group_id`)
- Inject refcode into links
- Handle Telegram entities (text_link URL update)
- Append CTA to end of message content

---

## TG-MultiHub-Auto (Auto-Post Multi-Language)
<img width="1440" height="751" alt="Screenshot 2026-04-23 at 14 49 47" src="https://github.com/user-attachments/assets/d17772a6-5e06-49f8-8e6a-bb61abf35e66" />

### 6. Đăng bài (Auto-Post Multi-Channel)

**File:** `TG-MultiHub-Auto/Đăng bài.json`

Automatic multi-channel posting with 4 languages: Russian, Chinese, Indonesian, Vietnamese.

**Flow:**
1. Schedule Trigger runs every 2 minutes
2. Get all posts from Supabase `Post` table
3. Normalize message content (photo/video/text/document)
4. Image moderation via Gemini — reject if content contains Vietnamese or BC Capital
5. Translate content to 4 languages via Information Extractor (Gemini)
6. Get channel list from Google Sheet (`n8n-sheet`)
7. Loop through each language, fetch corresponding channels
8. Send translated posts to Telegram channels
9. Save posting history to `Channel_Russian`, `Channel_Chinese`, `Channel_Indonesian`, `Channel_Vietnamese` tables
10. Delete post after publishing

**Languages:**
- Russian
- Simplified Chinese
- Indonesian
- Vietnamese

**Tables:**
- `Post` — Original posts from Telegram channel
- `Channel_main` — Main post metadata
- `Channel_Russian`, `Channel_Chinese`, `Channel_Indonesian`, `Channel_Vietnamese` — Per-language published posts
- `N8N_sheet` — Channel-to-language mapping (from Google Sheets)

---

### 7. Đăng bài channel public

**File:** `TG-MultiHub-Auto/Đăng bài channel public.json`

Publishes classified posts to public channel @bullpacksignal.

**Flow:**
1. Trigger from parent workflow (`executeWorkflowTrigger`)
2. Verify `chat_id` is in allowed list
3. Sentiment Analysis classifies content: `TIN_TUC`, `CALL_KEO`, `TP`, `SL`
4. Translate and rewrite content in professional English (Gemini)
5. Send to @bullpacksignal via Telegram
6. Save to `Other_Posts` table

---

### 8. Phân tích (Analysis)

**File:** `TG-MultiHub-Auto/Phân tích.json`

Analyzes TP/SL (Take Profit/Stop Loss) and Risk-Reward (RR) from post content.

**Flow:**
1. Schedule Trigger runs every 20 minutes
2. Get all posts from `Channel_main` (Supabase)
3. Filter posts without TP/Loss data
4. Information Extractor extracts `tp_status` and `rr_value`
5. Update TP/Loss in `Channel_main`
6. Fetch all posts for analysis
7. JavaScript Code node prepares data grouped by `ma_lenh`
8. Message a model (GPT-4.1-mini) analyzes `final_rr` and `tp_hit`
9. Merge results
10. Create row in `Summary` table

---

## Setup

Each workflow JSON file includes saved credentials for the following services:

- `n8nApi` — n8n account credentials
- `googleDriveOAuth2Api` — Google Drive
- `elevenLabsApi` — ElevenLabs
- `googlePalmApi` — Google Gemini
- `supabaseApi` — Supabase
- `openAiApi` — OpenAI
- `postgres` — PostgreSQL
- `telegramApi` — Telegram bots

**To use:** Import the JSON files into your n8n instance and update the credential references to match your environment.
