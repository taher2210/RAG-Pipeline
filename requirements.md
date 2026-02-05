# Requirements Document
## Open-Source Multilingual NEP Knowledge Assistant

---

## 1. Problem Statement

Government school teachers across India face significant challenges in accessing and understanding National Education Policy (NEP) guidelines due to:

- Complex policy documents spanning hundreds of pages
- Limited availability of NEP content in regional languages
- Difficulty in finding specific information relevant to their teaching context
- Time constraints preventing thorough document review
- Language barriers for non-English speaking teachers

This creates a critical gap between policy formulation and grassroots implementation, hindering effective NEP adoption in government schools.

---

## 2. Objectives

- Provide instant access to NEP guidelines through natural language queries
- Support queries and responses in 10+ Indian regional languages
- Ensure all responses are grounded in official NEP documents with source citations
- Enable low-bandwidth access for teachers in rural areas
- Build a fully open-source, transparent, and auditable system
- Create a scalable solution deployable at state or national level

---

## 3. Target Users

### Primary Users
- Government school teachers (primary and secondary)
- Teacher trainers and educators
- School administrators and principals

### Secondary Users
- Education department officials
- Policy implementation teams
- Educational researchers

### User Characteristics
- Varying levels of digital literacy
- Preference for regional language interaction
- Limited access to high-speed internet
- Use of low-end mobile devices or basic computers

---

## 4. Functional Requirements

### 4.1 Document Management

**FR-1.1**: System shall support upload of NEP PDF documents to Amazon S3  
**FR-1.2**: System shall automatically extract text from uploaded PDFs  
**FR-1.3**: System shall chunk documents into semantically meaningful sections (512-1024 tokens)  
**FR-1.4**: System shall preserve document metadata (source, page number, section title)  
**FR-1.5**: System shall support versioning of policy documents  
**FR-1.6**: System shall allow administrators to add, update, or remove documents

### 4.2 Query Processing

**FR-2.1**: System shall accept text queries in Hindi, English, Bengali, Tamil, Telugu, Marathi, Gujarati, Kannada, Malayalam, Punjabi, and Odia  
**FR-2.2**: System shall automatically detect the language of the query  
**FR-2.3**: System shall generate embeddings for user queries  
**FR-2.4**: System shall retrieve top 5-10 most relevant document chunks using semantic search  
**FR-2.5**: System shall handle queries up to 500 characters in length

### 4.3 Response Generation

**FR-3.1**: System shall generate responses strictly based on retrieved document context  
**FR-3.2**: System shall respond in the same language as the user query  
**FR-3.3**: System shall include source citations (document name, page number) for all responses  
**FR-3.4**: System shall indicate when sufficient information is not available in the documents  
**FR-3.5**: System shall limit responses to 200-300 words for readability  
**FR-3.6**: System shall prevent hallucination by grounding all answers in retrieved content

### 4.4 API Endpoints

**FR-4.1**: System shall provide `POST /query` endpoint for submitting queries  
**FR-4.2**: System shall provide `GET /health` endpoint for health checks  
**FR-4.3**: System shall provide `POST /feedback` endpoint for user feedback  
**FR-4.4**: System shall provide `GET /documents` endpoint to list available policy documents  
**FR-4.5**: System shall return responses in JSON format

### 4.5 User Feedback

**FR-5.1**: System shall allow users to rate response quality (1-5 stars)  
**FR-5.2**: System shall collect optional text feedback from users  
**FR-5.3**: System shall log feedback for system improvement

---

## 5. Non-Functional Requirements

### 5.1 Performance

**NFR-1.1**: System shall respond to queries within 5 seconds (95th percentile)  
**NFR-1.2**: System shall support at least 100 concurrent users  
**NFR-1.3**: Embedding generation shall process at least 1000 chunks per minute  
**NFR-1.4**: Vector search shall complete within 100ms

### 5.2 Scalability

**NFR-2.1**: System shall scale horizontally to handle increased load  
**NFR-2.2**: System shall support addition of new languages without architectural changes  
**NFR-2.3**: System shall handle document corpus of up to 10,000 pages  
**NFR-2.4**: System shall support incremental index updates without full rebuild

### 5.3 Reliability

**NFR-3.1**: System shall maintain 99% uptime  
**NFR-3.2**: System shall implement automatic retry for failed operations  
**NFR-3.3**: System shall backup FAISS index to S3 every 24 hours  
**NFR-3.4**: System shall log all errors for debugging and monitoring

### 5.4 Security

**NFR-4.1**: System shall authenticate API requests using JWT tokens or API keys  
**NFR-4.2**: System shall use HTTPS/TLS for all communications  
**NFR-4.3**: System shall implement rate limiting (100 requests per user per hour)  
**NFR-4.4**: System shall not store personally identifiable information (PII)  
**NFR-4.5**: System shall anonymize query logs

### 5.5 Maintainability

**NFR-5.1**: System shall use modular architecture for easy component replacement  
**NFR-5.2**: System shall include comprehensive API documentation  
**NFR-5.3**: System shall use Docker containers for consistent deployment  
**NFR-5.4**: System shall include automated tests with >80% code coverage

