# RAG Pipeline for Test Engineers - Overview

This guide explains the core functionalities of a Retrieval-Augmented Generation (RAG) pipeline, designed to understand and apply Generative AI in their work.

---

## 🎯 What is a RAG Pipeline?

A **RAG (Retrieval-Augmented Generation)** pipeline combines information retrieval with AI generation. Think of it as a smart search system that:
1. **Finds relevant information** from your test cases and user stories
2. **Uses AI to understand context and meaning** 
3. **Provides intelligent answers and insights**

---

## 📋 Functionalities Overview

### 1. Convert to JSON 📄
**Purpose:** Transforms raw test data (e.g., Excel files) into structured JSON format, making it ready for AI processing.

**Key Files:**
- `client/src/components/data/ConvertToJson.js` - Frontend UI for file upload
- `server/index.js` (lines 270-370) - Backend API endpoint `/api/upload-excel`
- `src/scripts/data-conversion/excel-to-json.js` - Test case conversion logic
- `src/scripts/data-conversion/excel-to-userstories.js` - User story conversion logic

**How it Works:**
1. Upload Excel file through web interface
2. Select data type (Test Cases or User Stories)
3. Backend dynamically generates conversion script
4. Converts Excel columns to standardized JSON format
5. Saves processed data to `src/data/` directory

**Example Transformation:**
```
Excel: "TC_001 | Login | Test user login functionality"
JSON: {"id": "TC_001", "module": "Authentication", "title": "Test user login functionality"}
```

---

### 2. Embeddings & Store 🔍
**Purpose:** Converts structured JSON data into numerical vector "embeddings" that capture semantic meaning for advanced search.

**Key Files:**
- `client/src/components/data/EmbeddingsStore.js` - Frontend for embedding management
- `server/index.js` (lines 725-830) - Backend APIs `/api/create-embeddings` and `/api/create-embeddings-batch`
- `src/scripts/utilities/mistralEmbedding.js` - MistralAI embedding generation

**How it Works:**
1. Select JSON files from converted data
2. Generate embeddings using MistralAI model
3. Store vectors in MongoDB Atlas with metadata
4. Track progress with background job system
5. Monitor costs and token usage

**Technical Process:**
```
Text Data → MistralAI API → 1024-dim Vector → MongoDB Atlas Storage
```

---

### 3. Query Preprocessing 🔄
**Purpose:** Enhances user search queries to improve retrieval accuracy through intelligent expansion and normalization.

**Key Files:**
- `client/src/components/processing/QueryPreprocessing.js` - Frontend query interface
- `server/index.js` (lines 1215-1267) - Backend APIs `/api/search/preprocess` and `/api/search/analyze`
- `src/scripts/query-preprocessing/queryPreprocessor.js` - Core preprocessing logic

**How it Works:**
1. User enters search query
2. System extracts and preserves test case IDs
3. Normalizes text (lowercase, remove special chars)
4. Expands abbreviations (TC → Test Case)
5. Generates synonyms (WhatsApp → messaging, chat, communication)
6. Returns multiple query variations for comprehensive search

**Example:**
```
Input: "TC_123 WhatsApp sharing test"
Output: [
  "tc_123 whatsapp sharing test",
  "tc_123 test case_123 messaging sharing test",
  "tc_123 test case_123 chat sharing test"
]
```

---

### 4. Vector Search 🔎
**Purpose:** Finds semantically similar content using vector embeddings and cosine similarity - understands meaning, not just keywords.

**Key Files:**
- `client/src/components/search/QuerySearch.js` - Frontend search interface
- `server/index.js` (lines 1781-1919) - Backend API `/api/search`
- MongoDB Atlas Vector Search configuration

**How it Works:**
1. User enters natural language query
2. Query is converted to embedding vector
3. MongoDB Atlas finds similar vectors using cosine similarity
4. Results ranked by similarity score (0-100)
5. Metadata filters applied (module, priority, risk)

**Technical Flow:**
```
Query → Embedding → Vector Search → Similarity Score → Ranked Results
```

---

### 5. BM25 Search 🔤
**Purpose:** Traditional keyword-based search using BM25 algorithm for exact term matching and technical precision.

**Key Files:**
- `client/src/components/search/BM25Search.js` - Frontend BM25 interface
- `server/index.js` (lines 1922-2046) - Backend API `/api/search/bm25`
- MongoDB Atlas Search with BM25 index

**How it Works:**
1. Analyzes term frequency in documents
2. Calculates inverse document frequency (IDF)
3. Applies BM25 scoring algorithm
4. Supports fuzzy matching (1-character typos)
5. Field-specific weighting for importance



---

### 6. Hybrid Search 🔀
**Purpose:** Combines BM25 and Vector search strengths for comprehensive retrieval with configurable weighting.

**Key Files:**
- `client/src/components/search/HybridSearch.js` - Frontend with weight controls
- `server/index.js` (lines 2049-2326) - Backend API `/api/search/hybrid`
- Score normalization and fusion logic

**How it Works:**
1. Executes both BM25 and Vector searches simultaneously
2. Normalizes scores to 0-1 range for fair comparison
3. Applies configurable weights (e.g., 50% BM25, 50% Vector)
4. Merges and deduplicates results
5. Final ranking based on combined scores

