# Digital-Twin-RAG-Model
Project: Customer Persona Twin — Trend-Conscious Sustainable Shopper (Persona Mimic)
 Notebook: Final LLM Project.ipynb

1. Project Overview
This project builds a Customer Persona Twin that mimics how a fashion-forward, sustainability-aware shopper thinks and speaks when asked fall styling questions. The twin is not a “stylist textbook” or a generic recommender—it is designed to respond with the persona’s tone, tradeoffs, and priorities (trend relevance + ethics + practicality).
The system is implemented as a Retrieval-Augmented Generation (RAG) pipeline on top of Gemini:
A user asks a question (e.g., “How do I upgrade my fall wardrobe sustainably?”)


The system retrieves relevant evidence snippets from a custom knowledge base


Gemini generates an answer in the persona’s voice grounded in retrieved context


This satisfies the project requirement to adapt a pretrained LLM using domain data, prompt engineering, RAG, and evaluation.


2. Digital Twin Definition
2.1 Digital Twin Type
Customer Persona Twin — Persona Mimic
2.2 Persona
Trend-Conscious Sustainable Shopper
Fashion-forward but practical


Influenced by trends/social media, but avoids fast fashion excess


Strong preference for ethical/sustainable choices (materials, thrifting, cost-per-wear)


Likes confident, curated recommendations and “wearable” trend translation


2.3 Twin Behavior
Persona Mimic
Responds like the shopper would talk


Makes realistic tradeoffs (trend + sustainability + versatility)


Uses “evidence used” citations from retrieved chunks to show grounding



3. Data Sources and Preprocessing (2 Sources)
Your notebook uses two distinct data sources, meeting the course requirement.
3.1 Data Source 1 — Reddit Threads (Fashion Discussions)
Goal: capture authentic language + real consumer opinions about trends, styling, practicality, and brand perceptions.
What the code does
Scrapes Reddit threads using Reddit’s JSON endpoints (via requests), including:


post title + post body


comment text (and optionally “more comments” expansion)


Filters to relevant subreddit: keeps only r/femalefashionadvice for fashion persona relevance.


Writes the scraped output to a JSONL file (one thread per line).


Preprocessing
Builds a single “document text” per thread: title + post + first N comments (to control size/cost)


Strips newlines and formats comments into readable bullets like - u/author: comment


3.2 Data Source 2 — YouTube (Audio → Transcript)
Goal: add higher-level trend guidance and seasonal fashion commentary often found in video essays and fashion channels.
What the code does
Downloads audio from a list of YouTube URLs using yt-dlp (audio-only MP3s).


Uploads MP3 files to Gemini and transcribes them into structured JSON objects (written to youtube_transcripts.jsonl).


Converts transcripts into “docs” and appends them with Reddit docs into a single combined corpus file:


all_docs_v2.jsonl


Preprocessing
Converts each transcript to plain text with a doc_id like youtube_<video_id>


Stores fields such as source_type, url, and text



4. Retrieval-Augmented Generation (RAG) Pipeline
Your notebook implements a full RAG system as required.
4.1 Chunking Strategy
To support retrieval, long documents are split into manageable pieces.
Reddit chunking
Word-based chunking (default ~250 words per chunk)


Each chunk includes metadata:


source (“reddit”)


post_id, title, url


chunk_id like reddit_<postid>_c000


Combined corpus chunking (Reddit + YouTube)
Same chunk size (~250 words)


Chunk metadata includes source (“reddit” or “youtube”) and unique chunk_id


Why chunking matters
Smaller chunks improve retrieval precision


Keeps prompt context short enough for the model while still informative


4.2 Embeddings
Each chunk is converted into a vector embedding using:
Gemini embedding model: gemini-embedding-001


The notebook defines an embed(text) function that returns a float32 numpy vector.
4.3 Vector Index (FAISS)
Embeddings are stored in FAISS (IndexFlatL2) for fast similarity search:
Build embedding matrix → index.add(vectors)


At query time: embed the query → index.search(query_vector, k)


4.4 Retrieval Function
retrieve(query, k) returns the top-k most similar chunks (by vector distance).
 These chunks become the grounding CONTEXT for generation.
4.5 Generation Step
The system constructs:
context = join(["[chunk_id] chunk_text", ...])


A persona prompt + grounding rules + response format


Then calls Gemini (e.g., models/gemini-2.5-flash) to generate the final answer.


4.6 Reliability / Error Handling
A safe_generate() wrapper retries on transient Gemini server errors to make demos more stable (important for classroom demos).