### 5.6 Usability

**NFR-6.1**: API responses shall include clear error messages in English  
**NFR-6.2**: System shall provide example queries for each supported language  
**NFR-6.3**: System shall return responses in plain, simple language

### 5.7 Compatibility

**NFR-7.1**: System shall support Python 3.10+  
**NFR-7.2**: System shall be deployable on AWS EC2 instances  
**NFR-7.3**: System shall work with standard REST API clients

---

## 6. Accessibility and Inclusion Requirements

### 6.1 Language Support

**AIR-1.1**: System shall support 10+ Indian regional languages  
**AIR-1.2**: System shall handle code-mixed queries (e.g., Hinglish)  
**AIR-1.3**: System shall support Unicode text input and output

### 6.2 Low-Bandwidth Optimization

**AIR-2.1**: API responses shall be under 50 KB in size  
**AIR-2.2**: System shall work on 2G/3G networks  
**AIR-2.3**: System shall not require client-side JavaScript for basic functionality

### 6.3 Device Compatibility

**AIR-3.1**: System shall be accessible from low-end smartphones  
**AIR-3.2**: System shall support text-only interfaces  
**AIR-3.3**: System shall be compatible with screen readers (future web interface)

### 6.4 Digital Literacy

**AIR-4.1**: System shall provide simple, intuitive query interface  
**AIR-4.2**: System shall include example queries to guide users  
**AIR-4.3**: System shall provide clear instructions in regional languages

---

## 7. Constraints and Assumptions

### 7.1 Technical Constraints

**C-1.1**: System must use only open-source models and libraries  
**C-1.2**: System must not depend on proprietary LLM APIs  
**C-1.3**: System must be deployable on AWS infrastructure  
**C-1.4**: System must fit within budget constraints of government institutions

### 7.2 Data Constraints

**C-2.1**: Only official NEP documents shall be used as knowledge source  
**C-2.2**: Documents must be in PDF format  
**C-2.3**: Document updates require manual upload by administrators

### 7.3 Assumptions

**A-1.1**: NEP documents are available in digital PDF format  
**A-1.2**: Users have basic literacy in at least one supported language  
**A-1.3**: Users have access to internet-connected devices  
**A-1.4**: AWS services are available and reliable  
**A-1.5**: Open-source LLMs provide sufficient quality for educational content

---

## 8. Success Metrics

### 8.1 Adoption Metrics

**SM-1.1**: Number of active users per month  
**SM-1.2**: Number of queries processed per day  
**SM-1.3**: Geographic distribution of users  
**SM-1.4**: Language distribution of queries

### 8.2 Quality Metrics

**SM-2.1**: Average user rating (target: >4.0/5.0)  
**SM-2.2**: Percentage of queries with relevant responses (target: >85%)  
**SM-2.3**: Percentage of responses with valid source citations (target: 100%)  
**SM-2.4**: User retention rate (target: >60% monthly)

### 8.3 Performance Metrics

**SM-3.1**: Average query response time (target: <3 seconds)  
**SM-3.2**: System uptime (target: >99%)  
**SM-3.3**: API error rate (target: <1%)

### 8.4 Impact Metrics

**SM-4.1**: Reduction in time spent searching for NEP information  
**SM-4.2**: Increase in teacher awareness of NEP guidelines  
**SM-4.3**: Number of education departments adopting the system  
**SM-4.4**: Community contributions to the open-source project

---

## 9. Out of Scope

The following features are explicitly out of scope for the initial release:

- Voice-based query interface
- Mobile native applications
- Multi-turn conversational dialogue
- Personalized recommendations
- Integration with existing education portals
- Offline/edge deployment
- Real-time document updates via webhooks
- User authentication and profile management
- Analytics dashboard for administrators

These features may be considered for future releases based on user feedback and resource availability.

---

## 10. Dependencies

### 10.1 External Dependencies

- AWS S3 for document storage
- AWS EC2 for compute resources
- Open-source LLM models (Llama, Mistral, Gemma)
- Sentence-transformers library
- FAISS library
- FastAPI framework

### 10.2 Data Dependencies

- Official NEP PDF documents from government sources
- Multilingual training data for embedding models

### 10.3 Infrastructure Dependencies

- Stable internet connectivity
- AWS account with appropriate permissions
- Domain name for API hosting (optional)

---

## 11. Compliance and Legal

**CL-1.1**: System shall comply with Indian data protection regulations  
**CL-1.2**: System shall use only openly licensed models and libraries  
**CL-1.3**: System shall provide proper attribution for all third-party components  
**CL-1.4**: System shall not violate copyright of NEP documents (fair use for educational purposes)  
**CL-1.5**: System shall include appropriate disclaimers about AI-generated content

---

## 12. Approval and Sign-off

This requirements document requires approval from:

- Project Sponsor
- Technical Lead
- Education Domain Expert
- Security Officer

**Version**: 1.0  
**Last Updated**: 2026-02-05  
**Status**: Draft
