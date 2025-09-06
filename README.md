# AI Chat with YouTube (RAG)

This project is a full-stack web application that allows you to "chat" with any YouTube video. It uses a Retrieval-Augmented Generation (RAG) architecture to provide answers based on the video's transcript.

The frontend is built with React, TypeScript, and Vite, styled with Tailwind CSS. The backend is a Node.js/Express server that leverages LangChain.js, Google's Gemini models for embeddings and generation, and Supabase for the vector database.

## Features

-   **Submit YouTube URL**: Start a conversation by providing a YouTube video URL.
-   **Transcript Processing**: The backend automatically fetches the video transcript, splits it into manageable chunks, and stores it as embeddings.
-   **Interactive Chat**: Ask questions about the video content in a clean, chat-like interface.
-   **Context-Aware Responses**: The AI uses the conversation history and retrieved video context to provide relevant answers.
-   **Streaming**: AI responses are streamed back to the client for a real-time experience.

## Tech Stack

-   **Frontend**: React, TypeScript, Vite, Tailwind CSS
-   **Backend**: Node.js, Express.js
-   **AI**: LangChain.js, Google Gemini (`gemini-2.0-flash`, `embedding-001`)
-   **Database**: Supabase (PostgreSQL for metadata, pgvector for embeddings)

## How It Works

1.  **Document Ingestion**: When a user submits a YouTube URL, the frontend calls the `/store-document` endpoint.
2.  The backend server uses `YoutubeLoader` from LangChain to fetch the video's transcript and metadata.
3.  The transcript is split into smaller chunks using `RecursiveCharacterTextSplitter`.
4.  Google's `embedding-001` model generates vector embeddings for each text chunk.
5.  These embeddings, along with their corresponding text and metadata, are stored in a Supabase PostgreSQL table with the `pgvector` extension.

6.  **Querying**: When a user asks a question, the frontend calls the `/query-document` endpoint.
7.  A `historyAwareRetriever` first reformulates the user's question to be a standalone query based on the chat history.
8.  The reformulated query is used to perform a similarity search against the vector store to retrieve the most relevant document chunks.
9.  The original question, chat history, and the retrieved context are passed to the `gemini-2.0-flash` model.
10. The model generates an answer, which is streamed back to the frontend and displayed to the user.

## Project Setup

### Prerequisites

-   Node.js and npm
-   A Supabase account
-   A Google AI API Key

### 1. Supabase Setup

1.  Create a new project on [Supabase](https://supabase.com/).
2.  Go to the "Database" section and run the SQL script from [`server/db/match_documents.sql`](server/db/match_documents.sql) in the SQL Editor to create the `match_documents` function.
3.  You will also need to create the following tables. You can use the SQL Editor for this:
    ```sql
    -- For storing video documents
    CREATE TABLE documents (
        id UUID PRIMARY KEY,
        created_at TIMESTAMPTZ DEFAULT NOW()
    );

    -- For storing embedded chunks of video transcripts
    CREATE TABLE embedded_documents (
        id UUID PRIMARY KEY,
        content TEXT,
        metadata JSONB,
        embedding VECTOR(768), -- Make sure the vector size matches your embedding model
        document_id UUID REFERENCES documents(id)
    );

    -- For storing conversations
    CREATE TABLE conversations (
        id UUID PRIMARY KEY,
        created_at TIMESTAMPTZ DEFAULT NOW()
    );

    -- For linking conversations to documents
    CREATE TABLE conversation_documents (
        conversation_id UUID REFERENCES conversations(id),
        document_id UUID REFERENCES documents(id),
        PRIMARY KEY (conversation_id, document_id)
    );

    -- For storing chat messages
    CREATE TABLE conversation_messages (
        id BIGSERIAL PRIMARY KEY,
        conversation_id UUID REFERENCES conversations(id),
        role TEXT, -- 'user' or 'assistant'
        content TEXT,
        created_at TIMESTAMPTZ DEFAULT NOW()
    );
    ```

### 2. Backend Setup

1.  Navigate to the `server` directory:
    ```sh
    cd server
    ```
2.  Install dependencies:
    ```sh
    npm install
    ```
3.  Create a `.env` file by copying the contents of [`server/.env`](server/.env) and fill in your credentials:
    ```
    SUPABASE_URL="YOUR_SUPABASE_PROJECT_URL"
    SUPABASE_ANON_KEY="YOUR_SUPABASE_ANON_KEY"
    GEMINI_API_KEY="YOUR_GOOGLE_AI_API_KEY"
    ```
4.  Start the backend server:
    ```sh
    npm start
    ```
    The server will be running on `http://localhost:8000`.

### 3. Frontend Setup

1.  Navigate to the root project directory.
2.  Install dependencies:
    ```sh
    npm install
    ```
3.  Create a `.env.local` file in the root directory and add your Supabase credentials. These are exposed to the client.
    ```
    VITE_SUPABASE_URL="YOUR_SUPABASE_PROJECT_URL"
    VITE_SUPABASE_ANON_KEY="YOUR_SUPABASE_ANON_KEY"
    ```
4.  Start the frontend development server:
    ```sh
    npm run dev
    ```
    The application will be available at `http://localhost:5173`.

## Available Scripts

### Root Directory

-   `npm run dev`: Starts the Vite development server for the frontend.
-   `npm run build`: Builds the frontend for production.
-   `npm run lint`: Lints the frontend code.
-   `npm run preview`: Previews the production build locally.

### `server/` Directory

-   `npm start`: Starts the Node.js backend server with `nodemon`, which automatically restarts on file changes.
