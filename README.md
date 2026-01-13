Retrieval-Augmented Generation (RAG) Application

==============================================
OVERVIEW
==============================================
This is an end-to-end Retrieval-Augmented Generation (RAG) application that allows users to upload documents, ask natural language questions, and receive context-grounded answers with evaluation and source attribution.

This project demonstrates the core fundamentals of building, evaluating, and deploying a simple yet robust RAG pipeline.

==============================================
FEATURES
==============================================
- Upload and index PDF / DOCX documents
- Semantic document retrieval using embeddings
- Context-aware answer generation using an LLM
- Ambiguity and contradiction awareness
- Answer evaluation (faithfulness & hallucination detection)
- Source attribution for transparency
- Chat-style web UI
- Persistent vector storage using ChromaDB
- Cloud-deployment ready (AWS)

==============================================
TECH STACK
==============================================
Backend API        : FastAPI
Frontend           : HTML, CSS, JavaScript
Embeddings Model   : sentence-transformers/paraphrase-MiniLM-L3-v2
Vector Database    : ChromaDB (persistent)
LLM                : Google Gemini
Evaluation         : LLM-as-a-Judge (faithfulness & hallucination)
Deployment         : AWS

==============================================
ARCHITECTURE
==============================================
A detailed application architecture diagram is provided under the Architecture_Diagram folder.

==============================================
PROJECT STRUCTURE
==============================================
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

==============================================
SETUP INSTRUCTIONS (LOCAL)
==============================================
1. Clone the repository
   git clone <repository-url>
   cd RAG

2. Create and activate a virtual environment
   python -m venv .venv
   source .venv/bin/activate        (Mac/Linux)
   .venv\Scripts\activate         (Windows)

3. Install dependencies
   pip install -r requirements.txt

4. Create a .env file with the following:
   GOOGLE_API_KEY=your_gemini_api_key
   EMBEDDING_MODEL_NAME=sentence-transformers/paraphrase-MiniLM-L3-v2
   PERSIST_DIRECTORY=./data/chroma_db
   DEVICE=cpu

==============================================
RUNNING THE APPLICATION
==============================================
uvicorn app.api:app --reload

Backend API  : http://localhost:8000
Web UI       : http://localhost:8000/ui
Swagger Docs : http://localhost:8000/docs

==============================================
HOW TO USE
==============================================
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

==============================================
DATA PREPARATION
==============================================
- Parent chunk size  : 1500 characters
- Child chunk size   : 300 characters
- Overlap used to preserve context continuity

==============================================
RETRIEVAL
==============================================
- Semantic vector search using sentence embeddings
- ParentDocumentRetriever ensures document-level coherence
- Retrieves only the most relevant chunks for each query
- Retrieved sources displayed below each answer to improve transparency and trust.
- Conversational memory is used to handle follow-up questions more effectively by retaining prior context.

==============================================
GENERATION
==============================================
- Retrieved chunks are injected into the LLM prompt
- LLM is instructed to answer strictly from context
- Explicitly handles contradictions and unknowns

==============================================
EVALUATION
==============================================
Two evaluation criteria:
1. Faithfulness – Is the answer grounded in retrieved context?
2. Hallucination – Does the answer introduce unsupported information?
3. Grounded Truth Assessment – Is the answer aligned with the retrieved context?
Evaluation is performed using Gemini as an LLM judge.

==============================================
TEST ARTIFACTS
==============================================
The following test artifacts are included in the Results folder for reference:

- Test-Results.xlsx  
  Contains evaluation results and metrics captured during testing.

- Test-Screenshots.docs  
  Includes screenshots of application flows and test executions.

==============================================
DEPLOYMENT
==============================================
The application is deployed on:
- AWS EC2 (m7i-flex.large)

The deployed application is accessible through https://ragapp.duckdns.org/ui

ChromaDB persists data on disk, making it suitable for lightweight cloud deployments.

==============================================
LIMITATIONS
==============================================
- Free-tier LLM API rate limits
- Vector DB grows unless explicitly cleared
- Single-user PoC design

==============================================
FINDINGS & TRADE-OFFS FOR DESIGN DECISIONS
==============================================
While building this Retrieval-Augmented Generation (RAG) application, several design decisions were made based on practical constraints, experimentation, and the scope of the challenge. Below are the key findings and trade-offs observed during development.

1. Document Chunking Strategy

Large documents were split into smaller chunks before embedding to ensure efficient retrieval. A parent–child chunking strategy was used so that retrieval happens at a fine-grained level while still preserving broader context.

Trade-off:
Smaller chunks improve retrieval precision but may lose global context. Using parent–child chunks adds complexity but results in more grounded and coherent answers.

2. Embedding Model Selection

A lightweight sentence-transformer model (paraphrase-MiniLM-L3-v2) was chosen for embeddings.

Trade-off:
This model offers fast inference on CPU and works well within free-tier cloud limits. However, it may not capture deeper semantic nuances as effectively as larger embedding models.

3. Vector Store Choice (ChromaDB)

ChromaDB was selected for local persistence and ease of setup.

Trade-off:
ChromaDB is ideal for prototyping and small-scale deployments but is not designed for large-scale, multi-tenant production systems. This was an acceptable trade-off given the challenge scope.

4. Multi-Document Retrieval Behavior

The system allows multiple documents to be indexed together. When documents contain conflicting information, the model may surface multiple viewpoints.

Finding:
This behavior mirrors real-world knowledge retrieval, where conflicting sources often exist.

Trade-off:
Instead of forcing a single “correct” answer, the system exposes ambiguity through source attribution. This improves transparency but may require additional logic in future versions to resolve contradictions.

5. Prompt Design & Hallucination Control

The LLM prompt explicitly instructs the model to answer only using retrieved context and to return a fallback response when the context is insufficient.

Trade-off:
This significantly reduces hallucinations but can make the model conservative, occasionally returning “I don’t know” even when partial information is available.

6. Evaluation Using LLM-based Metrics

Faithfulness and hallucination detection are performed using an LLM-based evaluator.

Trade-off:
This provides human-like judgment and richer explanations but increases response latency and API usage.

7. Conversational Memory

Conversation buffer memory was introduced to support follow-up questions and a more natural chat experience.

Trade-off:
While it improves usability, long conversations can introduce context drift. For this proof-of-concept, global memory was acceptable, but production systems would require session-based memory isolation.

8. UI & Deployment Decisions

The UI separates answers, evaluation results, and sources to improve clarity. The backend is optimized to run on CPU-only infrastructure and free cloud tiers.

Trade-off:
The system prioritizes clarity, explainability, and cost efficiency over high throughput or large-scale concurrency.

==============================================
SCOPE FOR IMPROVEMENT
==============================================
- Introduce tracing and metrics using Langfuse to monitor retrieval quality, latency, and model behavior.
- Extend the system to an agentic RAG architecture using LangGraph to enable multi-step reasoning and tool-based workflows.
- Incorporate hybrid retrieval and re-ranking to further improve retrieval accuracy.
- Enhance evaluation with automated benchmarks and human feedback loops.
- Improve scalability by adopting a managed vector database for production-scale deployments.

==============================================
CONCLUSION
==============================================
This application demonstrates a complete RAG pipeline covering data preparation, retrieval, generation, evaluation, and deployment. The project emphasizes correctness, transparency, and practical design trade-offs, making it well-suited for interviews and real-world learning.
