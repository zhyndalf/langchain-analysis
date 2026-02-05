# Sequence Diagrams for LangChain Interactions

## 1. Simple Chat Completion

This sequence diagram shows a basic chat interaction.

```mermaid
sequenceDiagram
    actor User
    participant App as Application
    participant Prompt as ChatPromptTemplate
    participant Model as ChatModel
    participant Provider as LLM Provider
    participant Callbacks as CallbackManager

    User->>App: Submit message
    App->>Callbacks: on_chain_start
    App->>Prompt: format({"input": message})
    Prompt-->>App: Formatted messages

    App->>Callbacks: on_llm_start
    App->>Model: invoke(messages)
    Model->>Provider: HTTP POST /chat/completions
    Provider-->>Model: Response with content
    Model->>Callbacks: on_llm_end
    Model-->>App: AIMessage

    App->>Callbacks: on_chain_end
    App-->>User: Display response
```

**Flow**:
1. User submits a message
2. Application formats message using prompt template
3. Formatted messages sent to chat model
4. Model calls external LLM provider
5. Response returned and displayed to user
6. Callbacks track the entire flow

---

## 2. Agent with Tools

This sequence diagram shows an agent using tools to answer a question.

```mermaid
sequenceDiagram
    actor User
    participant App as Application
    participant Agent as Agent Executor
    participant Model as ChatModel (with tools)
    participant Provider as LLM Provider
    participant Tool as Tool (e.g., Calculator)
    participant Callbacks as CallbackManager

    User->>App: "What is 25 * 17?"
    App->>Agent: invoke({"input": query})
    Agent->>Callbacks: on_chain_start

    Agent->>Model: invoke with tools bound
    Model->>Provider: POST with function calling
    Provider-->>Model: Tool call: calculator(25, 17)
    Model-->>Agent: AIMessage with tool_calls

    Agent->>Callbacks: on_tool_start
    Agent->>Tool: invoke({"a": 25, "b": 17})
    Tool-->>Agent: {"result": 425}
    Agent->>Callbacks: on_tool_end

    Agent->>Model: invoke with tool results
    Model->>Provider: POST with tool results
    Provider-->>Model: "The answer is 425"
    Model-->>Agent: AIMessage (final answer)

    Agent->>Callbacks: on_chain_end
    Agent-->>App: Final answer
    App-->>User: "The answer is 425"
```

**Flow**:
1. User asks a question requiring calculation
2. Agent sends query to LLM with available tools
3. LLM decides to use calculator tool
4. Agent executes tool and gets result
5. Agent sends tool result back to LLM
6. LLM formulates final answer
7. Answer returned to user

---

## 3. RAG Query

This sequence diagram shows a retrieval-augmented generation query.

```mermaid
sequenceDiagram
    actor User
    participant App as Application
    participant Retriever as VectorStoreRetriever
    participant VectorDB as Vector Database
    participant Embeddings as Embedding Model
    participant Prompt as ChatPromptTemplate
    participant Model as ChatModel
    participant Provider as LLM Provider
    participant Callbacks as CallbackManager

    User->>App: "What is LangChain?"
    App->>Callbacks: on_chain_start

    App->>Callbacks: on_retriever_start
    App->>Retriever: get_relevant_documents(query)
    Retriever->>Embeddings: embed_query("What is LangChain?")
    Embeddings-->>Retriever: Query vector
    Retriever->>VectorDB: similarity_search(vector, k=4)
    VectorDB-->>Retriever: [Doc1, Doc2, Doc3, Doc4]
    Retriever->>Callbacks: on_retriever_end
    Retriever-->>App: Retrieved documents

    App->>Prompt: format({<br/>  "context": docs,<br/>  "question": query<br/>})
    Prompt-->>App: Formatted prompt with context

    App->>Callbacks: on_llm_start
    App->>Model: invoke(prompt)
    Model->>Provider: POST with context
    Provider-->>Model: Answer based on context
    Model->>Callbacks: on_llm_end
    Model-->>App: AIMessage

    App->>Callbacks: on_chain_end
    App-->>User: Answer with sources
```

**Flow**:
1. User submits a question
2. Query is embedded and used to search vector database
3. Most relevant documents are retrieved
4. Documents are formatted into prompt as context
5. LLM generates answer based on retrieved context
6. Answer (optionally with sources) returned to user

---
