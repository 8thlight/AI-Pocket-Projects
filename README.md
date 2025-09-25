📦 AI Pocket Projects

**AI Pocket Projects** is a collection of small, composable AI engineering projects. 
Each project is designed to be practical, portable, and teachable — giving you hands-on experience with key AI system patterns in under a week.

The three core projects are:

T-RAG → a minimal Retrieval-Augmented Generation (RAG) system.
Voice Layer → Pipecat-driven voice interface for natural conversation.
Web MCP → a Model Context Protocol (MCP) server for search + scrape-to-Markdown.
Together, they demonstrate how to scaffold AI-powered systems from knowledge retrieval → multi-modal interaction → external tool augmentation.

🚀 Why Pocket Agents?

Concrete → Each project has a crisp end-to-end workflow.
Composable → Each agent can stand alone or plug into the others.
Practical → Focus on usable utilities (retrieval, ops, research).
Teachable → Each is designed to illustrate core AI engineering principles.
1. T-RAG: Tiny Retrieval-Augmented Generation
Goal
A minimal RAG system that can answer questions over a small corpus (Markdown, PDFs, HTML) with citations, latency targets, and evaluation hooks.
Scope
Ingest: Chunk + embed documents into a vector store.
Retrieve: Top-k retrieval, with optional reranker.
Generate: Answer via a chat model (LiteLLM gateway).
Convo memory: Manage last N turns as a separate retrieval domain.
Eval: Golden Q/A set with promptfoo tests.

Data

20–40 Markdown docs (short blog posts, white papers, FAQs).

One JSON or Markdown “conversation log” for retrieval across past chats.

Techniques

Token-aware chunking (600–900 tokens, 120–160 overlap).

SQLite or Chroma vector storage.

Dual retrievers (docs + convo), rerank fusion.

Prompt guardrails (“use only provided sources, cite them”).

Milestones

MVP ingest + ask API.

Add citations + conversation memory.

Add reranker + latency logging.

Add eval set (≥10 golden Qs).

2. Voice Layer: Pipecat Voice Interface
Goal

Enable full-duplex voice conversation with T-RAG: speak a question, hear the answer with citations.

Scope

ASR (speech → text) using Whisper or Deepgram.

TTS (text → speech) using OpenAI TTS or Piper.

Pipeline orchestration with Pipecat.

Web UI: microphone input, transcript view, clickable citations.

Techniques

Voice activity detection (push-to-talk or VAD).

End-to-end latency target: ≤1.5s for first response.

Barge-in cancellation: stop TTS if the user interrupts.

Milestones

CLI-only voice loop.

Web UI with live transcript.

Citations visible in UI alongside audio.

3. Web MCP: Search + Scrape-to-Markdown Server
Goal

Expose a Model Context Protocol (MCP) server with two tools:

web.search(query) → search provider (e.g. Serper, Tavily).

web.fetch_to_md(url) → fetch + clean HTML → Markdown file with front matter.

This provides live knowledge injection for T-RAG.

Scope

MCP stdio JSON-RPC server in TypeScript.

Pluggable search adapter (swap providers).

Local scraper with Readability + Markdown conversion.

Saves .md docs into /data/md_out so T-RAG can ingest them.

Techniques

Domain allow-list (ALLOWED_DOMAINS).

Resource limits (3 results, 1MB cap, 10s timeout).

Front matter for each Markdown file (title, source, fetched_at).

Milestones

Basic MCP handshake + ping.

Implement web.search.

Implement web.fetch_to_md.

Integration: feed Markdown into T-RAG /ingest.

🛠️ Tools & Frameworks

Languages: Python (T-RAG), TypeScript (MCP), JS/TS (UI).

Libraries:

FastAPI / Express (APIs)

SQLite + sqlite-vec / Chroma (vector store)

LiteLLM (model gateway)

Pipecat (voice pipeline)

Turndown / Readability (scraping → Markdown)

promptfoo (evaluation)

Infra: Docker Compose for local dev.

📂 Repo Structure
pocket-agents/
  api/             # T-RAG FastAPI server
  rag/             # retrieval, chunking, embeddings
  data/            # corpus + scraped Markdown
  eval/            # golden Q/A sets + promptfoo config
  voice/           # Pipecat voice loop + web UI
  mcp/             # MCP server with search + fetch_to_md
  docs/            # project docs + diagrams
  README.md

✅ Evaluation & Acceptance Criteria
T-RAG

≥75% “fully supported” in promptfoo golden set.

p50 latency <800ms for 100 docs.

Always returns citations or explicit “I don’t know.”

Voice Layer

Wake-to-response p50 ≤1.5s.

Barge-in cancellation ≤150ms.

UI always displays transcript + citations.

Web MCP

Search returns 3–5 results with {title,url,snippet}.

Fetch saves Markdown with valid front matter.

Invalid inputs (404s, oversized pages) handled gracefully.

📅 Suggested Build Sequence

Week 1 → T-RAG MVP (ingest + ask).

Week 2 → Add conversation memory + evals.

Week 3 → Build MCP server (search + scrape).

Week 4 → Add Pipecat voice layer + polish demos.
