# Additional Use Case Diagrams

## 5. Implement Conversation Memory

```mermaid
graph TB
    subgraph Actors
        Dev[Application Developer]
        User[End User]
    end

    subgraph "LangChain System"
        UC1[Configure Memory Store]
        UC2[Load Chat History]
        UC3[Add Messages to History]
        UC4[Trim History by Token Limit]
        UC5[Include History in Prompt]
        UC6[Save New Messages]
        UC7[Clear History]
    end

    subgraph "External Systems"
        Store[Memory Store<br/>Redis, PostgreSQL, In-Memory]
        LLM[LLM Provider]
    end

    Dev --> UC1
    User --> UC2
    User --> UC3
    User --> UC6
    User --> UC7

    UC2 -.-> Store
    UC3 --> UC2
    UC4 --> UC2
    UC5 --> UC4
    UC6 -.-> Store
    UC7 -.-> Store

    UC5 -.-> LLM

    style Dev fill:#e1f5ff
    style User fill:#fff4e1
    style Store fill:#ffe1e1
    style LLM fill:#ffe1e1
```

**Primary Actor**: Application Developer, End User
**Goal**: Maintain conversation context across multiple interactions
**Components Used**: BaseChatMessageHistory, ChatMessageHistory, RunnableWithMessageHistory

---

## 6. Parse Structured Output

```mermaid
graph TB
    subgraph Actors
        Dev[Application Developer]
        User[End User]
    end

    subgraph "LangChain System"
        UC1[Define Pydantic Schema]
        UC2[Create Output Parser]
        UC3[Add Format Instructions to Prompt]
        UC4[Parse LLM Response]
        UC5[Validate Against Schema]
        UC6[Handle Parsing Errors]
        UC7[Use Structured Data]
    end

    subgraph "External Systems"
        LLM[LLM Provider]
    end

    Dev --> UC1
    Dev --> UC2
    Dev --> UC3
    User --> UC4

    UC2 --> UC1
    UC3 --> UC2
    UC4 --> UC3
    UC5 --> UC4
    UC6 --> UC4
    UC7 --> UC5

    UC4 -.-> LLM

    style Dev fill:#e1f5ff
    style User fill:#fff4e1
    style LLM fill:#ffe1e1
```

**Primary Actor**: Application Developer, End User
**Goal**: Extract structured data from LLM responses
**Components Used**: PydanticOutputParser, JsonOutputParser, StructuredOutputParser

---

## 7. Stream Responses

```mermaid
graph TB
    subgraph Actors
        Dev[Application Developer]
        User[End User]
    end

    subgraph "LangChain System"
        UC1[Configure Streaming Chain]
        UC2[Invoke with astream]
        UC3[Receive Token Chunks]
        UC4[Display Incremental Output]
        UC5[Handle Stream Events]
        UC6[Track Intermediate Steps]
        UC7[Aggregate Final Result]
    end

    subgraph "External Systems"
        LLM[LLM Provider<br/>with streaming support]
    end

    Dev --> UC1
    User --> UC2
    User --> UC4

    UC2 --> UC1
    UC3 --> UC2
    UC4 --> UC3
    UC5 --> UC3
    UC6 --> UC5
    UC7 --> UC3

    UC2 -.-> LLM
    UC3 -.-> LLM

    style Dev fill:#e1f5ff
    style User fill:#fff4e1
    style LLM fill:#ffe1e1
```

**Primary Actor**: Application Developer, End User
**Goal**: Provide real-time feedback as LLM generates responses
**Components Used**: astream(), astream_events(), StreamingCallbackHandler

---

## 8. Integrate with Vector Databases

```mermaid
graph TB
    subgraph Actors
        Dev[Application Developer]
        User[End User]
    end

    subgraph "LangChain System"
        UC1[Initialize Vector Store]
        UC2[Add Documents]
        UC3[Update Embeddings]
        UC4[Similarity Search]
        UC5[MMR Search]
        UC6[Filtered Search]
        UC7[Delete Documents]
        UC8[Create Retriever]
    end

    subgraph "External Systems"
        VDB[Vector Database<br/>Pinecone, Weaviate, Chroma, FAISS]
        Embed[Embedding Provider]
    end

    Dev --> UC1
    Dev --> UC2
    User --> UC4
    User --> UC5
    User --> UC6
    Dev --> UC7
    Dev --> UC8

    UC1 -.-> VDB
    UC2 -.-> VDB
    UC2 -.-> Embed
    UC3 -.-> Embed
    UC4 -.-> VDB
    UC5 -.-> VDB
    UC6 -.-> VDB
    UC7 -.-> VDB
    UC8 --> UC1

    style Dev fill:#e1f5ff
    style User fill:#fff4e1
    style VDB fill:#ffe1e1
    style Embed fill:#ffe1e1
```

**Primary Actor**: Application Developer, End User
**Goal**: Store and retrieve documents using semantic similarity
**Components Used**: VectorStore, Embeddings, VectorStoreRetriever

---

## Actor Descriptions

### Application Developer
- Configures and builds LangChain applications
- Defines schemas, prompts, and chains
- Integrates external services
- Deploys and maintains applications

### End User
- Interacts with deployed applications
- Submits queries and receives responses
- Provides feedback through the application

### External Systems
- **LLM Providers**: OpenAI, Anthropic, Google, Cohere, etc.
- **Vector Databases**: Pinecone, Weaviate, Chroma, FAISS, etc.
- **Memory Stores**: Redis, PostgreSQL, in-memory stores
- **External APIs**: Web search, calculators, databases, custom services
- **Embedding Providers**: OpenAI, Cohere, HuggingFace, etc.

## Common Patterns

1. **Setup Phase** (Developer): Configure components, define schemas, build chains
2. **Execution Phase** (User): Invoke chains, receive results
3. **Integration Phase** (System): Communicate with external services
4. **Monitoring Phase** (Developer): Track performance, debug issues

## Use Case Relationships

- **Extends**: RAG extends Simple Chatbot with document retrieval
- **Includes**: Agent with Tools includes Chain Multiple LLM Calls
- **Depends On**: Most use cases depend on Parse Structured Output and Stream Responses
- **Combines**: Production applications often combine Memory + RAG + Streaming