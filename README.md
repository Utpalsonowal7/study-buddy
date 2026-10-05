# Study Buddy
## Your study materials, ready to talk back

Live demo: [https://studybuddy.utpx.in/](https://studybuddy.utpx.in/)

Study Buddy is a full stack study assistant that turns a learner’s own notes and course documents into interactive, source cited study sessions. Upload class material, ask a question in plain language, get an answer grounded in your documents, and open the citations to see the supporting passages.

The complete product combines the web application and API described here. The frontend and backend live in separate repositories, but together they deliver one learning experience.

## The product experience

A learner signs in, adds study material, and asks questions in a chat. Study Buddy retrieves relevant passages from that learner’s documents, streams the answer into the conversation, and attaches citations so the learner can check the source.

**Upload → Understand → Ask → Verify → Continue**

1. **Add course material.** Upload a PDF, text file, or Markdown note to a personal document library.
2. **Let Study Buddy prepare it.** The backend extracts text, splits it into overlapping passages, creates embeddings, and indexes those passages for search.
3. **Ask in your own words.** Start a conversation using the full library or narrow the question to selected documents.
4. **Read while it answers.** The answer streams into the chat as text is generated.
5. **Check the evidence.** Open numbered citations to read source excerpts and page or chunk details.
6. **Pick up later.** Conversations and their source snapshots are saved for later study.

This is designed to help learners explore their own material. It is not a generic chatbot with unrelated sample answers: the retrieval flow searches the user’s indexed documents and presents source context with responses.

## What learners can do

- Register and sign in with email verification codes.
- Start Google or GitHub sign in.
- Upload, browse, inspect, download, and delete PDF, TXT, and Markdown documents.
- Track upload progress while documents are indexed.
- Ask questions across the library or select up to 20 documents for a focused answer.
- Watch responses stream into the chat, with citations shown alongside the answer.
- Reopen or delete saved conversations.
- Use the workspace on desktop or mobile, with light and dark themes.

## How the full product works

~~~mermaid
flowchart LR
    learner[Learner] --> frontend[Study Buddy web app]
    frontend -->|Email verification or OAuth| auth[FastAPI authentication]
    auth --> redis[(Redis: OTP and rate limits)]
    auth --> postgres[(PostgreSQL)]
    learner -->|Uploads a document| frontend
    frontend -->|Authenticated multipart upload| api[FastAPI API]
    api --> extract[Extract and split document text]
    extract --> embeddings[Gemini document embeddings]
    embeddings --> vectors[(PostgreSQL and pgvector)]
    api --> files[Private Cloudinary file storage]
    learner -->|Asks a question| frontend
    frontend -->|Question and optional document selection| api
    api -->|Find relevant passages| vectors
    api -->|Question and retrieved context| gemini[Gemini answer generation]
    gemini -->|Streaming answer and citations| frontend
    api --> conversations[(Saved conversations and source excerpts)]
    conversations --> frontend
~~~

### The frontend

The React and TypeScript application provides the landing page, email OTP and social sign in flows, protected workspace, document library, dashboard, chat history, citations, and account settings. It handles responsive layouts and user facing loading, empty, validation, and API error states.

In chat, the frontend reads the backend’s server sent event stream. It shows answer text as it arrives, attaches source metadata, handles cancellation, and keeps the returned conversation ID for follow up messages. It also refreshes expired cookie sessions, caches frequently used lists briefly in memory, and invalidates that cache after uploads, deletes, and new answers.

### The backend

The FastAPI service manages authentication, document processing, retrieval, answer generation, and saved history. It validates uploads, extracts and chunks text, generates Gemini embeddings, stores vectors in PostgreSQL with pgvector, and keeps original files in private Cloudinary storage.

For a question, it searches the authenticated learner’s ready documents, ranks relevant passages, and supplies the selected context and recent conversation history to Gemini. The API returns numbered citations and persists source excerpts with the conversation. Redis supports OTP state and rate limiting. Document and conversation routes check ownership before returning user data.

## Project highlights

- **Answers with evidence:** Responses connect to retrieved document passages and show citations the learner can inspect.
- **Live responses:** Gemini text is streamed through FastAPI as SSE and rendered incrementally in the React chat.
- **Saved study context:** Conversations, messages, and citation snapshots are persisted for future sessions.
- **Private document flow:** User files live in authenticated Cloudinary storage; downloads use temporary links.
- **Account boundaries:** Cookie based sessions and backend ownership checks keep document and conversation access tied to the authenticated user.
- **Failure handling:** The frontend communicates API failures and offers recovery actions. The backend handles incomplete generation and attempts cleanup when document indexing fails.
- **Two repository integration:** The browser and API use documented authentication, upload, chat, and history contracts under the versioned /api/v1 base.

## Technology behind the experience

| Product area | Technologies |
|---|---|
| Web application | React 19, TypeScript, Vite, React Router, Tailwind CSS 4 |
| API and validation | Python 3.14+, FastAPI, Pydantic |
| Application database | PostgreSQL, SQLAlchemy async, asyncpg |
| Retrieval | pgvector, Gemini embeddings, cosine similarity |
| Answer generation | Gemini through the Google Gen AI SDK |
| Sessions and rate limits | HttpOnly cookies, JWT, Redis |
| Document storage | Private Cloudinary assets |
| Email and social sign in | Brevo, Google OAuth, GitHub OAuth |

## The two repositories

This project overview is meant to give visitors to the frontend repository a clear picture of the complete product before they choose to inspect the API implementation.

| Repository | What is there |
|---|---|
| [Study Buddy Frontend](https://github.com/Utpalsonowal7/Study-Buddy-Full-Stack-RAG-Chatbot-with-Production-Grade-Authentication-Frontend) | The web application and frontend API integration. Use this as the primary project link on a résumé. |
| [Study Buddy Backend](https://github.com/Utpalsonowal7/Study-Buddy-Full-Stack-RAG-Chatbot-with-Production-Grade-Authentication) | The FastAPI API and document RAG implementation. |

For the complete backend setup, endpoint contracts, operational notes, and implementation details, see the [backend README](https://github.com/Utpalsonowal7/Study-Buddy-Full-Stack-RAG-Chatbot-with-Production-Grade-Authentication#readme) and [RAG API guide](https://github.com/Utpalsonowal7/Study-Buddy-Full-Stack-RAG-Chatbot-with-Production-Grade-Authentication/blob/main/RAG_API.md). The frontend’s [API integration notes](https://github.com/Utpalsonowal7/Study-Buddy-Full-Stack-RAG-Chatbot-with-Production-Grade-Authentication-Frontend/blob/main/src/data/API_ROUTES.md) show how the browser talks to those routes.

## Try the project locally

The full experience needs both repositories running. Start with the backend’s [local setup instructions](https://github.com/Utpalsonowal7/Study-Buddy-Full-Stack-RAG-Chatbot-with-Production-Grade-Authentication#run-locally). The backend requires PostgreSQL with pgvector, Redis, Gemini credentials, and Cloudinary credentials.

Then run the frontend:

~~~bash
git clone https://github.com/Utpalsonowal7/Study-Buddy-Full-Stack-RAG-Chatbot-with-Production-Grade-Authentication-Frontend.git
cd Study-Buddy-Full-Stack-RAG-Chatbot-with-Production-Grade-Authentication-Frontend
npm install
~~~

Set its API base in a local .env file:

~~~dotenv
VITE_BACKEND_URL=http://localhost:8000/api/v1
~~~

Allow the frontend origin in backend CORS settings with credentials enabled, then run:

~~~bash
npm run dev
~~~

Vite prints the address for the web app, usually http://localhost:5173. Keep credentials in local environment configuration; do not put backend secrets in frontend code or commit them.

## Technical details

The frontend includes lint and production build commands:

~~~bash
npm run lint
npm run build
~~~

The backend repository documents its test command, environment settings, deployment notes, and API behavior. The project implements its core study workflow; production uptime, measured answer quality, and throughput benchmarks depend on deployed services and have not been claimed here.

