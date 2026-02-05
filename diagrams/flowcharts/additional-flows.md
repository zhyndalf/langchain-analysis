# Additional Flowcharts

## 4. Streaming Flow

This flowchart shows how streaming responses work in LangChain.

```mermaid
flowchart TD
    Start([User calls chain.astream input]) --> EmitStart[Emit on_chain_start event]
    EmitStart --> CheckType{Runnable type?}

    CheckType -->|Sequence| StreamSeq[Stream through sequence]
    CheckType -->|Parallel| StreamPar[Stream parallel execution]
    CheckType -->|LLM| StreamLLM[Stream from LLM]

    StreamSeq --> FirstChunk[Get first chunk from first runnable]
    FirstChunk --> YieldFirst[Yield first chunk]
    YieldFirst --> MoreChunks{More chunks?}

    MoreChunks -->|Yes| NextChunk[Get next chunk]
    MoreChunks -->|No| NextRunnable{More runnables<br/>in sequence?}

    NextChunk --> TransformChunk[Pass chunk through<br/>next runnable]
    TransformChunk --> YieldTransformed[Yield transformed chunk]
    YieldTransformed --> MoreChunks

    NextRunnable -->|Yes| FirstChunk
    NextRunnable -->|No| EmitEnd

    StreamPar --> StartAll[Start all parallel streams]
    StartAll --> MergeStreams[Merge streams with keys]
    MergeStreams --> YieldMerged[Yield merged chunks]
    YieldMerged --> AllDone{All streams done?}
    AllDone -->|No| MergeStreams
    AllDone -->|Yes| EmitEnd

    StreamLLM --> ConnectLLM[Connect to LLM stream]
    ConnectLLM --> ReceiveToken[Receive token chunk]
    ReceiveToken --> EmitToken[Emit on_llm_new_token event]
    EmitToken --> YieldToken[Yield token to user]
    YieldToken --> MoreTokens{More tokens?}
    MoreTokens -->|Yes| ReceiveToken
    MoreTokens -->|No| EmitEnd

    EmitEnd[Emit on_chain_end event] --> Complete([Stream complete])

    style Start fill:#e1f5ff
    style Complete fill:#e1ffe1
    style CheckType fill:#fff4e1
    style MoreChunks fill:#fff4e1
    style NextRunnable fill:#fff4e1
    style AllDone fill:#fff4e1
    style MoreTokens fill:#fff4e1
```

**Key Points**:
- Streaming allows incremental output as it's generated
- Sequences stream through each runnable in order
- Parallels merge multiple streams
- LLMs stream token by token
- Events are emitted for observability

---

## 5. Error Handling & Retries

This flowchart shows how LangChain handles errors and retries.

```mermaid
flowchart TD
    Start([Invoke runnable]) --> TryExecute[Try to execute]
    TryExecute --> Success{Execution<br/>successful?}

    Success -->|Yes| EmitSuccess[Emit on_chain_end event]
    Success -->|No| CatchError[Catch exception]

    CatchError --> EmitError[Emit on_chain_error event]
    EmitError --> HasRetry{Has retry<br/>configuration?}

    HasRetry -->|No| HasFallback{Has fallback<br/>runnables?}
    HasRetry -->|Yes| CheckAttempts{Attempts < max?}

    CheckAttempts -->|No| HasFallback
    CheckAttempts -->|Yes| CalcBackoff[Calculate exponential backoff]

    CalcBackoff --> WaitBackoff[Wait backoff duration]
    WaitBackoff --> IncrementAttempt[Increment attempt counter]
    IncrementAttempt --> LogRetry[Log retry attempt]
    LogRetry --> TryExecute

    HasFallback -->|No| RaiseError[Raise exception to caller]
    HasFallback -->|Yes| NextFallback{More fallbacks<br/>to try?}

    NextFallback -->|No| RaiseError
    NextFallback -->|Yes| TryFallback[Try next fallback runnable]

    TryFallback --> FallbackSuccess{Fallback<br/>successful?}
    FallbackSuccess -->|Yes| EmitSuccess
    FallbackSuccess -->|No| NextFallback

    EmitSuccess --> ReturnResult([Return result])
    RaiseError --> ReturnError([Propagate error])

    style Start fill:#e1f5ff
    style ReturnResult fill:#e1ffe1
    style ReturnError fill:#ffe1e1
    style Success fill:#fff4e1
    style HasRetry fill:#fff4e1
    style CheckAttempts fill:#fff4e1
    style HasFallback fill:#fff4e1
    style NextFallback fill:#fff4e1
    style FallbackSuccess fill:#fff4e1
```

