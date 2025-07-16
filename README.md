1  Purpose
Deliver a browser-based proof-of-concept that demonstrates the core loop: *upload a document → highlight text → ask a question → receive an LLM answer*. The prototype sacrifices non-essential features (e.g., multi-user auth, RAG retrieval) to minimise development time while still producing a fully runnable system.

2  Scope

- One user at a time; no authentication.
- File types: PDF, DOCX, TXT; size limit 20 MB.
- Single LLM request per question (no retrieval, no streaming).
- Local SQLite persistence; single machine deployment (Docker).

3  Functional Requirements


| ID | Requirement |
| :-- | :-- |
| F-1 | The front-end shall accept a file via drag-and-drop or file picker and send it to the back-end. |
| F-2 | After upload, the system shall convert the file to plain UTF-8 text and return it to the front-end. |
| F-3 | The client shall render the text in a scrollable reader that supports mouse-based text selection. |
| F-4 | Upon mouse-up, a modal dialog shall appear containing: selected text (read-only), a textarea for the user question, and a model selector with two options: *GPT* (default) or *Gemini*. |
| F-5 | When the user submits, the client shall call `POST /ask` with JSON: `{text, question, model}`. |
| F-6 | The back-end shall concatenate *selected text* and *question* into a single prompt: |

`Prompt = “Context:\n{selected_text}\n\nQuestion:\n{question}\n\nAnswer as an expert.”`
F-7 | The back-end shall invoke the chosen LLM (GPT-3.5-turbo or Gemini-Pro) and return the full answer in one payload: `{answer}`.
F-8 | The client shall display the answer in the same modal below the user question.
F-9 | The back-end shall append one record to SQLite (`chat_history` table) containing: id, timestamp, model, selected_text, question, answer.
F-10 | A simple “History” page shall list the last 20 Q\&A pairs in reverse chronological order.

4  Non-Functional Requirements


| Category | Constraint |
| :-- | :-- |
| Performance | End-to-end latency ≤ 8 s for a 600-word highlight. |
| Reliability | Prototype restart shall not lose stored history. |
| Security | All API traffic local to localhost; no external exposure required. |
| Maintainability | Code documented via docstrings; self-contained Docker Compose file for one-command start-up. |

5  System Architecture (MVP)

```
═════════════════════════════════════════════════════════════
|  Browser SPA  | ──── HTTP/JSON ───► |  FastAPI Service  |
                               │        (single process)
                               ▼
                        |   SQLite DB   |
═════════════════════════════════════════════════════════════
```

6  Data Model (SQLite)

Table: `chat_history`


| Column | Type | Description |
| :-- | :-- | :-- |
| id | INTEGER PK | Auto-increment |
| ts | TEXT | ISO-8601 timestamp |
| model | TEXT | 'gpt' or 'gemini' |
| selected_text | TEXT | Raw highlight |
| question | TEXT | User question |
| answer | TEXT | LLM response |

7  External Interfaces


| Method | Endpoint | Payload → Response |
| :-- | :-- | :-- |
| POST | `/upload` | multipart file → `{text}` (plain body) |
| POST | `/ask` | `{text, question, model}` → `{answer}` |
| GET | `/history` | ∅ → `[{ts, model, question, answer}]` |

8  Development Split (Two Developers, Balanced Workload)

Akhil – *Front-end \& Parsing*

- React SPA: upload widget, text reader, highlight detection, modal UI.
- Client-side PDF/TXT/DOCX parsing via `pdf.js`, `mammoth.js`.
- API integration (`/upload`, `/ask`, `/history`).
- Unit tests with Vitest.

Tazree and Sam – *Back-end \& LLM Integration*

- FastAPI routes: `/upload`, `/ask`, `/history`.
- Prompt construction, LLM calls (OpenAI, Google GenAI SDK).
- SQLite schema migration and CRUD helper.
- Dockerfile + docker-compose (FastAPI + static file server).
- Basic logging and error handling.

Both developers collaborate on the interface contract (OpenAPI YAML) and perform mutual code reviews to keep tasks comparable in effort.

9  Milestones


| Week | Deliverables |
| :-- | :-- |
| 1 | File upload \& plain-text extraction (A); FastAPI skeleton + `/upload` (B). |
| 2,3 | Highlight modal \& `/ask` wiring (A); LLM call + SQLite insert (B). |
| 4,5 | History page polish (A); Docker packaging \& README (B); joint demo. |

