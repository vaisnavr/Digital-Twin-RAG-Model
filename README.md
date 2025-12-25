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