5. Prompt Engineering (Iteration Evidence)
The assignment requires at least 2 prompt iterations and documentation of prompt techniques.
DSO599_Final_Project (1)
5.1 Prompt v1 (Initial Persona + Structured Output)
The first RAG prompt:
Defines the persona: fashion-forward + sustainability-aware


Adds grounding rule: “Use ONLY the CONTEXT”


Produces a structured response with sections like:


Style direction


Outfit formulas


Sustainable swaps


What I’d avoid


Cited chunk IDs (“signals”)


Technique used:
Role prompting (persona definition)


Grounding instruction (no hallucinated brands/facts)


Output schema (consistent formatting)


5.2 Prompt v2 (More “Mimic” + Messaging Test)
The second prompt iteration strengthens the “customer voice” and makes the twin more product-valuable by adding:
Stronger voice constraints (“concise, confident, trend-aware”)


Explicit dislikes (avoid fast-fashion, synthetic-heavy hype)


A new section: “Messaging test” — generates 2 marketing lines that would appeal to the persona


A required evidence list (“Evidence used”)


Why this improves the twin
Moves beyond styling into business value: brands can test messaging on the persona twin


Forces consistent outputs for easier evaluation/comparison



6. Demo Queries and Baselines (RAG vs No-RAG)
Your notebook demonstrates RAG impact by comparing:
Baseline: Gemini answers directly from the user question (no retrieval)


RAG: Gemini answers with retrieved context injected


Example demo/evaluation-style questions in the notebook:
Fall styling formulas that are trendy but sustainable


Materials to prioritize for sustainable fall basics


“Best sustainable shoe brands for fall 2025” (a deliberately tricky one that stresses grounding)


This comparison aligns with the requirement to show cases where RAG improves responses vs no retrieval.

7. Evaluation Design (What “Good” Means)
The assignment asks for a small but meaningful evaluation and comparison (RAG vs no RAG, prompt v1 vs v2, etc.).
7.1 Evaluation Criteria (Rubric)
A practical rubric for this persona mimic:
Persona voice match


confident, trend-aware, not overly academic


Sustainability alignment


mentions materials, thrifting, cost-per-wear, ethics


Trend + fall relevance


seasonal colors, layering logic, wearable trends


Grounding / non-hallucination


uses only retrieved context; avoids inventing brands/facts


Structure compliance


follows the required output format reliably


Evidence transparency


includes chunk IDs (“Evidence used”)


7.2 Evaluation Method in Notebook
Runs multiple demo questions through:


baseline generation


RAG generation with k set to values like 8–12


Performs a “retrieval sanity check”:


prints chunk IDs and sources returned by retrieve()


increases k to confirm YouTube + Reddit both appear in results after Phase 2


Adds a safety fallback:


if model forgets “Evidence used”, the code appends the retrieved chunk IDs automatically (helps consistency for grading/demo)


7.3 Expected Findings (How to Report Results)
When you write your results section (for slides/report), you can summarize patterns like:
Baseline tends to be fluent but may be generic or ungrounded.


RAG improves:


specificity (pulls real phrasing, real constraints from discussions/transcripts)


consistency with persona preferences (because evidence contains those preferences)


transparency (chunk ID evidence)


Weakness case: “best sustainable shoe brands for fall 2025”


With strict grounding rules, RAG may correctly refuse to name brands if they’re not in context (which is actually a win for non-hallucination).



8. System Architecture Summary
Input: user fashion question
 Step A: chunked corpus (Reddit + YouTube) stored as JSONL
 Step B: embed chunks with gemini-embedding-001
 Step C: FAISS similarity search retrieves top-k chunks
 Step D: prompt template injects persona + retrieved context
 Step E: Gemini generates response in the persona voice + evidence

9. How to Run (Repro Steps)
Install dependencies (praw, requests, faiss-cpu, yt-dlp, google-generativeai/google genai)


Set GEMINI_API_KEY securely (environment/secret manager—avoid hardcoding in final submission)


Run Phase 1:


scrape Reddit → build chunks → embed → FAISS → run RAG demo


Run Phase 2:


download YouTube audio → transcribe → merge docs → rebuild index


Run evaluation cells:


compare baseline vs RAG and capture outputs for slides



10. Limitations and Future Improvements
Limitations
Small dataset size (few threads + limited number of YouTube videos)


Word-based chunking may split concepts mid-thought


“Use ONLY context” can lead to conservative answers (good for grounding, but may feel less helpful)


Improvements
Add more sources (ethical fashion blogs, brand sustainability reports)


Add reranking (cross-encoder reranker) to improve retrieval quality


Add a scoring harness (LLM-as-judge or checklist scoring) over 10+ queries


Add persona memory knobs (e.g., budget sensitivity slider, workplace dress code)

