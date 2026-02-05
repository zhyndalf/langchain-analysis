# C4 Level 2: Container Diagram

This diagram shows the major containers (packages) within the LangChain system and their relationships.

```mermaid
C4Container
    title Container Diagram for LangChain Monorepo

    Person(developer, "Application Developer", "Builds LLM applications")

    Container_Boundary(langchain_system, "LangChain System") {
        Container(langchain_core, "langchain-core", "Python Package", "Base abstractions, interfaces, and protocols (Runnables, Language Models, Prompts, Tools)")

        Container(langchain_v1, "langchain", "Python Package", "High-level implementations, chains, agents, and utilities")

        Container(langchain_classic, "langchain-classic", "Python Package", "Legacy package (maintenance mode, no new features)")

        ContainerDb(text_splitters, "text-splitters", "Python Package", "Document chunking and splitting utilities")

        Container_Boundary(partners, "Partner Integrations") {
            Container(partner_openai, "langchain-openai", "Python Package", "OpenAI models and embeddings")
            Container(partner_anthropic, "langchain-anthropic", "Python Package", "Anthropic Claude integration")
            Container(partner_ollama, "langchain-ollama", "Python Package", "Local model support")
            Container(partner_others, "15+ other partners", "Python Packages", "Google, Cohere, AWS, Azure, etc.")
        }

        Container(standard_tests, "standard-tests", "Python Package", "Shared test suite for integration validation")
    }

    System_Ext(llm_providers, "LLM Providers", "External AI services")
    System_Ext(vector_dbs, "Vector Databases", "Embedding storage")
    System_Ext(external_tools, "External Tools", "APIs and services")

    Rel(developer, langchain_v1, "Uses", "pip install langchain")
    Rel(developer, partners, "Uses specific integrations", "pip install langchain-openai")

    Rel(langchain_v1, langchain_core, "Depends on", "Imports base classes")
    Rel(langchain_classic, langchain_core, "Depends on", "Legacy implementation")
    Rel(partners, langchain_core, "Implements interfaces from", "Extends base classes")
    Rel(langchain_v1, text_splitters, "Uses", "Document processing")
    Rel(standard_tests, partners, "Validates", "Integration tests")

    Rel(partners, llm_providers, "Calls", "HTTP/API")
    Rel(langchain_v1, vector_dbs, "Integrates with", "Native clients")
    Rel(langchain_v1, external_tools, "Invokes", "Tool calling")

    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```

## Key Containers

### Core Layer
- **langchain-core**: Foundation package with base abstractions
  - Runnables (LCEL protocol)
  - Language model interfaces
  - Prompt templates
  - Tool definitions
  - Output parsers
  - Vector store interfaces
  - Callback system

### Implementation Layer
- **langchain**: Main package for application development
  - Pre-built chains
  - Agent implementations
  - Memory systems
  - Document loaders
  - Retrievers
  - High-level utilities

- **langchain-classic**: Legacy package (maintenance only)
  - Older chain implementations
  - Deprecated patterns
  - Backward compatibility

### Integration Layer
- **Partner Packages**: Third-party service integrations
  - Each partner package implements langchain-core interfaces
  - Independently versioned and maintained
  - Examples: OpenAI, Anthropic, Ollama, Google, Cohere

### Supporting Packages
- **text-splitters**: Document chunking strategies
- **standard-tests**: Shared test suite for validating integrations

## Dependency Flow

1. **langchain-core** is the foundation - all other packages depend on it
2. **langchain** builds on core with concrete implementations
3. **Partner packages** implement core interfaces for specific providers
4. **Applications** typically depend on langchain + specific partner packages

## Monorepo Benefits

- Shared abstractions ensure interoperability
- Independent versioning allows rapid partner updates
- Standard tests validate all integrations consistently
- Coordinated releases maintain compatibility
