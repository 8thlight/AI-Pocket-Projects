# 🧠 Phase 1: RAG (Retrieval-Augmented Generation)

> **Build a knowledge system that never responds without citations**

## 🎯 What You're Building

A RAG system that can:
- Ingest documents from the `../../data/corpus/` directory
- Answer questions about AI and computing topics
- **Always cite sources** - never respond without attribution. If no sources are used say so.
- Monitor performance with Langfuse, log the prompt, rag chunks and response
- Evaluate responses using LLM-as-a-judge in real time
  - **Citation Compliance Check**: 100% of responses must include source attribution
  - **Retrieval Quality Assessment**: Measure precision/recall of chunk retrieval

## 🗺️ Your 2-Week Sprint

### **Week 1: Core RAG System**

#### **Day 1-3: Document Ingestion & Vector Storage**
**Your Mission:** Load, chunk, and embed all corpus documents in one go.

**What to Build:**
- Document loader for all `.md` files in `data/corpus/ai/` and `data/corpus/computing/`
- Chunking system (800-token pieces, 200-token overlap) with metadata preservation
- Vector database: **Pinecone (simplest - web-based)**, Chroma (local), or SQLite-vec (lightweight)
- Embeddings: **OpenAI text-embedding-3-large** or Hugging Face sentence-transformers
- Basic search functionality that retrieves relevant chunks

**🎮 Side Quest:** Try semantic chunking and hybrid search (vector + keyword matching).

**⚠️ Gotcha:** Preserve document metadata - you'll need it for citations! If using Pinecone + OpenAI embeddings, you get the simplest setup but highest cost. Hugging Face + Chroma is free but requires more setup.

#### **Day 4-7: Generation with Mandatory Citations**
**Your Mission:** Build citation-aware response generation with monitoring.

**What to Build:**
- Citation-aware prompt engineering that never responds without sources
- Response validation (no citations = "I don't know")
- Langfuse integration for logging prompts, chunks, and responses
- Basic test question set (10+ questions from your corpus)

**🎮 Side Quest:** Experiment with different citation formats in Langfuse playground.

**⚠️ Gotcha:** LLMs will try to answer without sources - be ruthless with validation!

**📸 Citation Example:** See [`citation example.png`](citation%20example.png) for a real-world example from InsightMesh showing proper source attribution in action.


### **Week 2: Evaluation & Polish**

#### **Day 8-10: Automated Evaluation**
**Your Mission:** Implement both core evaluations and LLM-as-a-judge.

**What to Build:**
- Citation Compliance Check (automated validation of 100% attribution rate)
- Retrieval Quality Assessment (precision/recall metrics for chunk retrieval)
- LLM-as-a-judge pipeline for response quality scoring
- Expanded test dataset (20+ questions)

**🎮 Side Quest:** Build adversarial test questions and a retrieval debugger.

**⚠️ Gotcha:** Validate LLM judge results manually - judges can be biased too! 

#### **Day 11-14: System Optimization & Documentation**
**Your Mission:** Polish your system and document learnings.

**What to Build:**
- Performance monitoring dashboard in Langfuse
- Error handling for edge cases (empty results, API failures)
- A/B testing setup for different prompt templates
- Documentation of what worked, what didn't, and key learnings

**🎮 Side Quest:** Test different chunking strategies and embedding models.

**⚠️ Gotcha:** Don't over-optimize - focus on citation compliance and basic accuracy first!

## 📊 Two Essential RAG Evaluations

### **Evaluation 1: Citation Compliance Check**
**Purpose:** Ensure 100% of responses include proper source attribution.

**How it Works:**
- For every response, check if it contains citation markers like `[Source: filename.md]`
- If no citation found, check if response is "I don't have enough information" or similar
- Flag any response that makes claims without citations

**Success Metric:** 100% compliance rate - zero tolerance for uncited responses

**🎮 Side Quest:** Build a regex pattern that catches different citation formats and validates them against your actual source files.

**⚠️ Gotcha:** Don't just check for citation format - verify the cited files actually exist in your corpus!

### **Evaluation 2: Retrieval Quality Assessment**
**Purpose:** Measure how well your system finds relevant information.

**How it Works:**
- For each test question, manually identify which corpus files contain the answer
- Compare your system's retrieved chunks against the "golden" relevant files
- Calculate precision (% of retrieved chunks that are relevant) and recall (% of relevant info that was retrieved)

