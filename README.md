# Retrieval-Augmented Generation (RAG) Application

## Overview
This is an end-to-end Retrieval-Augmented Generation (RAG) application that allows users to upload documents, ask natural language questions, and receive context-grounded answers with evaluation and source attribution.

This project demonstrates the core fundamentals of building, evaluating, and deploying a simple yet robust RAG pipeline.

## Features
- Upload and index PDF / DOCX documents
- Semantic document retrieval using embeddings
- Context-aware answer generation using an LLM
- Ambiguity and contradiction awareness
- Answer evaluation (faithfulness & hallucination detection)
- Source attribution for transparency
- Chat-style web UI
- Persistent vector storage using ChromaDB
- Cloud-deployment ready (AWS)

## Tech Stack
- **Backend API**: FastAPI
- **Frontend**: HTML, CSS, JavaScript
- **Embeddings Model**: sentence-transformers/paraphrase-MiniLM-L3-v2
- **Vector Database**: ChromaDB (persistent)
- **LLM**: Google Gemini
- **Evaluation**: LLM-as-a-Judge (faithfulness & hallucination)
- **Deployment**: AWS

## Architecture
A detailed application architecture diagram is provided under the Architecture_Diagram folder.

## Project Structure
```text
RAG/
 ├── app/
 │   ├── api.py              - FastAPI routes
 │   ├── retriver.py         - Retrieval & RAG logic
 │   ├── index.py            - Document indexing
 │   ├── pre_process.py      - Document preprocessing
 │   ├── evaluate.py         - Answer evaluation
 │   └── __init__.py
 ├── templates/
 │   └── index.html          - UI layout
 ├── static/
 │   ├── styles.css          - UI styling
 │   └── app.js              - Frontend logic
 ├── data/
 │   ├── chroma_db/          - Vector database
 │   └── parent_docs/        - Parent documents
 ├── uploaded_docs/          - Uploaded files
 ├── Architecture_Diagram/   - A detailed application architecture diagram
 ├── Test_Results/
 │   ├── Test-Results.xlsx   - Evaluation results and metrics
 │   └── Test-Screenshots.docs - Test execution and UI screenshots
 ├── requirements.txt
 └── README.md
```

---
### FastAPI Backend

FastAPI serves as the core backend API layer for the RAG application. It exposes RESTful APIs that handle document ingestion, indexing, retrieval, answer generation, evaluation, and UI rendering.

**Key responsibilities handled by FastAPI:**
- Provides REST endpoints for document upload, querying, and health checks
- Orchestrates the end-to-end RAG workflow (retrieval → generation → evaluation)
- Integrates semantic retrieval, LLM-based answer generation, and evaluation logic
- Serves the chat-based web UI using Jinja2 templates
- Hosts static frontend assets (CSS, JavaScript)
- Enforces structured request and response schemas using Pydantic models

**Exposed API Endpoints:**
- `GET /` – Health check endpoint
- `GET /ui` – Serves the chat-based web UI
- `POST /index` – Uploads and indexes PDF/DOCX documents into the vector store
- `POST /ask` – Executes the RAG pipeline:
 
The FastAPI application is served using Uvicorn.

## SETUP INSTRUCTIONS (LOCAL)

1. Clone the repository
```bash
git clone <repository-url>
cd RAG
```

2. Create and activate a virtual environment
```bash
python -m venv .venv
source .venv/bin/activate        # Mac/Linux
.venv\Scripts\activate         # Windows
```

3. Install dependencies
```bash
pip install -r requirements.txt
```

4. Create a `.env` file with the following:
```env
GOOGLE_API_KEY=your_gemini_api_key
EMBEDDING_MODEL_NAME=sentence-transformers/paraphrase-MiniLM-L3-v2
PERSIST_DIRECTORY=./data/chroma_db
DEVICE=cpu
```

---

## RUNNING THE APPLICATION
```bash
uvicorn app.api:app --reload
```

- Backend API  : http://localhost:8000  
- Web UI       : http://localhost:8000/ui  
- Swagger Docs : http://localhost:8000/docs  

---

## HOW TO USE

1. Upload a document
   - Choose a PDF or DOCX file
   - Click the Index button
   - The document is cleaned, chunked, embedded, and stored

2. Ask a question
   - Type your question in the chat input
   - Press Enter or click Send

3. View the response
   - Context-based answer
   - Evaluation results (faithfulness & hallucination)
   - Source attribution (file name, page, snippet)

If multiple documents provide conflicting information, the system highlights the ambiguity instead of guessing.

