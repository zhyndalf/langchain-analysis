# C4 Level 1: System Context Diagram

This diagram shows the LangChain system in the context of its users and external systems.

```mermaid
C4Context
    title System Context Diagram for LangChain

    Person(developer, "Application Developer", "Builds LLM-powered applications using LangChain")
    Person(endUser, "End User", "Interacts with LLM applications built with LangChain")

    System(langchain, "LangChain Framework", "Python framework for building agents and LLM-powered applications with composable components")

    System_Ext(llmProviders, "LLM Providers", "OpenAI, Anthropic, Google, Cohere, etc.")
    System_Ext(vectorDBs, "Vector Databases", "Pinecone, Weaviate, Chroma, FAISS, etc.")
    System_Ext(externalAPIs, "External APIs & Tools", "Web search, calculators, databases, custom APIs")
    System_Ext(embeddingProviders, "Embedding Providers", "OpenAI, Cohere, HuggingFace, etc.")
    System_Ext(langsmith, "LangSmith", "Monitoring, evaluation, and debugging platform")
    System_Ext(langgraph, "LangGraph", "Low-level agent orchestration framework")

    Rel(developer, langchain, "Builds applications with", "Python API")
    Rel(endUser, langchain, "Interacts with applications built on")

    Rel(langchain, llmProviders, "Sends prompts to, receives completions from", "HTTP/API")
    Rel(langchain, vectorDBs, "Stores and retrieves embeddings", "Native clients")
    Rel(langchain, externalAPIs, "Calls tools and integrations", "HTTP/API")
    Rel(langchain, embeddingProviders, "Generates embeddings", "HTTP/API")
    Rel(langchain, langsmith, "Sends traces and metrics", "HTTP/API")
    Rel(langchain, langgraph, "Uses for agent orchestration", "Python API")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## Key Elements

### Users
- **Application Developer**: Uses LangChain to build LLM applications with chains, agents, and RAG systems
- **End User**: Interacts with the final applications (chatbots, search systems, etc.)

### External Systems
- **LLM Providers**: Core language models (GPT-4, Claude, Gemini, etc.)
- **Vector Databases**: Store and retrieve document embeddings for semantic search
- **External APIs & Tools**: Extend agent capabilities with real-world actions
- **Embedding Providers**: Convert text to vector representations
- **LangSmith**: Production monitoring and debugging
- **LangGraph**: Advanced agent workflows with state management

## System Boundary

LangChain acts as an orchestration layer that:
1. Provides unified interfaces to multiple LLM providers
2. Manages data flow between components
3. Handles prompt templating and output parsing
4. Coordinates tool calling and agent execution
5. Integrates with vector stores for RAG
6. Enables observability through LangSmith
