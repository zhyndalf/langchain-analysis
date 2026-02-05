# Flowcharts for Key LangChain Processes

## 1. LCEL Execution Flow

This flowchart shows how a Runnable chain is executed using the LCEL protocol.

```mermaid
flowchart TD
    Start([User calls chain.invoke input]) --> CheckType{Is Runnable a<br/>Sequence?}

    CheckType -->|Yes| SeqStart[Get first, middle, last runnables]
    CheckType -->|No| CheckParallel{Is Runnable a<br/>Parallel?}

    SeqStart --> InvokeFirst[Invoke first runnable with input]
    InvokeFirst --> HasMiddle{Has middle<br/>runnables?}
    HasMiddle -->|Yes| InvokeMiddle[Invoke each middle runnable<br/>with previous output]
    HasMiddle -->|No| InvokeLast
    InvokeMiddle --> InvokeLast[Invoke last runnable]
    InvokeLast --> EmitEnd

    CheckParallel -->|Yes| ParStart[Get steps dictionary]
    CheckParallel -->|No| CheckLambda{Is Runnable a<br/>Lambda?}

    ParStart --> InvokeParallel[Invoke all steps concurrently<br/>with same input]
    InvokeParallel --> CollectResults[Collect results into dictionary]
    CollectResults --> EmitEnd

    CheckLambda -->|Yes| InvokeFunc[Call wrapped function]
    CheckLambda -->|No| InvokeCustom[Call custom invoke method]

    InvokeFunc --> EmitEnd
    InvokeCustom --> EmitEnd

    EmitEnd[Emit on_chain_end callback] --> ReturnOutput([Return output])

    Start --> EmitStart[Emit on_chain_start callback]
    EmitStart --> CheckConfig{Has config?}
    CheckConfig -->|Yes| ApplyConfig[Apply tags, metadata,<br/>callbacks from config]
    CheckConfig -->|No| CheckType
    ApplyConfig --> CheckType

    style Start fill:#e1f5ff
    style ReturnOutput fill:#e1ffe1
    style CheckType fill:#fff4e1
    style CheckParallel fill:#fff4e1
    style CheckLambda fill:#fff4e1
```

**Key Points**:
- All runnables emit callbacks at start and end
- Configuration (tags, metadata, callbacks) is applied before execution
- Sequences execute steps in order, passing output to next input
- Parallels execute all steps concurrently with the same input
- Lambdas wrap arbitrary functions

---

## 2. Agent Execution Loop

This flowchart shows the ReAct agent pattern with tool calling.

```mermaid
flowchart TD
    Start([User submits query]) --> InitAgent[Initialize agent with tools]
    InitAgent --> FormatPrompt[Format prompt with:<br/>- User query<br/>- Available tools<br/>- Chat history]

    FormatPrompt --> CallLLM[Call LLM with prompt]
    CallLLM --> ParseResponse{Response contains<br/>tool calls?}

    ParseResponse -->|Yes| ExtractTools[Extract tool names and arguments]
    ParseResponse -->|No| CheckFinal{Is final answer?}

    ExtractTools --> ValidateTools{All tools valid?}
    ValidateTools -->|No| ErrorMsg[Add error message to history]
    ValidateTools -->|Yes| ExecuteTools[Execute tools in parallel]

    ExecuteTools --> CollectToolResults[Collect tool results]
    CollectToolResults --> AddToHistory[Add tool calls and results<br/>to chat history]
    AddToHistory --> CheckMaxIter{Reached max<br/>iterations?}

    CheckMaxIter -->|Yes| ForceStop[Force stop, return<br/>partial result]
    CheckMaxIter -->|No| FormatPrompt

    ErrorMsg --> FormatPrompt

    CheckFinal -->|Yes| ExtractAnswer[Extract final answer]
    CheckFinal -->|No| AddResponse[Add response to history]

    AddResponse --> FormatPrompt
    ExtractAnswer --> ReturnResult([Return final answer])
    ForceStop --> ReturnResult

    style Start fill:#e1f5ff
    style ReturnResult fill:#e1ffe1
    style ParseResponse fill:#fff4e1
    style ValidateTools fill:#fff4e1
    style CheckFinal fill:#fff4e1
    style CheckMaxIter fill:#fff4e1
```

**Key Points**:
- Agent loops until it produces a final answer or hits max iterations
- Tools are validated before execution
- Tool results are added to chat history for context
- LLM decides when to stop by returning a final answer
- Errors are fed back to the LLM for correction

---

## 3. RAG Pipeline Flow

This flowchart shows the Retrieval Augmented Generation process.

```mermaid
flowchart TD
    Start([User submits query]) --> EmbedQuery[Generate query embedding]
    EmbedQuery --> SearchVectorDB[Search vector database<br/>for similar documents]

    SearchVectorDB --> RetrieveDocs[Retrieve top-k documents]
    RetrieveDocs --> CheckDocs{Documents found?}

    CheckDocs -->|No| NoDocsMsg[Return "No relevant<br/>documents found"]
    CheckDocs -->|Yes| FormatDocs[Format documents:<br/>- Extract page_content<br/>- Add metadata<br/>- Join with separators]

    FormatDocs --> BuildPrompt[Build prompt with:<br/>- System instructions<br/>- Retrieved context<br/>- User query]

    BuildPrompt --> CallLLM[Call LLM with context]
    CallLLM --> ParseResponse[Parse LLM response]
    ParseResponse --> CheckCitations{Include citations?}

    CheckCitations -->|Yes| AddSources[Add source metadata<br/>to response]
    CheckCitations -->|No| ReturnAnswer

    AddSources --> ReturnAnswer([Return answer with sources])
    NoDocsMsg --> ReturnAnswer

    style Start fill:#e1f5ff
    style ReturnAnswer fill:#e1ffe1
    style CheckDocs fill:#fff4e1
    style CheckCitations fill:#fff4e1
```

**Key Points**:
- Query is embedded using the same model as documents
- Vector database returns most similar documents
- Documents are formatted and injected into prompt
- LLM generates answer based on retrieved context
- Sources can be included for transparency

---
