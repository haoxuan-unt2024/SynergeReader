1 Purpose
Build a browser-based POC demonstrating: upload document → highlight text → ask question → get LLM answer using Gemini, enhanced with question analysis, vector similarity search from the full document, and incorporation of relevant history.

2 Scope

- Single user; no auth.
- Files: PDF, DOCX, TXT; max 20 MB.
- One LLM call per question (with analysis, vector-based retrieval, history incorporation, no streaming).
- Local SQLite storage for history; vector DB for document embeddings; deploy on single machine via Docker.

3 Functional Requirements


| ID | Requirement |
| :-- | :-- |
| F-1 | Front-end accepts file via drag-and-drop or picker, parses it to plain UTF-8 text locally (using pdf.js, mammoth.js). |
| F-2 | After parsing, front-end sends text to back-end via POST `/upload` for processing. |
| F-3 | Back-end chunks the full text, generates embeddings using an embedding model, and stores chunks with embeddings in a vector DB for similarity search. |
| F-4 | Front-end renders parsed text in scrollable reader with mouse selection. |
| F-5 | On selection end, show modal with: read-only selected text, question textarea. |
| F-6 | On submit, front-end POSTs `/ask` with JSON: `{selected_text, question}`. |
| F-7 | Back-end first analyzes the question combined with selected_text (e.g., via a preliminary LLM call or rule-based parsing) to identify key intents or topics. |
| F-8 | Based on analysis, back-end embeds the analyzed query and performs similarity search in vector DB to find top relevant text chunks from the full document. |
| F-9 | Back-end also searches SQLite history for relevant past Q\&A records (e.g., by similarity or keyword matching). |
| F-10 | Back-end builds prompt including analyzed query, similar chunks, relevant history, selected_text, and question: `Analysis:\n{query_analysis}\n\nRelevant History:\n{relevant_qa}\n\nContext:\n{similar_chunks}\n\nSelected:\n{selected_text}\n\nQuestion:\n{question}\n\nAnswer as an expert.` (Handle token limits by truncating if needed.) |
| F-11 | Back-end calls Gemini-Pro with the prompt, returns full answer as `{answer}`. |
| F-12 | Front-end shows answer in modal below question. |
| F-13 | Back-end adds record to SQLite `chat_history`: id, timestamp, selected_text, question, answer. |
| F-14 | History page lists last 20 Q\&A pairs (including selected_text) in reverse chronological order. |

4 Non-Functional Requirements


| Category | Constraint |
| :-- | :-- |
| Performance | End-to-end latency ≤ 8s for 600-word highlight, including analysis, search, and history retrieval. |
| Reliability | Restart keeps stored history and vector DB data. |
| Security | APIs on localhost only; no external access (configure Docker network accordingly). |
| Maintainability | Docstrings in code; one-command Docker Compose startup. |

5 System Architecture (MVP)

```
═════════════════════════════════════════════════════════════
| Browser SPA | ──── HTTP/JSON ───► | FastAPI Service |
│ (single process)                  ▼ | SQLite DB       |
│                                   ▼ | Vector DB       |
═════════════════════════════════════════════════════════════
```

6 Data Model (SQLite)
Table: `chat_history`


| Column | Type | Description |
| :-- | :-- | :-- |
| id | INTEGER PK | Auto-increment |
| ts | TEXT | ISO-8601 timestamp |
| selected_text | TEXT | Highlighted text |
| question | TEXT | User question |
| answer | TEXT | LLM response |

(Note: Vector DB stores document chunks with embeddings separately.)

7 External Interfaces


| Method | Endpoint | Payload → Response |
| :-- | :-- | :-- |
| POST | `/upload` | `{text}` → Confirmation |
| POST | `/ask` | `{selected_text, question}` → `{answer}` |
| GET | `/history` | None → `[{ts, selected_text, question, answer}]` |

8 Development Split (Two Developers, Balanced)
Akhil – Front-end \& Parsing

- React SPA: upload, local parsing, text reader, highlight, modal.
- API calls: `/upload`, `/ask`, `/history`.
- Vitest unit tests.

Tazree and Sam – Back-end \& LLM

- FastAPI routes: `/upload`, `/ask`, `/history`.
- Document chunking, embedding generation, vector DB integration for similarity search.
- Question analysis logic, history retrieval from SQLite.
- Prompt build, Gemini-Pro calls (Google GenAI SDK).
- SQLite schema, CRUD.
- Docker Compose file.
- Logging, error handling.

Both: Collaborate on OpenAPI YAML; mutual code reviews for balance.

9 Milestones


| Week | Deliverables |
| :-- | :-- |
| 1 | File upload \& local text parsing (A); FastAPI skeleton + `/upload` with chunking/embedding (B). |
| 2-3 | Highlight modal \& `/ask` (A); Question analysis, vector search, history retrieval, LLM call + SQLite insert (B). |
| 4-5 | History page (A); Docker packaging \& README (B); joint demo. |
