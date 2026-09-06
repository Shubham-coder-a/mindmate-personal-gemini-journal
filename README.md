# MindMate – Personal Gemini Journal

> A production-ready, full-stack personal AI journal and reflection sanctuary powered by **React**, **Node.js (Express)**, **Firebase Authentication**, **Google Cloud Firestore**, and **Gemini 3.8 Flash**.

---

## Features

- **Google Sign-In with Firebase Authentication**: Seamless authentication using Google Sign-In or guest access.
- **Complete UID Isolation in Cloud Firestore**: Every user's journal entries, conversations, and reflections are stored under `/users/{uid}/...` with strict Firestore security rules.
- **Multi-Turn Gemini AI Conversations**: Empathic, grounded reflection chat with Gemini that understands the context of your recent journal entries.
- **Mood & Reflection Summary**: Automated psychological synthesis analyzing recent entries for:
  - Detected overall mood & emotional score (1–10)
  - Key recurring themes
  - Positive highlights & inner strengths
  - Areas of concern & mindful guidance
  - Resonant takeaway narrative & forward-looking journaling prompt
- **Full Journal Management**: Create, edit, search, filter by mood/tag, and delete entries with real-time Firestore synchronization.
- **AI Prompt Sparks**: On-demand reflective questions tailored to your current mood to overcome writer's block.
- **Server-Side Security**: The `GEMINI_API_KEY` is strictly accessed server-side and never exposed to the client.
- **Google Cloud Run Ready**: Multi-stage Dockerfile and containerized architecture ready for deployment.

---

## Architecture Overview

```
Client (React + Vite + Tailwind CSS)
  │
  ├── Firebase Auth (Google Sign-In)
  ├── Cloud Firestore (users/{uid}/journalEntries, conversations, reflections)
  │
  └── Express Backend API (/api/...)
        │
        └── Google Gen AI SDK (@google/genai)
              └── Gemini 3.8 Flash (Server-Side with telemetry User-Agent)
```

---

## Data Schema (Firestore)

All documents are scoped strictly by the authenticated user's Firebase UID:

### 1. Journal Entries
- **Path**: `/users/{uid}/journalEntries/{entryId}`
- **Fields**:
  - `title`: string
  - `content`: string
  - `mood`: string (e.g. Grateful, Peaceful, Energized, Contemplative)
  - `moodScore`: number (1–10)
  - `tags`: string[]
  - `createdAt`: ISO 8601 string
  - `updatedAt`: ISO 8601 string

### 2. Conversations
- **Path**: `/users/{uid}/conversations/{convoId}`
- **Fields**:
  - `title`: string
  - `messages`: Array of `{ id, role: 'user' | 'model', content, timestamp }`
  - `relatedEntryId`: optional string
  - `createdAt`: ISO 8601 string
  - `updatedAt`: ISO 8601 string

### 3. Reflection Summaries
- **Path**: `/users/{uid}/reflections/{reflectionId}`
- **Fields**:
  - `detectedMood`: string
  - `moodScore`: number (1–10)
  - `moodColor`: string
  - `keyThemes`: string[]
  - `positiveHighlights`: string[]
  - `areasOfConcern`: string[]
  - `shortReflection`: string
  - `mindfulPrompt`: string
  - `entryCountAnalyzed`: number
  - `createdAt`: ISO 8601 string

---

## Firestore Security Rules

Stored in `firestore.rules`:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Only allow authenticated users to access their own UID-scoped data
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;

      match /journalEntries/{entryId} {
        allow read, write: if request.auth != null && request.auth.uid == userId;
      }

      match /conversations/{convoId} {
        allow read, write: if request.auth != null && request.auth.uid == userId;
      }

      match /reflections/{reflectionId} {
        allow read, write: if request.auth != null && request.auth.uid == userId;
      }

      match /{document=**} {
        allow read, write: if request.auth != null && request.auth.uid == userId;
      }
    }

    match /{document=**} {
      allow read, write: false;
    }
  }
}
```

---

## Environment Variables

Defined in `.env.example`:

| Variable | Description | Location |
|---|---|---|
| `GEMINI_API_KEY` | Google Gemini API key for server-side AI calls | Server-side only |
| `PORT` | Listening port for Express (default: `3000`) | Server-side only |
| `APP_URL` | Hosted URL of the application | Client & Server |
| `VITE_FIREBASE_*` | Optional Firebase overrides (defaults to `firebase-applet-config.json`) | Client-side |

---

## Local Development & Setup

1. **Install dependencies**:
   ```bash
   npm install
   ```

2. **Configure Environment**:
   Ensure `GEMINI_API_KEY` is provided in `.env` or the AI Studio Secrets panel.
   Firebase configuration is automatically loaded from `firebase-applet-config.json`.

3. **Start the development server**:
   ```bash
   npm run dev
   ```
   The application runs on `http://localhost:3000`.

4. **Run TypeScript verification**:
   ```bash
   npm run lint
   ```

5. **Build for production**:
   ```bash
   npm run build
   ```

---

## Google Cloud Run Deployment

You can build and deploy MindMate to Google Cloud Run with a single command using Google Cloud CLI:

### 1. Build & Deploy with gcloud
```bash
# Set your Google Cloud project
gcloud config set project YOUR_PROJECT_ID

# Deploy directly from source via Cloud Build & Cloud Run
gcloud run deploy mindmate \
  --source . \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars GEMINI_API_KEY="YOUR_GEMINI_API_KEY"
```

### 2. Or Build & Run with Docker Container
```bash
# Build container image
docker build -t gcr.io/YOUR_PROJECT_ID/mindmate:latest .

# Push to Google Container Registry / Artifact Registry
docker push gcr.io/YOUR_PROJECT_ID/mindmate:latest

# Deploy container image to Cloud Run
gcloud run deploy mindmate \
  --image gcr.io/YOUR_PROJECT_ID/mindmate:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars GEMINI_API_KEY="YOUR_GEMINI_API_KEY"
```
