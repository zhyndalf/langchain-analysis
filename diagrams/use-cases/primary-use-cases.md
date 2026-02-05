# Use Case Diagrams

This document contains use case diagrams for common LangChain application patterns.

## 1. Build a Simple Chatbot

```mermaid
graph TB
    subgraph Actors
        Dev[Application Developer]
        User[End User]
    end

    subgraph "LangChain System"
        UC1[Configure Chat Model]
        UC2[Create Prompt Template]
        UC3[Build Chat Chain]
        UC4[Handle User Messages]
        UC5[Stream Responses]
        UC6[Add System Instructions]
    end

    subgraph "External Systems"
        LLM[LLM Provider<br/>OpenAI, Anthropic]
    end

    Dev --> UC1
    Dev --> UC2
    Dev --> UC3
    Dev --> UC6
    User --> UC4
    User --> UC5

    UC1 -.-> LLM
    UC3 --> UC1
    UC3 --> UC2
    UC4 --> UC3
    UC5 --> UC3
    UC6 --> UC2

    style Dev fill:#e1f5ff
    style User fill:#fff4e1
    style LLM fill:#ffe1e1
```

**Primary Actor**: Application Developer, End User
**Goal**: Create an interactive chatbot with streaming responses
**Components Used**: ChatPromptTemplate, ChatModel, LCEL chains

---

## 2. Create a RAG Application

```mermaid
graph TB
    subgraph Actors
        Dev[Application Developer]
        User[End User]
    end

    subgraph "LangChain System"
        UC1[Load Documents]
        UC2[Split Documents]
        UC3[Generate Embeddings]
        UC4[Store in Vector DB]
        UC5[Create Retriever]
        UC6[Build RAG Chain]
        UC7[Query with Context]
        UC8[Format Retrieved Docs]
    end

    subgraph "External Systems"
        VDB[Vector Database<br/>Pinecone, Chroma]
        LLM[LLM Provider]
        Embed[Embedding Provider]
    end

    Dev --> UC1
    Dev --> UC2
    Dev --> UC3
    Dev --> UC4
    Dev --> UC5
    Dev --> UC6
    User --> UC7

    UC2 --> UC1
    UC3 --> UC2
    UC4 --> UC3
    UC5 --> UC4
    UC6 --> UC5
    UC7 --> UC6
    UC8 --> UC5

    UC3 -.-> Embed
    UC4 -.-> VDB
    UC5 -.-> VDB
    UC7 -.-> LLM

    style Dev fill:#e1f5ff
    style User fill:#fff4e1
    style VDB fill:#ffe1e1
    style LLM fill:#ffe1e1
    style Embed fill:#ffe1e1
```

**Primary Actor**: Application Developer, End User
**Goal**: Build a question-answering system over custom documents
**Components Used**: DocumentLoaders, TextSplitters, Embeddings, VectorStores, Retrievers, LCEL chains

---

## 3. Build an Agent with Tools

```mermaid
graph TB
    subgraph Actors
        Dev[Application Developer]
        User[End User]
    end

    subgraph "LangChain System"
        UC1[Define Tools]
        UC2[Create Tool Schemas]
        UC3[Configure Agent]
        UC4[Bind Tools to Model]
        UC5[Execute Agent Loop]
        UC6[Parse Tool Calls]
        UC7[Execute Tools]
        UC8[Return Results]
    end

    subgraph "External Systems"
        LLM[LLM Provider<br/>with function calling]
        APIs[External APIs<br/>Search, Calculator, DB]
    end

    Dev --> UC1
    Dev --> UC2
    Dev --> UC3
    Dev --> UC4
    User --> UC5

    UC2 --> UC1
    UC3 --> UC2
    UC4 --> UC3
    UC5 --> UC4
    UC6 --> UC5
    UC7 --> UC6
    UC8 --> UC7

    UC4 -.-> LLM
    UC5 -.-> LLM
    UC7 -.-> APIs

    style Dev fill:#e1f5ff
    style User fill:#fff4e1
    style LLM fill:#ffe1e1
    style APIs fill:#ffe1e1
```

**Primary Actor**: Application Developer, End User
**Goal**: Create an agent that can use tools to accomplish tasks
**Components Used**: BaseTool, StructuredTool, ChatModel with tool binding, Agent executors

---

## 4. Chain Multiple LLM Calls

```mermaid
graph TB
    subgraph Actors
        Dev[Application Developer]
        User[End User]
    end

    subgraph "LangChain System"
        UC1[Create First Prompt]
        UC2[Create Second Prompt]
        UC3[Chain with Pipe Operator]
        UC4[Add Output Parsers]
        UC5[Execute Sequential Chain]
        UC6[Pass Data Between Steps]
        UC7[Handle Intermediate Results]
    end

    subgraph "External Systems"
        LLM[LLM Provider]
    end

    Dev --> UC1
    Dev --> UC2
    Dev --> UC3
    Dev --> UC4
    User --> UC5

    UC3 --> UC1
    UC3 --> UC2
    UC4 --> UC3
    UC5 --> UC3
    UC6 --> UC5
    UC7 --> UC5

    UC5 -.-> LLM

    style Dev fill:#e1f5ff
    style User fill:#fff4e1
    style LLM fill:#ffe1e1
```

**Primary Actor**: Application Developer, End User
**Goal**: Create multi-step workflows with sequential LLM calls
**Components Used**: ChatPromptTemplate, ChatModel, OutputParsers, RunnableSequence (LCEL)