**Weight Configurations:**
- **BM25: 70%, Vector: 30%** - Better for technical terms
- **BM25: 30%, Vector: 70%** - Better for concepts  
- **BM25: 50%, Vector: 50%** - Balanced approach

---

### 7. Score Fusion (Reranking) 🤖
**Purpose:** Uses AI to intelligently rerank search results for enhanced relevance and accuracy.

**Key Files:**
- `client/src/components/search/RerankingSearch.js` - Frontend with before/after comparison
- `server/index.js` (lines 2329-1778) - Backend API `/api/search/rerank`
- `src/scripts/utilities/groqClient.js` - Groq AI reranking service

**How it Works:**
1. Retrieves broad candidate set (50+ results)
2. Sends candidates and query to Groq AI
3. AI analyzes semantic relevance to original query
4. Provides new rankings with reasoning explanations
5. Shows before/after comparison for transparency

**AI Analysis Criteria:**
- Functional relevance to query
- Technical term similarity
- Business context alignment
- Test scope appropriateness

---

### 8. Summarize & Dedup 📝
**Purpose:** AI-powered content analysis including duplicate detection and intelligent summarization of search results.

**Key Files:**
- `client/src/components/processing/SummarizationDedup.js` - Frontend workflow interface
- `server/index.js` (lines 1272-1332) - Backend API `/api/search/deduplicate`
- `server/index.js` (lines 1333-1400) - Backend API `/api/search/summarize`

**How it Works:**

**Deduplication:**
1. Calculates text similarity between results
2. Groups duplicates above threshold (0.85 default)
3. Preserves unique content, removes redundancy
4. Shows reduction statistics

**Summarization:**
1. Offers 4 summary types:
   - **Concise**: 2-3 paragraphs, key highlights
   - **Detailed**: Comprehensive analysis with gaps
   - **Technical**: Implementation focus
   - **Executive**: Business impact and risk
2. Uses Groq AI for intelligent content analysis
3. Provides actionable insights and recommendations

---

### 9. Prompt & Schema Management ⚙️
**Purpose:** Configure AI interactions and manage data structures for consistent system behavior.

**Key Files:**
- `client/src/components/processing/PromptSchemaManager.js` (671)- Frontend management interface
- `server/index.js` (lines 1401-1600) - Backend APIs for prompts and schemas
- Configuration files in `src/config/`

**How it Works:**

**Prompt Management:**
1. Create and customize prompt templates
2. Variable replacement system (`{query}`, `{focus_area}`)
3. Test templates with sample data
4. Save different prompt categories

**Schema Management:**
1. Define JSON schemas for data validation
2. Real-time schema validation
3. Test data against schema rules
4. Export documentation (JSON, YAML, Markdown)

**Example Prompt Template:**
```
"Find test cases that match: {query}. Focus on {focus_area}. Consider {context}."
```

---

## 🚀 Getting Started

### Prerequisites
- Node.js 16+
- MongoDB Atlas account
- API keys: MistralAI, Groq

### Quick Setup
```bash
# Install dependencies
npm install
cd client && npm install

# Configure environment
cp .env.example .env
# Add your API keys to .env

# Start development
npm run dev
```

### Environment Variables
```env
MONGODB_URI=mongodb+srv://...
MISTRAL_API_KEY=your_mistral_key
GROQ_API_KEY=your_groq_key
```

---

## 🎓 Learning Path for Test Engineers

### Phase 1: Foundation (Weeks 1-2)
1. **Convert to JSON** - Data preprocessing fundamentals
2. **Embeddings & Store** - Vector representation concepts
3. **Vector Search** - Semantic retrieval basics

### Phase 2: Advanced Search (Weeks 3-4)
4. **Query Preprocessing** - Query optimization techniques
5. **BM25 Search** - Traditional information retrieval
6. **Hybrid Search** - Combining multiple approaches

### Phase 3: AI Integration (Weeks 5-6)
7. **Score Fusion** - AI-powered result enhancement
8. **Summarize & Dedup** - Content analysis workflows
9. **Prompt & Schema** - System configuration

---

## 💡 Key Takeaways for Test Engineers

### Core Concepts:
1. **Vector Embeddings** - Text converted to numerical representations
2. **Semantic Search** - Finding content by meaning, not just keywords
3. **Hybrid Approaches** - Combining techniques for better results
4. **AI Integration** - Enhancing traditional testing workflows
5. **Data Processing** - Essential for AI applications

### Technical Skills Gained:
- Modern web development (React, Node.js)
- AI API integration (MistralAI, Groq)
- Vector databases and search algorithms
- Data processing and validation
- System architecture patterns

### Practical Applications:
- **Test Case Discovery** - Find relevant tests intelligently
- **User Story Analysis** - Understand semantic relationships
- **Quality Assurance** - Automated duplicate detection
- **Documentation** - AI-powered summarization
- **Schema Management** - Structured data validation

---



### Code Structure:
- **Frontend**: `client/src/components/` - React components
- **Backend**: `server/index.js` - Express API endpoints
- **Utilities**: `src/scripts/` - Processing logic
- **Configuration**: `src/config/` - System settings

