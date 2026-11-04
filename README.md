# 👧🏼 Eira – Personal Voice AI Assistant

Eira is a hands-free, voice-driven AI assistant that can **listen**, **think**, **see**, and **talk back**.
You speak to it through your microphone. It works out an answer with an LLM agent and can look through your webcam when a question needs it ("What colour is my T-shirt?", "How many people do you see?"). Then it reads the answer back to you out loud. A Gradio web UI shows the live webcam feed next to the conversation.

---

## ✨ Features

- 🎤 **Continuous voice input**: listens automatically as soon as the app loads; no buttons to press.
- 📝 **Speech-to-text** with Groq's hosted `whisper-large-v3`.
- 🧠 **Agentic reasoning** with Google Gemini (`gemini-3.6-flash`) through a LangGraph ReAct agent.
- 👀 **Vision on demand**: the agent decides on its own when to take a webcam snapshot and analyse it with Groq's `llama-4-maverick` vision model.
- 🔊 **Text-to-speech** with gTTS (free, the default) or ElevenLabs (higher quality, needs a key).
- 🖥️ **Web UI** built with Gradio: live webcam feed plus chat history.
- 👋 Say **"goodbye"** to end the listening loop.

---

## 🏗️ How it works

```
 ┌────────────┐   record    ┌──────────────────┐   text   ┌────────────────────────┐
 │ Microphone │ ──────────▶ │ Groq Whisper STT │ ───────▶ │ Gemini ReAct agent     │
 └────────────┘             └──────────────────┘          │ (LangGraph)            │
                                                          │   └─ tool: webcam +    │
                                                          │      Groq vision model │
                                                          └───────────┬────────────┘
                                                                      │ reply
 ┌────────────┐   playback  ┌──────────────────┐                      │
 │  Speakers  │ ◀────────── │ gTTS/ElevenLabs  │ ◀────────────────────┘
 └────────────┘             └──────────────────┘
                     (chat history is streamed to the Gradio UI)
```

### Project structure

| File | Purpose |
|------|---------|
| [main.py](main.py) | Entry point. Builds the Gradio UI, runs the listen → think → speak loop, and manages the webcam feed. |
| [speech_to_text.py](speech_to_text.py) | Records from the microphone (`SpeechRecognition` + `PyAudio`) and transcribes it with Groq Whisper. |
| [ai_agent.py](ai_agent.py) | Defines Eira's personality (system prompt) and the LangGraph ReAct agent backed by Gemini. |
| [tools.py](tools.py) | The `analyze_image_with_query` tool: grabs a webcam frame with OpenCV and sends it plus the question to Groq's vision model. |
| [text_to_speech.py](text_to_speech.py) | Turns replies into speech with gTTS or ElevenLabs and plays them on macOS, Windows, or Linux. |
| [pyproject.toml](pyproject.toml) | Project metadata and dependencies. |

---

## 📋 Prerequisites

- **Python 3.12+**
- **[uv](https://docs.astral.sh/uv/)** (recommended) or plain `pip`
- **FFmpeg** on your `PATH` (`pydub` needs it to read and write MP3)
  - Windows: `winget install ffmpeg` or `choco install ffmpeg`
  - macOS: `brew install ffmpeg`
  - Linux: `sudo apt install ffmpeg`
- **PortAudio** (needed to build `PyAudio` on macOS/Linux; Windows wheels already include it)
  - macOS: `brew install portaudio`
  - Linux: `sudo apt install portaudio19-dev`
- A working **microphone**, **speakers**, and **webcam**
- API keys for:
  - **Groq**, for speech-to-text and vision: <https://console.groq.com/keys>
  - **Google AI Studio (Gemini)**, for the agent: <https://aistudio.google.com/app/apikey>
  - **ElevenLabs** *(optional)*, for premium TTS: <https://elevenlabs.io>

---

## 🚀 Setup

### 1. Clone the repository

```bash
git clone https://github.com/aryantiwariji007/Eira--Personal-Voice-AI-Assistant.git
cd Eira--Personal-Voice-AI-Assistant
```

### 2. Install dependencies

**With uv (recommended):**

```bash
uv sync
```

**Or with pip:**

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install -e .
```

### 3. Configure your API keys

Create a `.env` file in the project root. It's git-ignored, so your keys stay private:

```env
GROQ_API_KEY=your_groq_api_key
GOOGLE_API_KEY=your_gemini_api_key
ELEVENLABS_API_KEY=your_elevenlabs_api_key   # optional
```

### 4. Run Eira

```bash
uv run main.py
# or, inside an activated venv:
python main.py
```

Open **<http://localhost:7861>** in your browser. Gradio also prints a temporary public `*.gradio.live` link, because `share=True` is set in `main.py`.

---

## 🗣️ Usage

1. When the page loads, Eira starts listening right away. Watch the terminal for *"Start speaking now..."*.
2. Ask anything, for example:
   - "Tell me a fun fact about space."
   - "What am I holding right now?" *(triggers the webcam tool)*
   - "How many people do you see?"
3. Eira answers out loud, and the exchange appears in the chat panel.
4. Click **Start Camera** / **Stop Camera** to toggle the live preview, and **Clear Chat** to reset the history.
5. Say **"goodbye"** to stop the listening loop.

---

## ⚙️ Customisation

- **Switch to ElevenLabs voices**: in [main.py](main.py), uncomment the `text_to_speech_with_elevenlabs(...)` line and comment out the gTTS one. Change `voice_id` in [text_to_speech.py](text_to_speech.py) to use a different voice.
- **Change the personality**: edit `system_prompt` in [ai_agent.py](ai_agent.py).
- **Change the LLM**: change `model="gemini-3.6-flash"` in [ai_agent.py](ai_agent.py).
- **Change port / disable public sharing**: edit the `demo.launch(...)` arguments at the bottom of [main.py](main.py).

---

## 🛠️ Troubleshooting

| Problem | Fix |
|---------|-----|
| `Couldn't find ffmpeg` / MP3 errors | Install FFmpeg and make sure it's on your `PATH`. |
| `PyAudio` fails to install | Install PortAudio first (see Prerequisites). |
| `Could not open any webcam` | Close other apps using the camera, or check camera permissions. |
| Authentication errors | Check that `.env` exists in the project root and the keys are correct. |
| Microphone not picking up speech | Check your default input device; Eira calibrates for 1 second of ambient noise before it listens. |

---

## 🧰 Tech stack

[Gradio](https://www.gradio.app/) · [LangGraph](https://langchain-ai.github.io/langgraph/) · [LangChain Google GenAI](https://python.langchain.com/docs/integrations/chat/google_generative_ai/) · [Groq](https://groq.com/) · [OpenCV](https://opencv.org/) · [SpeechRecognition](https://pypi.org/project/SpeechRecognition/) · [pydub](https://github.com/jiaaro/pydub) · [gTTS](https://pypi.org/project/gTTS/) · [ElevenLabs](https://elevenlabs.io/)
