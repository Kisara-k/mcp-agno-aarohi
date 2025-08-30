# Agentic RAG (Retrieval-Augmented Generation) Tutorial

## Overview

This tutorial demonstrates building **Agentic RAG systems** using both traditional LangChain approaches and the modern Agno framework. RAG combines retrieval of relevant documents with generative AI to provide contextually accurate responses, while agentic RAG adds intelligent reasoning and tool usage capabilities.

## Learning Objectives

By completing this tutorial, you will:

- Understand RAG architecture and components
- Learn traditional RAG implementation with LangChain
- Explore agentic RAG with Agno framework
- Master vector database setup and management
- Implement intelligent document retrieval and reasoning

## What is Agentic RAG?

**Traditional RAG**:

- Retrieves relevant documents
- Generates responses based on retrieved context
- Static, predetermined retrieval strategy

**Agentic RAG** (Enhanced):

- **Intelligent Retrieval**: Agents decide what to retrieve and when
- **Multi-step Reasoning**: Complex queries broken into sub-tasks
- **Tool Integration**: Access to external APIs and services
- **Dynamic Planning**: Adapts strategy based on query complexity

## Getting Started

### Prerequisites

```bash
# Core dependencies
pip install -U agno
pip install arxiv
pip install langchain langchain-community langchain-google-genai langchain-core langchain-text-splitters
pip install chromadb
pip install python-dotenv

# PostgreSQL dependencies (optional, for advanced setup)
pip install pgvector psycopg[binary]
```

### Environment Setup

1. **Create a `.env` file**:

   ```env
   OPENAI_API_KEY=your_openai_api_key_here
   GOOGLE_API_KEY=your_google_api_key_here
   ```

