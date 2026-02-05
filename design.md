# System Design Document
## Open-Source Multilingual NEP Knowledge Assistant

---

## 1. System Overview

The Open-Source Multilingual NEP Knowledge Assistant is a Retrieval-Augmented Generation (RAG) system designed to help government school teachers easily access and understand National Education Policy (NEP) guidelines. The system enables teachers to query official NEP-related documents in their local languages and receive accurate, grounded responses derived strictly from authoritative PDF sources.

The platform is fully open-source and deployed on Amazon Web Services (AWS), ensuring scalability, affordability, and ease of adoption by public institutions.

---

## 2. Design Goals

- Improve accessibility of NEP guidelines for government school teachers
- Support multilingual and regional language queries
- Ensure responses are grounded in official policy documents
- Use only open-source models and libraries
- Enable low-bandwidth and low-compute usability
- Allow easy deployment and extensibility by education departments

---

## 3. High-Level Architecture

The system follows a modular Retrieval-Augmented Generation architecture:

- **Document Storage Layer**: Amazon S3
- **Processing & Embedding Layer**: Amazon EC2
- **Vector Index Layer**: FAISS
- **Backend API Layer**: FastAPI
- **Language Generation Layer**: Open-source Large Language Models

---

## 4. Component Architecture

### 4.1 Document Storage (Amazon S3)

- Stores official NEP PDFs, circulars, and guideline documents
- Acts as the single source of truth
- Supports versioned updates of policy documents
- Organized bucket structure: `nep-documents/{year}/{category}/{filename}.pdf`

---

### 4.2 Document Processing Layer (Amazon EC2)

**Responsibilities:**
- Download PDFs from S3
- Extract text using PDF parsing libraries (PyMuPDF, pdfplumber)
- Clean and normalize extracted text
- Chunk documents into semantically meaningful sections (512-1024 tokens)
- Preserve document metadata (source, page number, section)

**Processing Pipeline:**
1. PDF ingestion from S3
2. Text extraction with layout preservation
3. Noise removal and normalization
4. Semantic chunking with overlap
5. Metadata tagging

This processing is performed asynchronously to allow batch ingestion of large document sets.

---

### 4.3 Embedding Generation Layer

**Model Selection:**
- Uses multilingual sentence-transformer models (e.g., `sentence-transformers/paraphrase-multilingual-mpnet-base-v2`)
- Generates dense vector representations (768-dimensional) for each document chunk
- Supports Indian regional languages: Hindi, Bengali, Tamil, Telugu, Marathi, Gujarati, Kannada, Malayalam, Punjabi, Odia

**Embedding Process:**
- Batch processing for efficiency
- GPU acceleration on EC2 instances (g4dn.xlarge or similar)
- Embeddings stored alongside chunk metadata
- Ensures semantic similarity across languages

---

### 4.4 Vector Indexing (FAISS)

**Index Configuration:**
- FAISS IndexFlatIP or IndexIVFFlat for semantic search
- Stores embeddings locally on EC2 with periodic S3 backups
- Enables fast nearest-neighbor semantic search (< 100ms)
- Supports approximate nearest neighbor (ANN) for scalability

**Index Management:**
- Incremental updates when new documents are added
- Version control for index snapshots
- Fully open-source and offline-capable

---

### 4.5 Query Handling API (FastAPI)

**Endpoints:**
- `POST /query` - Submit teacher queries
- `GET /health` - System health check
- `POST /feedback` - Collect user feedback
- `GET /documents` - List available policy documents

**Query Processing Flow:**
1. Receive query in local language
2. Detect language (optional)
3. Generate query embedding
4. Retrieve top-k relevant chunks from FAISS (k=5-10)
5. Pass context to LLM for answer generation
6. Return response with source references

**Features:**
- Lightweight and suitable for low-bandwidth environments
- Request validation and rate limiting
- Logging for analytics and improvement
- Can be integrated with web or mobile frontends

---

### 4.6 Response Generation Layer (Open-Source LLMs)

**Model Options:**
- Llama 3.1 8B Instruct
- Mistral 7B Instruct
- Gemma 2 9B Instruct
- Quantized versions (4-bit/8-bit) for resource efficiency

**Generation Strategy:**
- Receives retrieved document context
- Generates concise, grounded answers (max 200-300 words)
- Responds in the same language as the user query
- Prevents hallucination by restricting generation to retrieved content
- Includes source citations (document name, page number)

**Prompt Template:**
```
Context: {retrieved_chunks}
Question: {user_query}
Instructions: Answer based only on the provided context. If the answer is not in the context, say "I don't have enough information." Respond in {language}.
```

---

## 5. Data Flow

