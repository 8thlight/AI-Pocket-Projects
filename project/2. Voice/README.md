# Phase 2: Voice Interface 🎙️

*Building real-time, conversational AI that feels natural*

## What You're Building

A voice interface system that can:
- Accept real-time audio input with natural interruption handling
- Process speech-to-text with low latency (< 500ms)
- Generate contextual responses using your RAG system from Phase 1
- Convert responses to natural-sounding speech
- Stream audio back with minimal delay for fluid conversation
- Handle conversation state and context across multiple exchanges

**📚 Working Example:** [AI Operator](https://github.com/Kode-Rex/ai-operator) - Full voice pipeline with Pipecat framework

## Your Mission: 2-Week Voice Sprint

Transform your RAG system into a conversational voice assistant that people actually want to talk to.

### **Week 1: Core Voice Pipeline**

#### **Day 1-2: Speech-to-Text Foundation**
**Your Mission:** Get audio flowing from microphone to text with rock-solid reliability.

**What to Build:**
- **Deepgram Nova-2** for speech recognition (best accuracy/speed balance)
- WebSocket connection for real-time streaming
- Audio capture with proper sample rates (16kHz recommended)
- Voice Activity Detection (VAD) to know when user stops speaking
- Basic interruption handling - user can cut off the AI mid-sentence

**🎮 Side Quest:** Try OpenAI Whisper for comparison, experiment with different audio preprocessing.

**⚠️ Gotcha:** Audio buffering is tricky - too small and you get choppy recognition, too large and latency suffers. Start with 100ms chunks.

#### **Day 3-4: LLM Integration & Context**
**Your Mission:** Connect your RAG system to handle voice queries with conversation memory.

**What to Build:**
- Integrate your Phase 1 RAG system as the "brain"
- Conversation state management - remember what was discussed
- Prompt engineering for spoken responses (shorter, more conversational)
- Response streaming to reduce perceived latency

**🎮 Side Quest:** Add personality prompts to make the AI more engaging in conversation.

#### **Day 5-7: Text-to-Speech & Audio Streaming**
**Your Mission:** Turn text responses into natural, streaming audio.

**What to Build:**
- **Cartesia Sonic** or **ElevenLabs** for high-quality TTS
- Audio streaming - start playing while still generating
- WebSocket audio streaming to client
- Proper audio format handling (PCM, sample rates)
- Basic emotion/tone control for more engaging responses

**🎮 Side Quest:** Try voice cloning with ElevenLabs, experiment with different speaking styles.

**⚠️ Gotcha:** Audio streaming is complex - you need proper buffering and format conversion. This is why libraries like PipeCat are so critical. 

### **Week 2: Advanced Features & Polish**

#### **Day 8-10: Real-time Optimization**
**Your Mission:** Achieve sub-500ms total latency for natural conversation flow.

**What to Build:**
- **Pipecat framework** integration for optimized pipeline orchestration
- Parallel processing - start TTS while RAG is still retrieving
- Audio preprocessing optimization
- Connection pooling for external APIs
- Latency monitoring and optimization

**🎮 Side Quest:** Try **OpenAI Realtime API** for end-to-end voice (currently in beta).

**⚠️ Gotcha:** Real-time audio is unforgiving. One slow component ruins the entire experience. Profile everything.

#### **Day 11-12: Interruption & Flow Control**
**Your Mission:** Handle natural conversation patterns like interruptions and clarifications.

**What to Build:**
- Sophisticated interruption detection - stop TTS immediately when user speaks
- Conversation repair - "Sorry, what was that?" handling
- Context switching - user changes topics mid-conversation
- Confirmation patterns - "Did you mean X or Y?"
- Graceful error handling with spoken feedback

**🎮 Side Quest:** Add conversation summarization for long chats, implement "repeat that" functionality.

**⚠️ Gotcha:** Interruptions are the hardest part. Users expect instant response when they start speaking.

#### **Day 13-14: Production Polish & Web Interface**
**Your Mission:** Make it production-ready with monitoring, reliability, and a polished demo interface.

**What to Build:**
- Comprehensive error handling with user-friendly voice feedback
- Audio quality monitoring and fallback strategies
- Rate limiting and cost management for APIs
- Conversation logging and analytics
- **Interactive web interface** with real-time audio visualizer

**Web Interface Features:**
- Clean, modern UI with large "Push to Talk" or "Voice Active" button
- **Real-time audio visualizer** showing input levels and voice activity
- Live conversation transcript with source citations highlighted
- Connection status indicators (STT, LLM, TTS pipeline status)
- Basic controls: mute, volume, clear conversation
- Mobile-responsive design for phone/tablet testing

**🎮 Side Quest:** Add waveform visualization during TTS playback, conversation export features, multi-language support.

**⚠️ Gotcha:** Voice interfaces fail in ways text interfaces don't. Always have fallback strategies. Audio visualizers can be CPU-intensive - use requestAnimationFrame and throttle updates.

## Redis Session Memory Management

Voice conversations generate lots of data quickly. Smart memory management is crucial for performance and cost control.

### **What to Store in Redis**

**Essential (Always Store):**
- Last 5-10 conversation turns (user + assistant messages)
- Current conversation context/topic
- User preferences (speaking speed, voice type)
- Session metadata (start time, total turns, user ID)

**Contextual (Store Strategically):**
- RAG source documents used (for follow-up questions)
- Entity mentions and their context
- Conversation summary for long sessions
- Error states and recovery context

**Avoid Storing:**
- Full audio buffers (too large, use streaming)
- Complete conversation history (summarize instead)
- Temporary processing states
- API response caches (use separate cache layer)

### **Memory Management Strategies**

**Token-Based Pruning:**
```python
# Use tiktoken to count tokens and prune when needed
import tiktoken

def should_prune_memory(conversation_history, max_tokens=4000):
    encoding = tiktoken.encoding_for_model("gpt-4")
    total_tokens = sum(len(encoding.encode(msg)) for msg in conversation_history)
    return total_tokens > max_tokens
```

**Sliding Window Approach:**
- Keep last N turns + conversation summary
- When window full, summarize oldest turns and remove details
- Preserve important context (user name, preferences, key facts)

**Importance-Based Retention:**
- Score messages by importance (questions > statements, errors > success)
- Keep high-importance messages longer
- Use semantic similarity to avoid storing redundant information

### **Supporting Libraries**

**LangChain Memory Classes:**
- `ConversationSummaryBufferMemory` - Automatic summarization when full
- `ConversationTokenBufferMemory` - Token-based memory limits
- `ConversationEntityMemory` - Track entities mentioned in conversation

**Custom Redis Patterns:**
- Use Redis EXPIRE for automatic cleanup
- Hash structures for efficient updates
- Sorted sets for time-based pruning
- Pub/Sub for real-time memory updates across instances

**Memory Decision Framework:**
1. **Recency**: Recent messages more important than old ones
2. **Relevance**: Messages related to current topic get priority
3. **User Intent**: Questions and requests more important than acknowledgments
4. **Error Recovery**: Keep context around errors for better handling
5. **Citations**: Always preserve source attribution for fact-checking

**🎮 Side Quest:** Implement conversation branching - let users say "go back to when we were discussing X"

**⚠️ Gotcha:** Voice conversations feel more personal than text. Users expect the AI to "remember" more context, but storing everything kills performance.

## Tech Stack Recommendations

**Speech-to-Text:**
- **Deepgram Nova-2** (best balance of speed/accuracy)
- OpenAI Whisper (high accuracy, higher latency)
- AssemblyAI (good real-time features)

**Text-to-Speech:**
- **Cartesia Sonic** (ultra-low latency, great for real-time)
- ElevenLabs (highest quality, good for demos)
- OpenAI TTS (solid middle ground)

**Framework:**
- **Pipecat** (purpose-built for voice AI pipelines)
- Custom WebSocket implementation
- OpenAI Realtime API (experimental, all-in-one)

**Infrastructure:**
- WebRTC for audio streaming
- **Redis for conversation state** with smart memory management
- WebSocket connections for real-time communication

**Redis Session Management:**
- **redis-py** or **ioredis** for Redis client
- **LangChain Memory** classes for conversation management
- **tiktoken** for token counting and memory pruning
- **Custom memory strategies** for voice-specific needs

**Web Interface:**
- **Web Audio API** for microphone access and audio processing
- **Canvas API** or **Web Audio Analyser** for real-time visualizations
- WebSocket client for real-time communication
- Modern CSS/JavaScript (or React/Vue if preferred)
- Responsive design frameworks (Tailwind CSS, Bootstrap)

## Environment Variables

```bash
# Speech Recognition
DEEPGRAM_API_KEY=your_deepgram_key

# Text-to-Speech  
CARTESIA_API_KEY=your_cartesia_key
ELEVENLABS_API_KEY=your_elevenlabs_key

# LLM & RAG (from Phase 1)
OPENAI_API_KEY=your_openai_key
LANGFUSE_SECRET_KEY=your_langfuse_secret
LANGFUSE_PUBLIC_KEY=your_langfuse_public

# Optional
ASSEMBLYAI_API_KEY=your_assemblyai_key
```

## Success Criteria

By the end of 2 weeks, you should have:
- ✅ Sub-500ms end-to-end latency for simple queries
- ✅ Natural interruption handling that feels responsive
- ✅ Reliable source citation in spoken responses
- ✅ Conversation memory that maintains context
- ✅ Production-ready error handling and fallbacks
- ✅ Working web demo that showcases the capabilities

## Advanced Challenges

**🚀 Voice Personality:** Give your AI a consistent speaking style and personality
**🚀 Multi-turn RAG:** Handle complex queries that require multiple document lookups
**🚀 Voice Commands:** Add special commands like "search for X" or "summarize that"
**🚀 Emotion Detection:** Respond to user emotional state in voice tone
**🚀 Background Processing:** Continue processing while user is speaking

## Common Gotchas

**Audio Format Hell:** Different browsers, devices, and APIs expect different audio formats. Test extensively.

**Latency Creep:** Each component adds latency. 50ms here, 100ms there, suddenly you're at 2 seconds total.

**WebSocket Reliability:** Connections drop, especially on mobile. Implement robust reconnection logic.

**Citation in Speech:** "According to document slash data slash corpus slash AI slash deep underscore learning dot MD" sounds terrible. Make it natural.

**Memory Leaks:** Audio buffers and WebSocket connections can leak memory fast in long conversations.

**API Rate Limits:** Voice generates lots of API calls quickly. Monitor usage and implement queuing.

**Cross-Platform Audio:** What works on Chrome desktop might fail on Safari mobile.

Remember: Voice interfaces are judged by their worst moment, not their average performance. Focus on reliability and graceful degradation over fancy features.

---

**Next Phase:** [Web Intelligence (MCP)](../3.%20MCP/README.md) - Add web search and real-time information to your platform.
