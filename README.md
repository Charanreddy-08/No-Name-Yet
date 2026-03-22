# AI Intent Launcher – Zero App Searching

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Kotlin](https://img.shields.io/badge/kotlin-1.9.0-purple.svg)](https://kotlinlang.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100.0+-009688.svg)](https://fastapi.tiangolo.com/)

**AI Intent Launcher** is a semantic command layer for your smartphone. Instead of hunting through folders and clicking multiple buttons, users simply state their goal in natural language. The system bridges the gap between human language and application-specific actions using LLMs, deep linking, and OS-level automation.

---

## 📌 Project Overview

Traditional app usage is "App-Centric": Find App → Open App → Search Feature → Execute.
**AI Intent Launcher** is "Intent-Centric": State Goal → System Executes.

### Supported Commands:
* 🚖 *"Book a cab to the airport"*
* 💬 *"Tell Mom I'll be there in 10 minutes on WhatsApp"*
* 🎵 *"Play some lo-fi beats on Spotify"*
* 📅 *"Remind me to call the dentist tomorrow at 4 PM"*

---

## 🎯 Key Features

*   **LLM-Powered Intent Parsing:** Converts messy human language into structured JSON actions.
*   **Zero-UI Execution:** Directly triggers deep links or API calls, bypassing app landing pages.
*   **Smart Entity Extraction:** Identifies contacts, locations, times, and messages using NER (Named Entity Recognition).
*   **Contextual Awareness:** Resolves vague terms like "Home," "Office," or "Rahul" based on user history and saved preferences.
*   **Privacy-First Architecture:** Local PII (Personally Identifiable Information) scrubbing before cloud processing.
*   **Fallback Intelligence:** If an app isn't installed, the system suggests web alternatives or Play Store links.

---

## 🧠 Architecture

The system follows a decoupled architecture separating the UI, the Reasoning Engine, and the Execution Layer.

### 1. Frontend (Android Client)
The entry point. It captures user input via a minimalist floating search bar or voice overlay. It is responsible for final action execution using Android's `Intent` system.

### 2. Backend (Orchestration Layer)
A FastAPI-based service that acts as the brain.
*   **Intent Classifier:** Routes the request to specific app domains (Transport, Messaging, Media).
*   **Entity Extractor:** Pulls parameters (e.g., `destination="JFK Airport"`).

### 3. AI Layer
*   **LLM (OpenAI/Claude/Llama-3):** Uses high-precision prompting to map strings to an internal schema.
*   **Vector Database (ChromaDB/Pinecone):** Maps semantic locations (e.g., "The coffee shop we went to yesterday") to GPS coordinates or specific addresses.

### 4. Action Engine
*   **Deep Link Generator:** Constructs URI schemes (e.g., `uber://`, `whatsapp://`).
*   **Android Intent Bridge:** Handles `ACTION_VIEW` and `ACTION_SEND` intents.

---

## 🛠️ Tech Stack

| Component | Technology |
| :--- | :--- |
| **Mobile App** | Kotlin (Jetpack Compose), Hilt (DI), Coroutines |
| **Backend** | FastAPI (Python), Pydantic |
| **AI/ML** | LangChain, OpenAI GPT-4o / Local Mistral 7B |
| **Database** | PostgreSQL (User Data), Redis (Caching) |
| **Vector Search** | ChromaDB (Contextual Mapping) |
| **Deployment** | Docker, AWS Lambda (Serverless Backend) |

---

## 🔄 Example Flow

**Scenario:** User types *"Book cab to office"*

1.  **Input:** User enters text in the Android Launcher.
2.  **Process:** Client sends payload + `UserContext` (Location, Time) to Backend.
3.  **AI Parsing:** LLM processes the request:
    ```json
    {
      "intent": "TRANSPORT_BOOK",
      "entities": { "destination": "office" },
      "confidence": 0.98,
      "suggested_app": "uber"
    }
    ```
4.  **Context Resolution:** The system looks up "office" in the user’s `SavedLocations`.
    *   *Result:* `40.7128° N, 74.0060° W`
5.  **Action Generation:** Backend returns a Deep Link:
    *   `uber://?action=setPickup&dropoff[latitude]=40.7128&dropoff[longitude]=-74.0060`
6.  **Execution:** Android Client receives the link and triggers `startActivity(intent)`.

---

## 📱 Android Implementation Details

*   **Intent System:** Utilizing `Intent.ACTION_VIEW` and specific package data for deep-linking.
*   **Accessibility Service (Optional):** For apps without public APIs or deep links, an optional Accessibility Service can be enabled to automate "click and type" sequences.
*   **App Indexing:** Local scanning of installed apps to ensure the Launcher only suggests available tools.

---

## 🔐 Privacy Considerations

*   **Local Scrubbing:** Contact names and addresses are tokenized locally before being sent to the LLM.
*   **Ephemeral Backend:** No command history is stored on the server unless explicitly opted-in for "AI Memory."
*   **Permission Transparency:** Only requests `QUERY_ALL_PACKAGES` and `LOCATION` permissions when necessary.

---

## 🚀 Getting Started

### Prerequisites
*   Android Studio Hedgehog+
*   Python 3.10+
*   OpenAI API Key (or local Ollama instance)

### Installation Steps

1.  **Clone the Repository**
    ```bash
    git clone https://github.com/your-org/ai-intent-launcher.git
    ```

2.  **Backend Setup**
    ```bash
    cd backend
    pip install -r requirements.txt
    cp .env.example .env  # Add your API keys
    uvicorn main:app --reload
    ```

3.  **Mobile Setup**
    *   Open `/frontend` in Android Studio.
    *   Sync Gradle.
    *   Update `Constants.kt` with your local IP for the backend API.
    *   Build and Run on an emulator or physical device.

---

## 📦 Folder Structure

```text
├── frontend/             # Kotlin Android Project
│   ├── app/src/main/java # UI & Intent Logic
│   └── domain/           # Models and Use Cases
├── backend/              # FastAPI Server
│   ├── api/              # Endpoints (v1)
│   ├── core/             # AI Logic & LangChain
│   └── db/               # PostgreSQL & Vector Store
├── ai/                   # Fine-tuning scripts & Prompt templates
└── docs/                 # API Specs & Architecture Diagrams
```

---

## 💡 Bonus Reference

### Sample Intent Extraction Prompt
```text
SYSTEM: You are a mobile intent parser. Convert user input into a JSON action.
USER: "Tell Sarah I'm running 5 mins late on Telegram"
OUTPUT:
{
  "action": "SEND_MESSAGE",
  "app": "telegram",
  "params": {
    "recipient": "Sarah",
    "content": "I'm running 5 mins late"
  }
}
```

### Deep Link Cheat-Sheet
*   **Spotify:** `spotify:track:<id>` or `spotify:search:<query>`
*   **Uber:** `uber://?action=setPickup&pickup=my_location&dropoff[nickname]=office`
*   **WhatsApp:** `whatsapp://send?phone=<number>&text=<message>`

---

## 🔮 Future Enhancements

*   **Voice-First Mode:** Integration with Whisper for high-accuracy voice commands.
*   **Multi-Step Workflows:** "If my meeting ends late, text my wife and book an Uber."
*   **On-Device LLM:** Moving inference to the device using MediaPipe or MLC LLM to reduce latency and enhance privacy.
*   **Predictive Actions:** Suggesting actions based on time of day (e.g., showing the "Gym Music" command at 6 PM).

---