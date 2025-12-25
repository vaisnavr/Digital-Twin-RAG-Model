# Digital-Twin-RAG-Model

**Project:** Customer Persona Twin — *Trend-Conscious Sustainable Shopper* (Persona Mimic)  
**Notebook:** `Final LLM Project.ipynb`

---

## 📌 1. Project Overview

This project builds a **Customer Persona Twin** that mimics how a **fashion-forward, sustainability-aware shopper** thinks and speaks when asked fall styling questions.

Unlike a generic recommender or a “stylist textbook,” the twin responds using the persona’s:

- Tone  
- Tradeoffs  
- Priorities (**trend relevance + ethics + practicality**)

### 🔁 System Flow (RAG Pipeline)

The system is implemented as a **Retrieval-Augmented Generation (RAG)** pipeline on top of **Gemini**:

1. A user asks a question  
   *(e.g., “How do I upgrade my fall wardrobe sustainably?”)*
2. The system retrieves relevant evidence snippets from a custom knowledge base
3. Gemini generates an answer in the persona’s voice grounded in retrieved context

✅ This satisfies the project requirement to adapt a pretrained LLM using **domain data, prompt engineering, RAG, and evaluation**.

---

## 🧍 2. Digital Twin Definition

### 2.1 Digital Twin Type

**Customer Persona Twin — Persona Mimic**

### 2.2 Persona

**Trend-Conscious Sustainable Shopper**

- Fashion-forward but practical  
- Influenced by trends and social media, but avoids fast fashion excess  
- Strong preference for ethical and sustainable choices:
  - materials  
  - thrifting  
  - cost-per-wear  
- Likes confident, curated recommendations and “wearable” trend translation  

### 2.3 Twin Behavior

- Responds like the shopper would talk  
- Makes realistic tradeoffs (trend + sustainability + versatility)  
- Uses **“Evidence used” citations** from retrieved chunks to show grounding  

---

## 📚 3. Data Sources and Preprocessing

The notebook uses **two distinct data sources**, meeting the course requirement.

### 3.1 Data Source 1 — Reddit Threads (Fashion Discussions)

**Goal:**  
Capture authentic language and real consumer opinions about trends, styling, practicality, and brand perceptions.

**What the code does:**

- Scrapes Reddit threads using Reddit’s JSON endpoints (`requests`)
- Collects:
  - post title  
  - post body  
  - comment text (optionally expanding “more comments”)
- Filters to the relevant subreddit: `r/femalefashionadvice`
- Writes scraped output to a `.jsonl` file (one thread per line)

**Preprocessing:**

- Builds a single document per thread (title + post + first *N* comments)
- Strips newlines and formats comments into readable bullet points

---

### 3.2 Data Source 2 — YouTube (Audio → Transcript)

**Goal:**  
Add higher-level trend guidance and seasonal fashion commentary from video essays and fashion channels.

**What the code does:**

- Downloads audio from YouTube URLs using `yt-dlp` (audio-only MP3s)
- Uploads MP3 files to Gemini for transcription
- Writes transcripts to `youtube_transcripts.jsonl`
- Merges Reddit and YouTube documents into a combined corpus

**Preprocessing:**

- Converts transcripts to plain text
- Assigns document IDs like `youtube_<video_id>`
- Stores metadata:
  - `source_type`
  - `url`
  - `text`

---

## 🔍 4. Retrieval-Augmented Generation (RAG) Pipeline

### 4.1 Chunking Strategy

**Reddit Chunking**

- Word-based chunking (~250 words per chunk)
- Metadata includes:
  - `source` (reddit)
  - `post_id`, `title`, `url`
  - `chunk_id` (e.g., `reddit_<postid>_c000`)

**Combined Corpus Chunking (Reddit + YouTube)**

- Same chunk size (~250 words)
- Metadata includes:
  - `source` (reddit or youtube)
  - unique `chunk_id`

**Why Chunking Matters**

- Improves retrieval precision  
- Keeps prompt context within model limits  

---

### 4.2 Embeddings

Each chunk is embedded using:

- **Gemini embedding model:** `gemini-embedding-001`

