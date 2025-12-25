# Digital-Twin-RAG-Model

**Project:** Customer Persona Twin — *Trend-Conscious Sustainable Shopper* (Persona Mimic)  
**Notebook:** `Final LLM Project.ipynb`

---

## 1. Project Overview

This project builds a **Customer Persona Twin** that mimics how a fashion-forward, sustainability-aware shopper thinks and speaks when asked fall styling questions. The twin is not a “stylist textbook” or a generic recommender; instead, it responds using the persona’s tone, tradeoffs, and priorities (**trend relevance + ethics + practicality**).

The system is implemented as a **Retrieval-Augmented Generation (RAG)** pipeline on top of **Gemini**:

1. A user asks a question (e.g., *“How do I upgrade my fall wardrobe sustainably?”*)
2. The system retrieves relevant evidence snippets from a custom knowledge base
3. Gemini generates an answer in the persona’s voice grounded in retrieved context

This satisfies the project requirement to adapt a pretrained LLM using **domain data, prompt engineering, RAG, and evaluation**.

---

## 2. Digital Twin Definition

### 2.1 Digital Twin Type
**Customer Persona Twin — Persona Mimic**

### 2.2 Persona
**Trend-Conscious Sustainable Shopper**

- Fashion-forward but practical  
- Influenced by trends and social media, but avoids fast fashion excess  
- Strong preference for ethical and sustainable choices (materials, thrifting, cost-per-wear)  
- Likes confident, curated recommendations and “wearable” trend translation  

### 2.3 Twin Behavior

- Responds like the shopper would talk  
- Makes realistic tradeoffs (trend + sustainability + versatility)  
- Uses **“Evidence used” citations** from retrieved chunks to show grounding  

---

## 3. Data Sources and Preprocessing (2 Sources)

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
- Filters to relevant subreddit: `r/femalefashionadvice`
- Writes scraped output to a `.jsonl` file (one thread per line)

**Preprocessing:**
- Builds a single document per thread (title + post + first *N* comments)
- Strips newlines and formats comments into readable bullets:

  
---

### 3.2 Data Source 2 — YouTube (Audio → Transcript)

**Goal:**  
Add higher-level trend guidance and seasonal fashion commentary from video essays and fashion channels.

**What the code does:**
- Downloads audio from YouTube URLs using `yt-dlp` (audio-only MP3s)
- Uploads MP3 files to Gemini for transcription
- Writes transcripts to `youtube_transcripts.jsonl`
- Merges Reddit and YouTube documents into:

**Preprocessing:**
- Converts transcripts to plain text
- Assigns document IDs like `youtube_<video_id>`
- Stores metadata fields:
- `source_type`
- `url`
- `text`

---

## 4. Retrieval-Augmented Generation (RAG) Pipeline

### 4.1 Chunking Strategy

**Reddit chunking**
- Word-based chunking (~250 words per chunk)
- Metadata includes:
- `source` (“reddit”)
- `post_id`, `title`, `url`
- `chunk_id` like `reddit_<postid>_c000`

**Combined corpus chunking (Reddit + YouTube)**
- Same chunk size (~250 words)
- Metadata includes source (`reddit` or `youtube`) and unique `chunk_id`

**Why chunking matters**
- Improves retrieval precision  
- Keeps prompt context within model limits  

---

### 4.2 Embeddings

Each chunk is embedded using:
- **Gemini embedding model:** `gemini-embedding-001`

The notebook defines:
```python
embed(text) -> float32 numpy vector

4.3 Vector Index (FAISS)

The system uses FAISS IndexFlatL2 for fast similarity search over embedding vectors.

Workflow:

Build an embedding matrix from all chunk embeddings

Add embeddings to the FAISS index using index.add(vectors)

At query time, embed the user query and run:

index.search(query_vector, k) to retrieve the top-k nearest chunks

4.4 Retrieval Function

The retrieval function is defined as:

retrieve(query, k)

Behavior:

Returns the top-k most similar chunks based on vector distance

Retrieved chunks become the grounding CONTEXT for generation

4.5 Generation Step

The system constructs the final generation prompt using:

Persona prompt

Grounding rules

Response format

Injected retrieved context in the form:

[chunk_id] chunk_text


The prompt is then passed to Gemini (e.g., models/gemini-2.5-flash) to generate the final response in the persona’s voice.

4.6 Reliability / Error Handling

To improve robustness:

A safe_generate() wrapper retries transient Gemini server errors

This improves reliability and stability during classroom demos

5. Prompt Engineering (Iteration Evidence)
5.1 Prompt v1 — Initial Persona + Structured Output

Features:

Persona definition (fashion-forward + sustainability-aware)

Grounding rule: “Use ONLY the CONTEXT”

Structured response format including:

Style direction

Outfit formulas

Sustainable swaps

What I’d avoid

Cited chunk IDs

Techniques Used:

Role prompting

Grounding constraints

Output schema enforcement

5.2 Prompt v2 — Stronger Persona Mimic + Messaging Test

Enhancements:

Stronger voice constraints (concise, confident, trend-aware)

Explicit dislikes (fast fashion, synthetic-heavy hype)

Adds a “Messaging test” section generating two marketing lines

Requires an explicit “Evidence used” list

Why this improves the twin:

Adds business value by enabling brand messaging tests

Improves consistency and comparability across outputs

6. Demo Queries and Baselines (RAG vs No-RAG)

The notebook compares:

Baseline: Gemini responses without retrieval

RAG: Gemini responses grounded in retrieved context

Example demo queries:

Sustainable fall styling formulas

Materials to prioritize for sustainable fall basics

“Best sustainable shoe brands for fall 2025” (hallucination stress test)

7. Evaluation Design
7.1 Evaluation Criteria

Persona voice match

Sustainability alignment

Trend and fall relevance

Grounding / non-hallucination

Structure compliance

Evidence transparency

7.2 Evaluation Method

Runs baseline vs RAG across multiple queries

Prints retrieved chunk IDs for retrieval sanity checks

Increases k to ensure both Reddit and YouTube chunks are retrieved

Automatically appends evidence if the model omits it

7.3 Expected Findings

Baseline responses tend to be fluent but generic

RAG improves:

Specificity

Persona consistency

Transparency through evidence citation

Conservative refusals are treated as non-hallucination wins

8. System Architecture Summary

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

9. How to Run

Install dependencies:

praw

requests

faiss-cpu

yt-dlp

google-generativeai

Set GEMINI_API_KEY securely (environment variable or secret manager)

Run Phase 1:

Scrape Reddit data

Chunk, embed, and build FAISS index

Run RAG demo

Run Phase 2:

Download YouTube audio

Transcribe audio

Merge documents

Rebuild index

Run evaluation cells

10. Limitations and Future Improvements
Limitations

Small dataset size

Word-based chunking may split concepts mid-thought

Strict grounding can feel conservative

Improvements

Add more sources (ethical fashion blogs, sustainability reports)

Add reranking (cross-encoder)

Add scoring harness (LLM-as-judge)

Add persona controls (budget sensitivity, dress code)