---

## DATA PREPARATION & CHUNKING STRATEGY

To ensure efficient retrieval while preserving semantic coherence, the application uses a **parent–child document chunking strategy** during indexing.

### Parent Chunks
- **Chunk size**: 1500 characters  
- **Overlap**: 100 characters  

Parent chunks represent larger semantic units of the document (such as sections or paragraphs). These chunks are stored in a persistent document store and are used to preserve broader contextual meaning during answer generation.

### Child Chunks
- **Chunk size**: 300 characters  
- **Overlap**: 50 characters  

Child chunks are smaller, fine-grained segments derived from parent chunks. These are embedded and stored in the vector database to enable precise semantic retrieval.

---

### Indexing & Vector Storage (ChromaDB)

Once documents are chunked, the indexing process converts each child chunk into a dense vector embedding using a sentence-transformer model. These embeddings are then stored in a persistent ChromaDB vector store.

A dedicated ChromaDB collection is created to hold the embedded child chunks. 

While child chunks are stored in the vector database for efficient semantic search, their corresponding parent documents are stored separately in a persistent document store. Stable unique identifiers are assigned to parent documents to maintain a reliable mapping between parent and child chunks.

During indexing:
- Parent documents are processed, assigned stable IDs, and stored in a local document store.
- Child chunks are embedded and added to a ChromaDB collection on disk.
- Vector data is persisted, ensuring that indexed content remains available across application restarts.

This design separates **semantic search** from **context reconstruction**, enabling accurate retrieval while preserving document-level meaning.

---

## RETRIEVAL

When a user asks a question, the system first focuses on finding the most relevant information from the indexed documents.

The query is converted into a semantic embedding and matched against a persistent vector database using similarity search. Retrieval operates on smaller, fine-grained chunks to ensure accuracy, but these chunks are mapped back to their original parent documents before being used. This prevents fragmented answers and helps preserve the original meaning and structure of the source content.

Only the most relevant parent documents are selected and passed forward. Source metadata such as file name and page number is also captured at this stage, enabling transparency later in the response.

This step ensures that the system works with relevant, high-quality context before attempting to generate an answer.

---

## AUGMENTATION

After retrieval, the selected document content is prepared and structured before being sent to the language model.

The retrieved documents are cleaned, formatted, and combined into a coherent context block. Conversational history from previous interactions is also injected, allowing the system to handle follow-up questions naturally. This augmented prompt provides the language model with both the necessary background knowledge and the conversational context needed to respond accurately.

By carefully controlling how much context is included and how it is structured, the system avoids overwhelming the model while still providing enough information to generate grounded responses.

This stage acts as the bridge between raw document retrieval and answer generation.

---

## GENERATION

Once the augmented context is ready, it is passed to the language model to generate the final response.

The model is explicitly instructed to answer strictly based on the provided document context. If multiple documents present conflicting information, the model is guided to clearly highlight the disagreement rather than choosing a single answer arbitrarily. When the context does not contain enough information to answer the question, the model responds cautiously instead of guessing.

This controlled generation approach helps minimize hallucinations and ensures that every answer remains faithful to the retrieved sources, clear in its reasoning, and transparent about uncertainty.

---

## EVALUATION

After an answer is generated, the system performs an explicit evaluation step to assess its quality and reliability.

Evaluation is handled using Gemini as an LLM-based judge. The original question, the generated answer, and the retrieved document context are passed to the evaluator, which is instructed to judge the answer strictly against the provided context.

The evaluation focuses on three aspects:
- **Faithfulness**: Measures how well the answer is grounded in the retrieved documents.
- **Hallucination**: Checks whether the answer introduces information that is not supported by the context.
- **Explanation**: Provides a brief justification for the evaluation outcome.

The evaluator returns a structured JSON response containing a numerical faithfulness score, a hallucination flag, and a short explanation. Additional safeguards are implemented to ensure a valid evaluation output is always returned, even if the model response is malformed.

This evaluation step adds an extra layer of transparency and confidence, making it easier to identify unreliable answers and improving trust in the system’s responses.

---

## TEST ARTIFACTS

The following test artifacts are included in the Results folder for reference:

- **Test-Results.xlsx**  
  Contains evaluation results and metrics captured during testing.

- **Test-Screenshots.docs**  
  Includes screenshots of application flows and test executions.

---

## DEPLOYMENT

The application is deployed on an Amazon EC2 Linux instance and is designed to run as a lightweight, self-contained service.

The deployed application is publicly accessible at:  
https://ragapp.duckdns.org/ui