2. **Get API Keys**:
   - **OpenAI**: [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
   - **Google AI**: [ai.google.dev/gemini-api/docs/api-key](https://ai.google.dev/gemini-api/docs/api-key)

### Project Structure

```
2 Agno RAG/
├── RAG.ipynb                   # Traditional RAG with LangChain
├── AgenticRAG.ipynb            # Agentic RAG with Agno
├── laptops_info.txt            # Sample knowledge base
└── README.md                   # This tutorial
```

## How to Run

### Option 1: Traditional RAG (LangChain)

```bash
jupyter notebook RAG.ipynb
```

### Option 2: Agentic RAG (Agno)

```bash
jupyter notebook AgenticRAG.ipynb
```

## Implementation Details

### 1. Traditional RAG Pipeline (LangChain)

**Step 1: Document Loading**

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("laptops_info.txt")
documents = loader.load()
```

**Step 2: Text Chunking**

```python
from langchain_text_splitters import CharacterTextSplitter
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = text_splitter.split_documents(documents)
```

**Step 3: Vector Embeddings**

```python
from langchain_google_genai import GoogleGenerativeAIEmbeddings
embeddings = GoogleGenerativeAIEmbeddings(model="models/embedding-001")
```

**Step 4: Vector Store Creation**

```python
from langchain_community.vectorstores import Chroma
vector_store = Chroma.from_documents(chunks, embeddings)
```

**Step 5: Retrieval and Generation**

```python
retriever = vector_store.as_retriever(search_kwargs={"k": 3})
relevant_docs = retriever.get_relevant_documents("laptop recommendation")
```

### 2. Agentic RAG Pipeline (Agno)

**Step 1: Agent Creation**

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.knowledge.vector_db import VectorDB

agent = Agent(
    model=OpenAIChat(id="gpt-4"),
    knowledge_base=VectorDB(),
    description="Intelligent RAG assistant"
)
```

**Step 2: Knowledge Base Integration**

```python
# Agent automatically handles retrieval and reasoning
response = agent.run("Find the best laptop for machine learning under $2000")
```

## Database Setup Options

### Option 1: ChromaDB (Simple Setup)

- **In-memory vector database**
- **No installation required**
- **Perfect for development and testing**

### Option 2: PostgreSQL + pgvector (Production Setup)

#### Installation Steps:

1. **Download PostgreSQL**: [postgresql.org](https://www.postgresql.org)

2. **Verify Installation**:

   ```bash
   psql --version
   ```

3. **Create Database and User**:

   ```sql
   psql -U postgres
   CREATE USER ai WITH PASSWORD 'ai';
   CREATE DATABASE ai;
   GRANT ALL PRIVILEGES ON DATABASE ai TO ai;
   ```

4. **Install pgvector Extension**:

   - Download: [pgvector releases](https://github.com/andreiramani/pgvector_pgsql_windows/releases)
   - Extract `vector.v0.8.0-pg17.3.zip`
   - Copy files:
     - `vector.dll` → `C:\Program Files\PostgreSQL\17\lib`
     - `vector.control`, `vector--*.sql` → `C:\Program Files\PostgreSQL\17\share\extension`

5. **Restart PostgreSQL** (as Administrator):
   ```bash
   net stop postgresql-x64-17
   net start postgresql-x64-17
   ```

## Key Concepts

### 1. **Document Chunking Strategies**

**Fixed-size Chunking**:

```python
text_splitter = CharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separator="\n"
)
```

**Semantic Chunking**:

```python
# Split by sentences or paragraphs for better context
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separators=["\n\n", "\n", ".", "!", "?", ",", " ", ""]
)
```

### 2. **Embedding Models**

**Google Embeddings**:

```python
embeddings = GoogleGenerativeAIEmbeddings(model="models/embedding-001")
```

**OpenAI Embeddings**:

```python
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings(model="text-embedding-ada-002")
```

### 3. **Retrieval Strategies**

**Similarity Search**:

```python
# Find most similar documents
docs = vector_store.similarity_search("query", k=3)
```

**Maximum Marginal Relevance (MMR)**:

```python
# Balance relevance and diversity
docs = vector_store.max_marginal_relevance_search("query", k=3)
```

### 4. **Agentic Enhancements**

**Query Analysis**:

- Agent analyzes query complexity
- Determines optimal retrieval strategy
- Plans multi-step reasoning if needed

**Contextual Reasoning**:

- Synthesizes information from multiple sources
- Handles follow-up questions
- Maintains conversation context

## Comparison: Traditional vs. Agentic RAG

| Aspect               | Traditional RAG | Agentic RAG          |
| -------------------- | --------------- | -------------------- |
| **Retrieval**        | Fixed strategy  | Adaptive strategy    |
| **Reasoning**        | Single-step     | Multi-step planning  |
| **Tool Usage**       | Limited         | Extensive            |
| **Context Handling** | Static          | Dynamic              |
| **Query Processing** | Direct          | Analyzed and planned |
| **Response Quality** | Good            | Superior             |

## Performance Optimization

### 1. **Chunking Optimization**

```python
# Experiment with different chunk sizes
chunk_sizes = [500, 1000, 1500, 2000]
for size in chunk_sizes:
    test_chunking_performance(size)
```

### 2. **Retrieval Tuning**

```python
# Adjust number of retrieved documents
retriever = vector_store.as_retriever(
    search_type="mmr",
    search_kwargs={"k": 5, "fetch_k": 20}
)
```

### 3. **Embedding Selection**

- **Speed**: `text-embedding-ada-002`
- **Quality**: `text-embedding-3-large`
- **Balance**: `models/embedding-001` (Google)

## Practical Examples

### 1. **Laptop Recommendation System**

```python
query = "Recommend a laptop for machine learning with NVIDIA GPU under $2000"
response = agent.run(query)
```

### 2. **Research Assistant**

```python
query = "What are the latest developments in transformer architectures?"
response = agent.run(query)
```

### 3. **Document Q&A**

```python
query = "Summarize the key features of the laptops in the database"
response = agent.run(query)
```

## Advanced Features

### 1. **Multi-Modal RAG**

```python
# Support for text, images, and documents
from agno.knowledge.multi_modal import MultiModalDB
knowledge_base = MultiModalDB()
```

### 2. **Real-time Updates**

```python
# Dynamic knowledge base updates
agent.knowledge_base.add_documents(new_documents)
```

### 3. **Query Routing**

```python
# Route queries to different knowledge bases
router = QueryRouter({
    "technical": tech_kb,
    "business": business_kb,
    "general": general_kb
})
```

## Exercises

### Beginner

1. Build a simple RAG system with your own documents
2. Experiment with different chunk sizes
3. Compare different embedding models

### Intermediate

1. Implement query classification for better retrieval
2. Add conversation memory to maintain context
3. Create a web interface for your RAG system

### Advanced

1. Build a multi-document RAG system
2. Implement real-time knowledge base updates
3. Create an agentic RAG with tool integration

## Next Steps

After mastering RAG systems, explore:

1. **Agno Memory** (see `3 Agno Memory/`)
2. **Multi-Agent RAG Systems**
3. **Knowledge Graph Integration**
4. **Real-time RAG with streaming**

## Further Reading

- [RAG Paper](https://arxiv.org/abs/2005.11401) - Original RAG research
- [LangChain RAG Tutorial](https://python.langchain.com/docs/use_cases/question_answering/)
- [Vector Database Comparison](https://blog.langchain.dev/evaluating-rag-architectures-on-benchmark-tasks/)
- [Advanced RAG Techniques](https://blog.llamaindex.ai/advanced-rag-techniques-an-illustrated-overview-04d193d8fec6)

## Video Tutorial

**Hindi Tutorial**: https://youtu.be/PB9yf7E5YTw

## Troubleshooting

### Common Issues:

1. **ChromaDB Persistence**:

   ```python
   vector_store = Chroma(
       persist_directory="./chroma_db",
       embedding_function=embeddings
   )
   ```

2. **Memory Issues with Large Documents**:

   ```python
   # Use smaller chunk sizes or batch processing
   chunk_size = 500
   ```

3. **API Rate Limits**:
   ```python
   # Add delays between API calls
   import time
   time.sleep(1)
   ```

---

**Note**: Agentic RAG represents the evolution of information retrieval systems, combining the power of large language models with intelligent retrieval strategies to provide more accurate, contextual, and useful responses. This tutorial provides both traditional and cutting-edge approaches to building production-ready RAG systems.