**Success Metrics:**
- Precision >80% (most retrieved chunks should be useful)
- Recall >70% (shouldn't miss important information)
- Top-3 retrieval should contain the answer for 90% of answerable questions

**🎮 Side Quest:** Create a "retrieval debugger" that shows you exactly which chunks were retrieved for each question, so you can spot patterns in what your system misses.

**⚠️ Gotcha:** High precision but low recall means your system is too conservative. Low precision but high recall means it's grabbing irrelevant chunks. Balance is key!

## 🧪 Testing Your System

**Create Test Questions from Your Corpus:**
- Write 20+ questions that can be answered from your AI and computing materials
- Include easy questions ("What is machine learning?") and hard ones ("How did WWII influence computer development?")
- Add trick questions that can't be answered from your sources

**🎮 Side Quest:** Create questions that require combining information from multiple files. Can your system cite both `ai_bias.md` and `transformers.md` in one response?

**⚠️ Gotcha:** Don't just test happy paths! Try questions with typos, very long questions, and questions in different languages.

## 🎯 Success Criteria

By the end of your 2-week sprint, you should have:
- [ ] **100% Citation Rate**: Every response includes source attribution or says "I don't know"
- [ ] **>75% Accuracy**: On your test question set
- [ ] **Langfuse Integration**: All queries logged and monitored
- [ ] **LLM Judge Working**: Automated evaluation pipeline
- [ ] **Corpus Coverage**: Can answer questions from both AI and computing materials

## 🔧 Technical Setup

**Required Dependencies:**
- `openai` - For embeddings and LLM calls
- `langchain` - For document processing
- `langfuse` - For monitoring and evaluation
- `tiktoken` - For token counting

**Vector Database Options (choose one):**
- `pinecone-client` - **Recommended for beginners** (web-based, no setup)
- `chromadb` - For local development
- `sqlite-vec` - Lightweight embedded option

**Embedding Options (choose one):**
- **OpenAI text-embedding-3-small** - Best quality, costs ~$0.02/1M tokens
- **Hugging Face sentence-transformers** - Free, runs locally (e.g., `all-MiniLM-L6-v2`)

**Environment Variables:**
- `OPENAI_API_KEY` - Your OpenAI API key (required for embeddings/LLM)
- `PINECONE_API_KEY` - Your Pinecone API key (if using Pinecone)
- `LANGFUSE_SECRET_KEY` - Your Langfuse secret key
- `LANGFUSE_PUBLIC_KEY` - Your Langfuse public key
- `LANGFUSE_HOST` - Usually `https://cloud.langfuse.com`

## 📚 Learning Resources

**From Your Corpus:**
- [RAG Introduction](../../data/corpus/ai/rag_intro.md) - Start here for basic concepts
- [RAG Challenges](../../data/corpus/ai/rag_challenges.md) - Common problems you'll face
- [Prompt Engineering](../../data/corpus/ai/prompt_engineering.md) - Essential for citation prompts

**External Resources:**
- [Langfuse Docs](https://langfuse.com/docs) - For monitoring setup
- [Chroma Docs](https://docs.trychroma.com/) - Vector database guide
- [OpenAI Embeddings](https://platform.openai.com/docs/guides/embeddings) - Best practices

## 🚨 Critical Success Factors

**1. Citations Are Non-Negotiable**
- Never let your system respond without proper source attribution
- "I don't have enough information" is better than an uncited answer
- Validate every response before returning it

**2. Test with Your Own Data**
- Your corpus has rich AI and computing content - use it!
- Create questions that span multiple documents
- Test edge cases where no good answer exists

**3. Monitor Everything**
- Use Langfuse to track query patterns and response quality
- Set up alerts for low citation rates or poor accuracy
- Experiment with different prompts and measure the results

## 🎯 Ready to Start?

1. **Set up your development environment** with the dependencies listed above
2. **Start with document loading** - get your corpus ingested first
3. **Build incrementally** - test each component as you build it
4. **Use AI assistance** - ask Claude/GPT to help with implementation details, architecture, testing, design, etc. Lean into collaboration with it. 
5. **Focus on citations** - this is your #1 priority throughout

## ➡️ Next Phase

Once you have a working RAG system with 100% citation compliance and >75% accuracy, move to **Phase 2: Voice Interface** in [`../2. Voice/README.md`](../2.%20Voice/README.md).

You'll connect your RAG system to a voice interface using Pipecat or OpenAI's Realtime API!

---

**Remember:** The goal is learning by building, not perfect code on the first try. Use your AI coding assistant liberally and focus on getting something working quickly! 🚀