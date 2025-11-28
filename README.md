# 🎙️ AI Voice Interview Agent (100x-style)  
An end-to-end, voice-interactive AI interview system that uses real-time speech recognition, voice interruption, memory systems, and Gemini LLM to simulate a true conversational interview experience.

This project is built with **Vite + React** on the frontend and **Vercel Serverless Functions** for the backend.  

---

## 🚀 Features

### 🎤 **1. Real-time Voice Interaction**
- Start speaking anytime — the bot listens using browser SpeechRecognition.
- The bot responds using high-quality browser TTS.
- Automatic conversation looping (you talk ➝ bot replies ➝ listens again).

### 🛑 **2. Voice Interruption (Command Recognition)**
Interrupt the bot mid-sentence using voice:
- `"stop"`
- `"please stop"`
- `"clear transcript"`
- `"clear screen"`

The bot halts **LLM, TTS, and all microphones** instantly and waits for your next question.

### 🧠 **3. Short-Term Memory (Redis)**
The bot remembers:
- Your previous questions  
- Its own previous answers  

### 📚 **4. Long-Term Memory (Supabase)**
The system extracts **stable candidate facts** from each turn and stores them as durable memory:
- Skills  
- Experience  
- Goals  
- Personal attributes  

### 🔊 **5. Browser TTS with Natural Voice Selection**
Auto-selects best available:
- Google US English Female  
- Samantha  
- UK English Female  
- Falls back to pitch-tuned default if needed.

### 🌐 **6. Zero-Config Deployment**
Deploy by simply pushing to Vercel:
- Frontend + API in one repo  
- Automatic build  
- Environment variable support  

---

### ⚠️ **Challenges I Faced …and How I Fought Through Them**

### **1. Whisper API Limitations (Premium / Paid)**

**The challenge:**
I initially planned to use Whisper for real-time speech-to-text because of its accuracy.

But Whisper:
- Is not available on the free tier
- Requires paid billing to even test at scale
- Would drastically increase project cost for continuous conversation loops

**How I overcame it:**
I pivoted to using the browser’s built-in SpeechRecognition API, which let me maintain:
- Continuous real-time voice capture
- Fast detection
- Zero cost
- No rate limits

This allowed the project to stay fully deployable on Vercel without paid dependencies.

### **2. OpenAI Realtime API (WebRTC Complexity)**

**The challenge:**
OpenAI’s Realtime API requires:
- WebRTC connections
- Bidirectional audio streams
- Event-based state handling
- Always-on sockets

This conflicted hard with my architecture:
- I needed interruptibility
- I needed clean loop restarts
- I needed a lightweight deployment
- WebRTC broke everything whenever I tried to inject my custom memory logic and conversation loop.

**How I overcame it:**
- I scrapped WebRTC entirely and instead engineered:
- A dual recognition system (one recognizer for user questions, one for voice commands)
- A TTS-driven loop that restarts listening after every bot answer
- A clean “STOP” and “CLEAR” voice-interrupt system
- 100% stateless serverless endpoints

This made the system smooth, predictable, and browser-first.

### **3. Voice Interruption Logic (The Hardest Part)**

**The challenge:**
When the bot speaks, the browser:
- Locks the audio pipeline
- Cancels recognition
- Prevents user interruption

This made it almost impossible to talk over the bot.

**How I overcame it:**
I built a manual override system:
- During TTS playback → launch a parallel recognizer only listening for command words
- If user says “STOP” or “CLEAR” →
→ cancel speech
→ kill both recognizers
→ reset the system
→ immediately start listening again for the next question

This replicated the behavior of advanced assistants like Alexa / Google Assistant.

### **4. Redis + Supabase Memory (Serverless)**

**The challenge:**
Converting the full Node backend into serverless functions meant:
- No persistent process
- No local state
- No in-memory cache
- CORS issues
- Path restructuring
- Ensuring Redis and Supabase clients behave in serverless cold boots

**How I overcame it:**
I restructured the entire backend into:
- /api/text-interview.js (serverless logic)
- _memory.js (Upstash Redis for short-term memory)
- longMemory.js (Supabase for long-term LTM)

All stateless, all serverless, all Vercel-ready.

### **5. Browser Conflicts (Speech vs. Speech)**

**The challenge:**
Browsers do NOT like:
- SpeechRecognition + speechSynthesis running at the same time
- Two recognisers running together
- Auto-restarting loops
- Chrome kept stopping unexpectedly.

**How I overcame it:**
I built a full coordinator:
- A unified stopAllOperations()
- A strict order of who’s allowed to speak or listen
- Auto-cleanup of abandoned recognizers
- Intelligent restarting after each event cycle

This made the entire loop stable, even though browsers don’t naturally support this flow.