1. NEP PDF documents are uploaded to Amazon S3
2. EC2 instances extract and preprocess text
3. Text is chunked and embedded using multilingual models
4. Embeddings are indexed in FAISS
5. Teacher submits a query in a local language via API
6. Query embedding is generated
7. FAISS retrieves the most relevant document chunks (top-k)
8. Open-source LLM generates a grounded response
9. Answer is returned in the teacher's language with source references
10. Feedback is optionally collected for system improvement

---

## 6. Security and Privacy

**Access Control:**
- IAM roles restrict access to AWS resources
- S3 bucket policies enforce read-only access for processing
- API authentication via JWT tokens or API keys

**Data Privacy:**
- No personal teacher data is stored
- Queries are anonymized in logs
- No data sharing with third parties

**Infrastructure Security:**
- EC2 security groups restrict inbound traffic
- HTTPS/TLS for all API communications
- Regular security patches and updates

**Compliance:**
- Logs contain no sensitive information
- GDPR-compliant data handling (if applicable)

---

## 7. Scalability and Deployment

**Horizontal Scaling:**
- Multiple EC2 instances behind a load balancer
- Stateless API design for easy replication
- FAISS index replication across instances

**Vertical Scaling:**
- GPU instances for faster embedding generation
- Larger instance types for increased throughput

**Deployment Strategy:**
- Docker containers for consistent environments
- Infrastructure as Code (Terraform/CloudFormation)
- CI/CD pipeline for automated deployments

**Cost Optimization:**
- Spot instances for batch processing
- Reserved instances for production workloads
- S3 lifecycle policies for archival

**Modularity:**
- Adding new languages requires only model updates
- Adding new policy documents is automated via S3 triggers
- Containerization-ready for future orchestration (Kubernetes)
- Suitable for state-level or national deployment

---

## 8. Accessibility and Inclusion Considerations

**Language Support:**
- Multilingual support for 10+ Indian regional languages
- Low-resource language handling
- Automatic language detection

**Device Compatibility:**
- Text-based interface compatible with low-end devices
- Responsive web design for mobile access
- Minimal bandwidth requirements (< 100 KB per query)

**User Experience:**
- Simple, intuitive query interface
- Clear source citations for transparency
- Designed for future voice-based extensions

**Adoption:**
- Open-source licensing enables adoption by public education systems
- Documentation in multiple languages
- Training materials for teachers

---

## 9. Open-Source Strategy

**Technology Stack:**
- All components use open-source libraries and models
- No dependency on proprietary LLM APIs
- Transparent, auditable codebase

**Licensing:**
- MIT or Apache 2.0 license for code
- Clear attribution for models and datasets

**Community Engagement:**
- Public GitHub repository
- Contribution guidelines
- Issue tracking and feature requests
- Community-driven enhancements possible

**Benefits:**
- Transparency and trust
- Cost-effective for government institutions
- Customizable for specific regional needs
- No vendor lock-in

---

## 10. Future Enhancements

**Voice Interface:**
- Voice-first interface for teachers with low literacy
- Speech-to-text and text-to-speech in regional languages
- Integration with WhatsApp or IVR systems

**Offline Deployment:**
- Edge deployments for rural schools without internet
- Lightweight models for resource-constrained environments
- Periodic sync with central repository

**Mobile Application:**
- Native Android app for wider reach
- Offline caching of frequently accessed policies
- Push notifications for policy updates

**Intelligent Features:**
- Automatic updates when NEP documents change
- Personalized recommendations based on teacher role
- Multi-turn conversational interface
- Summarization of long policy documents

**Analytics Dashboard:**
- Usage analytics for education departments
- Popular queries and knowledge gaps
- System performance monitoring
- Teacher feedback aggregation

**Integration:**
- Integration with existing education portals
- Single sign-on (SSO) with government systems
- API for third-party applications

---

## 11. Technology Stack Summary

| Component | Technology |
|-----------|-----------|
| Document Storage | Amazon S3 |
| Compute | Amazon EC2 (CPU/GPU) |
| PDF Processing | PyMuPDF, pdfplumber |
| Embeddings | sentence-transformers |
| Vector Index | FAISS |
| Backend API | FastAPI |
| LLM | Llama 3.1 / Mistral 7B |
| Language | Python 3.10+ |
| Containerization | Docker |
| Infrastructure | Terraform / CloudFormation |

---

## 12. Conclusion

The Open-Source Multilingual NEP Knowledge Assistant provides a practical, inclusive, and scalable solution to bridge the information access gap faced by government school teachers. By combining open-source AI with cloud infrastructure, the system ensures trustworthy access to policy information in local languages, directly supporting effective NEP implementation at the grassroots level.