**Key Points**:
- Errors are caught and emitted as events
- Retries use exponential backoff
- Fallbacks are tried in order
- If all retries and fallbacks fail, error propagates
- Each attempt is logged for debugging

---

## 6. Callback System Flow

This flowchart shows how the callback/event system works.

```mermaid
flowchart TD
    Start([Runnable execution begins]) --> GetCallbacks[Get callbacks from config]
    GetCallbacks --> HasCallbacks{Callbacks<br/>configured?}

    HasCallbacks -->|No| Execute[Execute without callbacks]
    HasCallbacks -->|Yes| CreateManager[Create CallbackManager]

    CreateManager --> EmitStart[Emit on_chain_start]
    EmitStart --> NotifyStart[Notify all handlers:<br/>- on_chain_start]

    NotifyStart --> ExecuteWithCallbacks[Execute runnable]
    ExecuteWithCallbacks --> CheckType{Runnable type?}

    CheckType -->|LLM| EmitLLMStart[Emit on_llm_start]
    CheckType -->|Tool| EmitToolStart[Emit on_tool_start]
    CheckType -->|Retriever| EmitRetrieverStart[Emit on_retriever_start]
    CheckType -->|Other| ContinueExec

    EmitLLMStart --> StreamTokens{Streaming?}
    StreamTokens -->|Yes| EmitTokens[Emit on_llm_new_token<br/>for each token]
    StreamTokens -->|No| EmitLLMEnd
    EmitTokens --> EmitLLMEnd[Emit on_llm_end]

    EmitToolStart --> ExecuteTool[Execute tool]
    ExecuteTool --> EmitToolEnd[Emit on_tool_end]

    EmitRetrieverStart --> RetrieveDocs[Retrieve documents]
    RetrieveDocs --> EmitRetrieverEnd[Emit on_retriever_end]

    EmitLLMEnd --> ContinueExec[Continue execution]
    EmitToolEnd --> ContinueExec
    EmitRetrieverEnd --> ContinueExec
    ContinueExec --> CheckSuccess{Execution<br/>successful?}

    CheckSuccess -->|Yes| EmitChainEnd[Emit on_chain_end]
    CheckSuccess -->|No| EmitChainError[Emit on_chain_error]

    EmitChainEnd --> NotifyEnd[Notify all handlers]
    EmitChainError --> NotifyError[Notify all handlers]

    NotifyEnd --> Cleanup[Cleanup callback manager]
    NotifyError --> Cleanup
    Execute --> Cleanup

    Cleanup --> Complete([Execution complete])

    style Start fill:#e1f5ff
    style Complete fill:#e1ffe1
    style HasCallbacks fill:#fff4e1
    style CheckType fill:#fff4e1
    style StreamTokens fill:#fff4e1
    style CheckSuccess fill:#fff4e1
```

**Key Points**:
- Callbacks are configured via RunnableConfig
- CallbackManager coordinates multiple handlers
- Different events for different component types
- Streaming emits token-by-token events
- All handlers are notified of start, end, and error events
- Callbacks enable tracing, logging, and monitoring

---

## Flow Patterns Summary

### Sequential Flow (LCEL)
Input → Runnable 1 → Runnable 2 → ... → Output

### Parallel Flow
Input → [Runnable A, Runnable B, Runnable C] → {a: Output A, b: Output B, c: Output C}

### Loop Flow (Agent)
Input → LLM → Tool → LLM → Tool → ... → Final Answer

### Streaming Flow
Input → Runnable → Chunk 1 → Chunk 2 → ... → Complete

### Error Flow
Try → Error → Retry → Error → Fallback → Success/Error

### Event Flow
Start → Execute → Events → End → Callbacks