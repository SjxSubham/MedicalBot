# MedicalBot System Architecture

## Overview
MedicalBot is a Retrieval-Augmented Generation (RAG) system that provides medical information by combining a vector database of medical knowledge with a large language model.

## System Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                               MEDICALBOT ARCHITECTURE                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐    ┌──────────────────────┐    ┌─────────────────────────────────┐
│   DATA PHASE    │    │   PROCESSING PHASE   │    │        INFERENCE PHASE          │
└─────────────────┘    └──────────────────────┘    └─────────────────────────────────┘

┌─────────────────┐    ┌──────────────────────┐    ┌─────────────────────────────────┐
│ Medical PDFs    │───▶│ create_memory_       │───▶│ Vector Database (FAISS)         │
│ (data/)         │    │ for_llm.py           │    │ (vectorstore/db_faiss)          │
│                 │    │                      │    │                                 │
│ • GALE Medical  │    │ 1. PyPDFLoader       │    │ • 500-char chunks               │
│   Encyclopedia  │    │ 2. Text Splitter     │    │ • HuggingFace embeddings        │
│ • Other medical │    │ 3. HF Embeddings     │    │ • Fast similarity search        │
│   documents     │    │ 4. FAISS storage     │    │                                 │
└─────────────────┘    └──────────────────────┘    └─────────────────────────────────┘
                                │                                   │
                                ▼                                   ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                            USER INTERACTION LAYER                                   │
└─────────────────────────────────────────────────────────────────────────────────────┘

                       ┌─────────────────────────────────┐
                       │         medibot.py              │
                       │      (Streamlit Web App)        │
                       └─────────────────────────────────┘
                                        │
                                        ▼
                       ┌─────────────────────────────────┐
                       │        User Query               │
                       │    "What is diabetes?"          │
                       └─────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              RAG PIPELINE                                           │
└─────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐    ┌──────────────────────┐    ┌─────────────────────────────────┐
│ Vector Search   │───▶│ Context Formation    │───▶│ LLM Generation                  │
│                 │    │                      │    │                                 │
│ • Query FAISS   │    │ • Custom prompt      │    │ • Mistral-7B-Instruct-v0.3     │
│ • Find top-3    │    │ • Add context        │    │ • HuggingFace Endpoint          │
│   similar docs  │    │ • Medical question   │    │ • Temperature: 0.5              │
│ • Retrieve      │    │ • Instructions       │    │ • Conversational task           │
│   medical text  │    │                      │    │                                 │
└─────────────────┘    └──────────────────────┘    └─────────────────────────────────┘
         ▲                                                           │
         │                                                           ▼
         └───────────────────────────────────┬───────────────────────────────────────┐
                                            ▼                                        │
                       ┌─────────────────────────────────┐                          │
                       │      Final Response             │◀─────────────────────────┘
                       │                                 │
                       │ • Medical answer                │
                       │ • Source documents              │
                       │ • Chat interface display        │
                       └─────────────────────────────────┘
```

## Component Details

### 1. Data Preparation (`create_memory_for_llm.py`)
```
PDF Documents → PyPDFLoader → RecursiveCharacterTextSplitter → HuggingFaceEmbeddings → FAISS
     │               │                    │                           │                  │
  Medical         Extract              Chunk into              Convert to           Store as
 Knowledge         Text               500 characters           Embeddings         Vector DB
```

### 2. LLM Integration (`connect_memory_with_llm.py`)
```
FAISS Database → RetrievalQA Chain → HuggingFace Endpoint → Response
      │                │                      │               │
  Load Vector       Create RAG           Mistral-7B        Generate
   Database          Pipeline            Language          Answer
                                         Model
```

### 3. Web Application (`medibot.py`)
```
Streamlit UI → User Input → Query Processing → Vector Search → LLM Response → Display
     │             │              │                │              │            │
 Chat Web      Medical          Parse &          Find         Generate      Show Answer
Interface      Question         Validate       Relevant        Medical       + Sources
                                Query          Context        Response
```

## Technology Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                    MEDICALBOT TECH STACK                        │
├─────────────────────────────────────────────────────────────────┤
│ Frontend Layer                                                  │
│ • Streamlit (Web UI, Chat Interface, Session Management)       │
├─────────────────────────────────────────────────────────────────┤
│ AI/ML Layer                                                     │
│ • LangChain (Document processing, RAG pipeline)                │
│ • HuggingFace (Mistral-7B LLM, Sentence Transformers)         │
│ • Transformers (Text embeddings, Model inference)              │
├─────────────────────────────────────────────────────────────────┤
│ Data Layer                                                      │
│ • FAISS (Vector database, Similarity search)                   │
│ • PyPDF (Document loading and parsing)                         │
│ • Medical Documents (GALE Encyclopedia, PDFs)                  │
├─────────────────────────────────────────────────────────────────┤
│ Infrastructure Layer                                            │
│ • Python 3.13 (Runtime environment)                           │
│ • Pipenv (Dependency management)                               │
│ • Environment Variables (API tokens, configurations)           │
└─────────────────────────────────────────────────────────────────┘
```

## Key Features

- **RAG Architecture**: Combines retrieval and generation for accurate medical responses
- **Vector Search**: Fast similarity matching using FAISS for relevant medical information
- **Source Attribution**: Provides references to original medical documents
- **Interactive Chat**: Real-time conversation interface with message history
- **Medical Knowledge Base**: Built on comprehensive medical encyclopedia
- **Configurable LLM**: Uses state-of-the-art Mistral-7B language model

## Setup Process

1. **Data Preparation**: Run `create_memory_for_llm.py` to process medical documents
2. **Testing**: Use `connect_memory_with_llm.py` for command-line testing
3. **Production**: Launch `medibot.py` for web-based medical consultations