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

## Private-TG-Tracker (KPI & KOL Tracking)
<img width="1437" height="673" alt="Screenshot 2026-04-23 at 14 51 02" src="https://github.com/user-attachments/assets/f889721e-462c-400a-88a2-9b0a241b3028" />
### 4. Bot Listen
**File:** `Private-TG-Tracker/Bot Listen.json`

Bot lắng nghe reply từ KOL khi gửi bài post về. Tracking KPI Post và Event Salary.

**Flow:**
1. Telegram Trigger nhận tin nhắn reply của KOL
2. Kiểm tra tin nhắn reply có phải từ bot BingX không (KPI Post / Event Salary)
3. Extract link bài đăng từ `link_preview_options.url` hoặc text
4. Tính duration giữa BD gửi bài và KOL đăng bài
5. Update KPI vào Google Sheets (sheet KPI_Post) và Lark Bitable
6. Ghi nhận vào sheet Result_KPI_Post và Result_Event_Salary

**Integrations:**
- Telegram Bot (Huyền - KPI Post)
- Google Sheets (KPI POST EVENT SALARY AUTO POST & REPORT)
- Lark/Feishu Bitable (VFtYwvJsmivhoGkdd7glkd66gYd)

---

### 5. Bot group BD_Send
**File:** `Private-TG-Tracker/Bot group BD_Send.json`

Bot gửi bài KPI cho KOL và tracking khi KOL phản hồi link bài đăng.

**Flow:**
1. Get Unprocessed Rows từ DataTable (danh sách bài cần gửi)
2. Switch phân loại: KPI Post vs Event Salary
3. Append row vào sheet KPI_Post hoặc Event_Salary
4. Get ref code từ sheet Ref_code
5. Gửi media (photo/video/document) qua Telegram cho từng group
6. Gửi tin nhắn reminder cho Admin group
7. Delete row sau khi gửi xong
8. Update status thành Done trong sheet

**Features:**
- Hỗ trợ album (media_group_id)
- Inject refcode vào link
- Xử lý entities (text_link URL update)
- Append CTA vào cuối nội dung

---

## TG-MultiHub-Auto (Auto-Post Multi-Language)
<img width="1440" height="751" alt="Screenshot 2026-04-23 at 14 49 47" src="https://github.com/user-attachments/assets/d17772a6-5e06-49f8-8e6a-bb61abf35e66" />

### 6. Đăng bài (Auto-Post Multi-Channel)
**File:** `TG-MultiHub-Auto/Đăng bài.json`

Đăng bài tự động đa channel với các ngôn ngữ khác nhau (Russian, Chinese, Indonesian, Vietnamese).

**Flow:**
1. Schedule Trigger chạy mỗi 2 phút
2. Get all posts từ Supabase Post table
3. Normalize message (photo/video/text/document)
4. Kiểm tra image moderation qua Gemini (reject nếu có tiếng Việt hoặc BC Capital)
5. Dịch nội dung sang 4 ngôn ngữ qua Information Extractor (Gemini)
6. Get channel list từ Google Sheet (n8n-sheet)
7. Loop qua từng ngôn ngữ, lấy channel tương ứng
8. Gửi bài đăng đã dịch tới các channel Telegram
9. Lưu lịch sử đăng vào Channel_Russian, Channel_Chinese, Channel_Indonesian, Channel_Vietnamese
10. Delete post sau khi đăng xong

**Languages:**
- Russian
- Simplified Chinese
- Indonesian
- Vietnamese

**Tables:**
- `Post` - Lưu bài gốc từ Telegram channel
- `Channel_main` - Lưu thông tin bài đăng chính
- `Channel_Russian`, `Channel_Chinese`, `Channel_Indonesian`, `Channel_Vietnamese` - Lưu bài đã đăng theo ngôn ngữ
- `N8N_sheet` - Mapping channel theo ngôn ngữ (từ Google Sheets)

---

### 7. Đăng bài channel public
**File:** `TG-MultiHub-Auto/Đăng bài channel public.json`

Đăng bài được phân loại sang public channel @bullpacksignal.

**Flow:**
1. Trigger từ workflow khác (executeWorkflowTrigger)
2. Kiểm tra chat_id thuộc danh sách cho phép
3. Sentiment Analysis phân loại: TIN_TUC, CALL_KEO, TP, SL
4. Dịch và viết lại nội dung bằng tiếng Anh chuyên nghiệp (Gemini)
5. Gửi tới @bullpacksignal qua Telegram
6. Lưu vào Other_Posts table

---

### 8. Phân tích (Analysis)
**File:** `TG-MultiHub-Auto/Phân tích.json`

Phân tích TP/SL và RR từ nội dung bài đăng.

**Flow:**
1. Schedule Trigger chạy mỗi 20 phút
2. Get all posts từ Channel_main (Supabase)
3. Filter những bài chưa có TP/Loss
4. Information Extractor extract tp_status và rr_value
5. Update TP/Loss vào Channel_main
6. Get all posts để analyze tiếp
7. Code in JavaScript chuẩn bị data theo ma_lenh
8. Message a model (GPT-4.1-mini) phân tích final_rr và tp_hit
9. Merge kết quả
10. Create row vào Summary table

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
- `telegramApi` - Telegram bots

Import into n8n and update credentials to match your instance.
