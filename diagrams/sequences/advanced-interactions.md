# Additional Sequence Diagrams

## 4. Streaming Response

This sequence diagram shows streaming token-by-token responses.

```mermaid
sequenceDiagram
    actor User
    participant App as Application
    participant Chain as LCEL Chain
    participant Model as ChatModel
    participant Provider as LLM Provider
    participant Callbacks as CallbackManager

    User->>App: Submit query
    App->>Chain: astream(input)
    Chain->>Callbacks: on_chain_start

    Chain->>Model: astream(prompt)
    Model->>Callbacks: on_llm_start
    Model->>Provider: POST with stream=true

    loop For each token
        Provider-->>Model: Token chunk
        Model->>Callbacks: on_llm_new_token(token)
        Model-->>Chain: Yield token
        Chain-->>App: Yield token
        App-->>User: Display token (incremental)
    end

    Provider-->>Model: Stream complete
    Model->>Callbacks: on_llm_end
    Model-->>Chain: Stream complete
    Chain->>Callbacks: on_chain_end
    Chain-->>App: Stream complete
    App-->>User: Display complete
```

**Flow**:
1. User submits query
2. Application calls astream() for streaming
3. Model connects to provider with streaming enabled
4. Each token is yielded as it arrives
5. User sees incremental updates in real-time
6. Stream completes when all tokens received

---

## 5. LCEL Chain Composition

This sequence diagram shows how LCEL chains compose multiple components.

```mermaid
sequenceDiagram
    actor User
    participant App as Application
    participant Seq as RunnableSequence
    participant Prompt as PromptTemplate
    participant Model as ChatModel
    participant Parser as StrOutputParser
    participant Callbacks as CallbackManager

    User->>App: chain = prompt | model | parser
    Note over App,Parser: Chain construction (no execution)

    User->>App: chain.invoke({"topic": "AI"})
    App->>Seq: invoke(input)
    Seq->>Callbacks: on_chain_start

    Seq->>Prompt: invoke({"topic": "AI"})
    Prompt->>Callbacks: on_chain_start
    Prompt-->>Seq: Formatted prompt string
    Prompt->>Callbacks: on_chain_end

    Seq->>Model: invoke(prompt_string)
    Model->>Callbacks: on_llm_start
    Note over Model: Call LLM provider
    Model-->>Seq: AIMessage
    Model->>Callbacks: on_llm_end

    Seq->>Parser: invoke(AIMessage)
    Parser->>Callbacks: on_chain_start
    Parser-->>Seq: Extracted string content
    Parser->>Callbacks: on_chain_end

    Seq->>Callbacks: on_chain_end
    Seq-->>App: Final string output
    App-->>User: Display result
```

**Flow**:
1. User constructs chain using pipe operator
2. Chain is invoked with input
3. Each component processes output from previous step
4. Prompt formats input → Model generates response → Parser extracts content
5. Final output returned to user
6. Each step emits callbacks for tracing

---

## 6. Memory Integration

This sequence diagram shows conversation memory in action.

```mermaid
sequenceDiagram
    actor User
    participant App as Application
    participant Memory as ChatMessageHistory
    participant Store as Memory Store (Redis/DB)
    participant Chain as Chat Chain
    participant Model as ChatModel
    participant Callbacks as CallbackManager

    User->>App: First message: "Hello"
    App->>Memory: get_messages(session_id)
    Memory->>Store: LRANGE messages:session_id
    Store-->>Memory: [] (empty)
    Memory-->>App: []

    App->>Chain: invoke({<br/>  "input": "Hello",<br/>  "history": []<br/>})
    Chain->>Model: invoke with history
    Model-->>Chain: "Hi! How can I help?"
    Chain-->>App: Response

    App->>Memory: add_message(HumanMessage("Hello"))
    Memory->>Store: RPUSH messages:session_id
    App->>Memory: add_message(AIMessage("Hi! How can I help?"))
    Memory->>Store: RPUSH messages:session_id
    App-->>User: "Hi! How can I help?"

    Note over User,Store: Second message in same session

    User->>App: "What's my name?"
    App->>Memory: get_messages(session_id)
    Memory->>Store: LRANGE messages:session_id
    Store-->>Memory: [HumanMsg, AIMsg]
    Memory-->>App: Previous messages

    App->>Chain: invoke({<br/>  "input": "What's my name?",<br/>  "history": [prev messages]<br/>})
    Chain->>Model: invoke with full history
    Model-->>Chain: "I don't know your name yet"
    Chain-->>App: Response

    App->>Memory: add_message(HumanMessage("What's my name?"))
    Memory->>Store: RPUSH messages:session_id
    App->>Memory: add_message(AIMessage("I don't know..."))
    Memory->>Store: RPUSH messages:session_id
    App-->>User: "I don't know your name yet"
```

**Flow**:
1. First message: Memory is empty
2. Response generated without context
3. Both user and AI messages saved to memory
4. Second message: Previous messages loaded from memory
5. Full conversation history included in prompt
6. LLM has context from previous interaction
7. New messages appended to memory

---

## Interaction Patterns Summary

### Request-Response Pattern
User → App → LLM → App → User

### Retrieval Pattern
User → App → VectorDB → App → LLM → App → User

### Tool Calling Pattern
User → App → LLM → Tool → LLM → App → User

### Streaming Pattern
User → App → LLM → (Token → User)* → Complete

### Sequential Processing Pattern
Input → Component1 → Component2 → Component3 → Output

### Memory Pattern
User → Load History → LLM with Context → Save History → User

## Key Observations

1. **Callbacks are pervasive**: Every operation emits events for observability
2. **Async by default**: Most operations support both sync and async
3. **Composability**: Components chain together seamlessly
4. **Stateless execution**: State (like memory) is explicitly managed
5. **Provider abstraction**: LLM providers are interchangeable
6. **Streaming support**: Built into the protocol at every level