The notebook defines an embedding function that converts text into a float32 NumPy vector.

---

### 4.3 Vector Index (FAISS)

The system uses **FAISS `IndexFlatL2`** for fast similarity search.

**Workflow:**

- Build an embedding matrix from all chunk embeddings  
- Add embeddings to the FAISS index  
- At query time, embed the user query and retrieve the top-k nearest chunks  

---

### 4.4 Retrieval Function

- Returns the **top-k most similar chunks** by vector distance  
- Retrieved chunks become the grounding **CONTEXT** for generation  

---

### 4.5 Generation Step

The system constructs the final prompt using:

- Persona prompt  
- Grounding rules  
- Response format  
- Injected retrieved context:

The prompt is passed to Gemini (e.g., `models/gemini-2.5-flash`) to generate the final response.

---

### 4.6 Reliability / Error Handling

- A `safe_generate()` wrapper retries transient Gemini server errors  
- Improves stability for classroom demos  

---

## 🧠 5. Prompt Engineering (Iteration Evidence)

### 5.1 Prompt v1 — Initial Persona + Structured Output

**Features:**

- Persona definition  
- Grounding rule: *“Use ONLY the CONTEXT”*
- Structured output:
- Style direction  
- Outfit formulas  
- Sustainable swaps  
- What I’d avoid  
- Cited chunk IDs  

**Techniques Used:**

- Role prompting  
- Grounding constraints  
- Output schema enforcement  

---

### 5.2 Prompt v2 — Stronger Persona Mimic + Messaging Test

**Enhancements:**

- Stronger voice constraints (concise, confident, trend-aware)
- Explicit dislikes (fast fashion, synthetic-heavy hype)
- Adds a **“Messaging test”** section (2 marketing lines)
- Requires an explicit **“Evidence used”** list

**Why This Improves the Twin**

- Adds business value through messaging evaluation  
- Improves consistency across outputs  

---

## 🧪 6. Demo Queries and Baselines

The notebook compares:

- **Baseline:** Gemini without retrieval  
- **RAG:** Gemini with retrieved context  

**Example Queries:**

- Sustainable fall styling formulas  
- Materials to prioritize for fall basics  
- *“Best sustainable shoe brands for fall 2025”* (hallucination stress test)

---

## 📊 7. Evaluation Design

### 7.1 Evaluation Criteria

- Persona voice match  
- Sustainability alignment  
- Trend and seasonal relevance  
- Grounding / non-hallucination  
- Structure compliance  
- Evidence transparency  

### 7.2 Evaluation Method

- Runs baseline vs RAG across multiple queries  
- Prints retrieved chunk IDs for sanity checks  
- Increases `k` to ensure Reddit + YouTube coverage  
- Appends evidence automatically if omitted  

### 7.3 Expected Findings

- Baseline: fluent but generic  
- RAG improves:
- specificity  
- persona consistency  
- transparency  

Conservative refusals are treated as **non-hallucination wins**.

---

## 🏗️ 8. System Architecture Summary

User Question
↓
Chunked Corpus (Reddit + YouTube)
↓
Gemini Embeddings
↓
FAISS Similarity Search
↓
Persona Prompt + Retrieved Context
↓
Gemini Response + Evidence


---

## ▶️ 9. How to Run

**Install dependencies:**

- `praw`
- `requests`
- `faiss-cpu`
- `yt-dlp`
- `google-generativeai`

**Setup:**

- Set `GEMINI_API_KEY` securely

**Run Phase 1:**

- Scrape Reddit  
- Chunk, embed, build FAISS index  
- Run RAG demo  

**Run Phase 2:**

- Download YouTube audio  
- Transcribe  
- Merge documents  
- Rebuild index  

---

## 🚧 10. Limitations and Future Improvements

### Limitations

- Small dataset size  
- Word-based chunking may split concepts  
- Strict grounding can feel conservative  

### Future Improvements

- Add more sources (ethical fashion blogs, sustainability reports)  
- Add reranking (cross-encoder)  
- Add scoring harness (LLM-as-judge)  
- Add persona controls (budget sensitivity, dress code)
