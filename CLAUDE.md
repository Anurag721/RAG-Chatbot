# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Start the application:**
```bash
./run.sh
```
or manually:
```bash
cd backend
uv run uvicorn app:app --reload --port 8000
```

**Install dependencies:**
```bash
uv sync
```

**Environment setup:**
Create `.env` file with:
```
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

## Architecture Overview

This is a RAG (Retrieval-Augmented Generation) chatbot system for course materials with a tool-calling architecture where Claude decides when to search content.

### Core Architecture

**Request Flow:**
1. Frontend (vanilla JS) → FastAPI endpoint (`/api/query`)
2. RAG System orchestrates the query processing
3. AI Generator calls Claude with tool definitions
4. If Claude decides to search, Search Tools execute vector search
5. Vector Store (ChromaDB) returns relevant course content
6. Claude generates response based on search results

**Key Components:**

- **`backend/app.py`** - FastAPI server with CORS and static file serving
- **`backend/rag_system.py`** - Main orchestrator that coordinates all components
- **`backend/ai_generator.py`** - Handles Claude API calls and tool execution
- **`backend/search_tools.py`** - Tool-calling interface for semantic search
- **`backend/vector_store.py`** - ChromaDB integration for embeddings storage
- **`backend/document_processor.py`** - Structured document parsing and chunking
- **`backend/session_manager.py`** - Conversation history management

### Document Processing Pipeline

Documents are expected in this format:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 1: Introduction
[content]

Lesson 2: Advanced Topics
[content]
```

The system:
1. Extracts course metadata from first 3 lines
2. Identifies lessons using regex `Lesson \d+: title`
3. Chunks content using sentence-based splitting (800 chars, 100 overlap)
4. Adds contextual metadata: `"Course [title] Lesson [num] content: [text]"`
5. Stores in ChromaDB with metadata for filtering

### Configuration

All settings centralized in `backend/config.py`:
- Claude model: `claude-sonnet-4-20250514`
- Embedding model: `all-MiniLM-L6-v2`
- Chunk size: 800 characters with 100 overlap
- ChromaDB path: `./chroma_db`

### Tool-Calling System

Claude has access to `search_course_content` tool that can:
- Search by query text
- Filter by course name (partial matches work)
- Filter by lesson number
- Return formatted results with source tracking

The system uses "auto" tool choice - Claude decides when to search based on the query context.