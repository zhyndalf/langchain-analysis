# C4 Level 3: Component Diagram (langchain-core)

This diagram shows the major components within the langchain-core package.

```mermaid
C4Component
    title Component Diagram for langchain-core Package

    Container_Boundary(langchain_core, "langchain-core") {
        Component(runnables, "Runnables", "Python Module", "LCEL protocol - base classes for composable components (Runnable, RunnableSequence, RunnableParallel)")

        Component(language_models, "Language Models", "Python Module", "Base classes for LLMs and Chat Models (BaseChatModel, BaseLLM)")

        Component(prompts, "Prompts", "Python Module", "Template system (PromptTemplate, ChatPromptTemplate, FewShotPromptTemplate)")

        Component(tools, "Tools", "Python Module", "Tool definitions and schemas (BaseTool, StructuredTool, tool decorator)")

        Component(output_parsers, "Output Parsers", "Python Module", "Parse LLM outputs (StrOutputParser, JsonOutputParser, PydanticOutputParser)")

        Component(vectorstores, "Vector Stores", "Python Module", "Vector database interfaces (VectorStore, VectorStoreRetriever)")

        Component(embeddings, "Embeddings", "Python Module", "Text embedding interfaces (Embeddings base class)")

        Component(callbacks, "Callbacks", "Python Module", "Event system for tracing and monitoring (BaseCallbackHandler, CallbackManager)")

        Component(messages, "Messages", "Python Module", "Chat message types (HumanMessage, AIMessage, SystemMessage, ToolMessage)")

        Component(documents, "Documents", "Python Module", "Document representation (Document class with page_content and metadata)")

        Component(retrievers, "Retrievers", "Python Module", "Document retrieval interfaces (BaseRetriever)")

        Component(memory, "Memory", "Python Module", "Conversation history interfaces (BaseChatMessageHistory)")
    }

    Person(developer, "Developer", "Uses langchain-core abstractions")
    Container(langchain_impl, "langchain", "Concrete implementations")
    Container(partner_packages, "Partner Packages", "Provider-specific implementations")

    Rel(developer, runnables, "Composes chains with", "LCEL syntax")
    Rel(developer, prompts, "Creates templates with")
    Rel(developer, tools, "Defines tools with")

    Rel(runnables, language_models, "Invokes")
    Rel(runnables, prompts, "Uses")
    Rel(runnables, output_parsers, "Pipes to")
    Rel(runnables, callbacks, "Emits events to")

    Rel(language_models, messages, "Sends/receives")
    Rel(language_models, tools, "Can call")
    Rel(language_models, callbacks, "Emits events to")

    Rel(vectorstores, embeddings, "Uses for similarity")
    Rel(retrievers, vectorstores, "Queries")
    Rel(retrievers, documents, "Returns")

    Rel(langchain_impl, runnables, "Extends")
    Rel(langchain_impl, language_models, "Implements")
    Rel(partner_packages, language_models, "Implements")
    Rel(partner_packages, embeddings, "Implements")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## Core Components

### 1. Runnables (LCEL)
The foundation of LangChain's composability. Key classes:
- `Runnable`: Base protocol with invoke/stream/batch methods
- `RunnableSequence`: Chain components sequentially (pipe operator `|`)
- `RunnableParallel`: Execute components in parallel
- `RunnablePassthrough`: Pass input through unchanged
- `RunnableLambda`: Wrap arbitrary functions

**Purpose**: Universal interface for all LangChain components

### 2. Language Models
Base abstractions for AI models:
- `BaseChatModel`: Chat-based models (messages in/out)
- `BaseLLM`: Text completion models (string in/out)
- Methods: `invoke()`, `stream()`, `batch()`, `ainvoke()` (async)

**Purpose**: Provider-agnostic model interface

### 3. Prompts
Template system for dynamic prompts:
- `PromptTemplate`: String-based templates with variables
- `ChatPromptTemplate`: Message-based templates
- `FewShotPromptTemplate`: Examples-based prompting
- `MessagesPlaceholder`: Dynamic message insertion

**Purpose**: Reusable, parameterized prompts

### 4. Tools
Function calling and tool integration:
- `BaseTool`: Abstract tool interface
- `StructuredTool`: Tools with Pydantic schemas
- `@tool` decorator: Convert functions to tools
- Tool schemas define inputs/outputs

**Purpose**: Extend LLM capabilities with external functions

### 5. Output Parsers
Parse and validate LLM outputs:
- `StrOutputParser`: Extract string content
- `JsonOutputParser`: Parse JSON responses
- `PydanticOutputParser`: Validate with Pydantic models
- `CommaSeparatedListOutputParser`: Parse lists

**Purpose**: Structure unstructured LLM outputs

### 6. Vector Stores
Embedding storage and retrieval:
- `VectorStore`: Base interface for vector databases
- `VectorStoreRetriever`: Query interface
- Methods: `add_documents()`, `similarity_search()`, `as_retriever()`

**Purpose**: Semantic search for RAG applications

### 7. Embeddings
Text-to-vector conversion:
- `Embeddings`: Base interface
- Methods: `embed_documents()`, `embed_query()`

**Purpose**: Convert text to numerical representations

### 8. Callbacks
Event system for observability:
- `BaseCallbackHandler`: Event listener interface
- `CallbackManager`: Coordinate multiple handlers
- Events: on_llm_start, on_llm_end, on_chain_start, etc.

**Purpose**: Tracing, logging, and monitoring

### 9. Messages
Chat message types:
- `HumanMessage`: User input
- `AIMessage`: Model response
- `SystemMessage`: System instructions
- `ToolMessage`: Tool execution results
- `FunctionMessage`: Function call results

**Purpose**: Structured chat conversations

### 10. Documents
Document representation:
- `Document`: Contains `page_content` (text) and `metadata` (dict)

**Purpose**: Standardized document format

### 11. Retrievers
Document retrieval interface:
- `BaseRetriever`: Abstract retriever
- Methods: `get_relevant_documents()`, `aget_relevant_documents()`

**Purpose**: Fetch relevant documents for RAG

### 12. Memory
Conversation history:
- `BaseChatMessageHistory`: Store/retrieve messages
- Methods: `add_message()`, `get_messages()`, `clear()`

**Purpose**: Maintain conversation context

## Component Interactions

1. **LCEL Chains**: Runnables compose Prompts → Language Models → Output Parsers
2. **RAG Pipeline**: Retrievers query Vector Stores using Embeddings, results feed Language Models
3. **Agent Loop**: Language Models call Tools, results processed by Output Parsers
4. **Observability**: All components emit events to Callbacks
5. **Chat Applications**: Messages flow through Language Models with Memory