Nginx is used as a reverse proxy in front of the FastAPI backend, handling HTTPS termination using self-signed CA certificates. Public access to the application is restricted to port 443, while the FastAPI service runs on an internal port, improving security by isolating the backend from direct exposure.

ChromaDB runs locally on the EC2 instance and uses persistent disk storage, ensuring that indexed vector data survives application restarts and system reboots. This allows documents to remain searchable without requiring re-indexing.

The language model itself is not hosted locally. Instead, the application securely communicates with the Gemini LLM via API calls, keeping infrastructure requirements minimal and avoiding the overhead of managing large model runtimes.

Configuration and secrets such as API keys and storage paths are managed through environment variables. Both the application and Nginx are configured to restart automatically on system reboot, ensuring high availability with minimal operational overhead.

Overall, this deployment approach prioritizes simplicity, security, and cost efficiency while remaining suitable for real-world usage and demonstrations.

---

## LIMITATIONS
- Free-tier LLM API rate limits
- Vector DB grows unless explicitly cleared
- Single-user PoC design

---

## FINDINGS & TRADE-OFFS FOR DESIGN DECISIONS

While building this Retrieval-Augmented Generation (RAG) application, several design decisions were made based on practical constraints, experimentation, and the scope of the challenge. Below are the key findings and trade-offs observed during development.

### 1. Document Chunking Strategy
Large documents were split into smaller chunks before embedding to ensure efficient retrieval. A parent–child chunking strategy was used so that retrieval happens at a fine-grained level while still preserving broader context.

**Trade-off:**  
Smaller chunks improve retrieval precision but may lose global context. Using parent–child chunks adds complexity but results in more grounded and coherent answers.

### 2. Embedding Model Selection
A lightweight sentence-transformer model (paraphrase-MiniLM-L3-v2) was chosen for embeddings.

**Trade-off:**  
This model offers fast inference on CPU and works well within free-tier cloud limits. However, it may not capture deeper semantic nuances as effectively as larger embedding models.

### 3. Vector Store Choice (ChromaDB)
ChromaDB was selected for local persistence and ease of setup.

**Trade-off:**  
ChromaDB is ideal for prototyping and small-scale deployments but is not designed for large-scale, multi-tenant production systems. This was an acceptable trade-off given the challenge scope.

### 4. Multi-Document Retrieval Behavior
The system allows multiple documents to be indexed together. When documents contain conflicting information, the model may surface multiple viewpoints.

**Finding:**  
This behavior mirrors real-world knowledge retrieval, where conflicting sources often exist.

**Trade-off:**  
Instead of forcing a single “correct” answer, the system exposes ambiguity through source attribution. This improves transparency but may require additional logic in future versions to resolve contradictions.

### 5. Prompt Design & Hallucination Control
The LLM prompt explicitly instructs the model to answer only using retrieved context and to return a fallback response when the context is insufficient.

**Trade-off:**  
This significantly reduces hallucinations but can make the model conservative, occasionally returning “I don’t know” even when partial information is available.

### 6. Evaluation Using LLM-based Metrics
Faithfulness and hallucination detection are performed using an LLM-based evaluator.

**Trade-off:**  
This provides human-like judgment and richer explanations but increases response latency and API usage.

### 7. Conversational Memory
Conversation buffer memory was introduced to support follow-up questions and a more natural chat experience.

**Trade-off:**  
While it improves usability, long conversations can introduce context drift. For this proof-of-concept, global memory was acceptable, but production systems would require session-based memory isolation.

### 8. UI & Deployment Decisions
The UI separates answers, evaluation results, and sources to improve clarity. The backend is optimized to run on CPU-only infrastructure and free cloud tiers.

**Trade-off:**  
The system prioritizes clarity, explainability, and cost efficiency over high throughput or large-scale concurrency.

---

## SCOPE FOR IMPROVEMENT
- Introduce tracing and metrics using Langfuse to monitor retrieval quality, latency, and model behavior.
- Extend the system to an agentic RAG architecture using LangGraph to enable multi-step reasoning and tool-based workflows.
- Incorporate hybrid retrieval and re-ranking to further improve retrieval accuracy.
- Enhance evaluation with automated benchmarks and human feedback loops.
- Improve scalability by adopting a managed vector database for production-scale deployments.

---

## CONCLUSION
This application demonstrates a complete RAG pipeline covering data preparation, retrieval, generation, evaluation, and deployment. The project emphasizes correctness, transparency, and practical design trade-offs, making it well-suited for real-world learning.
