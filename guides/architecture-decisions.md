# Architecture Decision Records (ADRs)

This document captures key architectural decisions made in the LangChain project.

---

## ADR-001: Monorepo Structure

**Status**: Accepted

**Context**:
LangChain consists of multiple packages (core, main, partner integrations) that need to be developed and released together while maintaining independent versioning.

**Decision**:
Use a Python monorepo structure with multiple independently versioned packages under a single repository.

**Structure**:
```
langchain/
├── libs/
│   ├── core/              # langchain-core
│   ├── langchain_v1/      # langchain (main package)
│   ├── langchain/         # langchain-classic (legacy)
│   ├── partners/          # Partner integrations
│   ├── text-splitters/    # Text splitting utilities
│   └── standard-tests/    # Shared test suite
```

**Rationale**:
1. **Coordinated Development**: Changes across packages can be made atomically
2. **Shared Tooling**: Single CI/CD pipeline, linting, and testing setup
3. **Code Sharing**: Easy to share code between packages during development
4. **Independent Releases**: Each package can be versioned and released independently
5. **Easier Refactoring**: Cross-package refactoring is simpler
6. **Consistent Standards**: Enforces consistent code style and practices

**Consequences**:
- **Positive**:
  - Simplified dependency management during development
  - Easier to maintain consistency across packages
  - Single source of truth for documentation
  - Reduced overhead for contributors (one repo to clone)
  - Atomic commits across package boundaries

- **Negative**:
  - Larger repository size
  - More complex build system
  - Potential for tight coupling if not careful
  - Longer CI/CD times (testing all packages)

**Alternatives Considered**:
1. **Multi-repo**: Separate repository for each package
   - Rejected: Too much overhead for coordinated changes
2. **Single Package**: Everything in one package
   - Rejected: Forces users to install unnecessary dependencies

**Tools Used**:
- `uv` for fast dependency management
- `hatchling` for building packages
- `ruff` for linting
- `mypy` for type checking

---

## ADR-002: Separation of langchain-core from langchain

**Status**: Accepted

**Context**:
LangChain needs stable base abstractions that partner integrations can depend on without pulling in the entire framework.

**Decision**:
Create `langchain-core` as a separate package containing only base abstractions, interfaces, and protocols. The main `langchain` package builds on top of core.

**Core Contains**:
- Runnables (LCEL protocol)
- Base classes for language models, prompts, tools
- Output parsers
- Vector store interfaces
- Callback system
- Message types
- Document representation

**Main Package Contains**:
- Concrete implementations
- Pre-built chains
- Agent implementations
- Document loaders
- Retrievers
- High-level utilities

**Rationale**:
1. **Stable API**: Core provides a stable foundation that rarely changes
2. **Minimal Dependencies**: Core has minimal dependencies, reducing conflicts
3. **Partner Integration**: Partners only depend on core, not the full framework
4. **Faster Iteration**: Main package can evolve faster without breaking partners
5. **Clear Boundaries**: Separates abstractions from implementations
6. **Reduced Bloat**: Users can install only what they need

**Consequences**:
- **Positive**:
  - Partner packages have minimal dependencies
  - Core API is more stable and carefully considered
  - Easier to maintain backward compatibility
  - Clearer separation of concerns
  - Smaller install size for integration-only users

- **Negative**:
  - Two packages to maintain
  - Potential confusion about which package to use
  - Import paths are longer (`langchain_core.runnables` vs `langchain.runnables`)
  - Circular dependency risk (mitigated by clear layering)

**Alternatives Considered**:
1. **Single Package**: Everything in `langchain`
   - Rejected: Forces heavy dependencies on partner packages
2. **Three Layers**: core, abstractions, implementations
   - Rejected: Too complex, diminishing returns

**Migration Path**:
- `langchain-classic` maintains old API for backward compatibility
- `langchain` (v1) is the new main package
- Users gradually migrate from classic to v1

---

## ADR-003: LCEL (Runnables) Over Traditional Chains

**Status**: Accepted

**Context**:
Traditional chains (LLMChain, SequentialChain, etc.) had limitations:
- Inconsistent interfaces
- Limited composability
- No streaming support by default
- Difficult to debug
- Hard to extend

**Decision**:
Introduce LCEL (LangChain Expression Language) based on the Runnable protocol as the primary way to compose components.

**Runnable Protocol**:
```python
class Runnable(Generic[Input, Output]):
    def invoke(input: Input) -> Output
    def stream(input: Input) -> Iterator[Output]
    def batch(inputs: List[Input]) -> List[Output]
    # + async versions
```

**Composition Operators**:
- Pipe (`|`): Sequential composition
- Dict literal: Parallel composition
- Methods: `with_retry()`, `with_fallbacks()`, etc.

**Rationale**:
1. **Universal Interface**: All components implement the same protocol
2. **Composability**: Easy to chain components with `|` operator
3. **Built-in Features**: Streaming, batching, async support automatically
4. **Type Safety**: Generic types enable type checking
5. **Declarative**: Chains are data structures, not imperative code
6. **Debuggable**: Clear data flow, easy to inspect
7. **Extensible**: Easy to create custom runnables

**Consequences**:
- **Positive**:
  - Consistent API across all components
  - Automatic streaming and batching support
  - Better type safety and IDE support
  - Easier to test (runnables are pure functions)
  - More flexible composition patterns
  - Cleaner, more readable code

- **Negative**:
  - Learning curve for existing users
  - Breaking change from traditional chains
  - More abstract (can be harder for beginners)
  - Requires understanding of protocols and generics

**Alternatives Considered**:
1. **Keep Traditional Chains**: Improve existing chain classes
   - Rejected: Fundamental limitations in design
2. **Function Composition**: Use plain functions
   - Rejected: Loses benefits of protocol (streaming, callbacks, etc.)
3. **Graph-Based**: Use explicit graph structure
   - Partially adopted: LangGraph for complex workflows

**Migration Strategy**:
- Traditional chains remain in `langchain-classic`
- New code uses LCEL
- Documentation emphasizes LCEL patterns
- Conversion utilities help migrate old chains

---

## ADR-004: LangGraph for Agent Orchestration

**Status**: Accepted

**Context**:
Agents need complex control flow:
- Loops and cycles
- Conditional branching
- State management
- Human-in-the-loop
- Persistence and resumability

LCEL is great for linear/parallel flows but not for complex state machines.

**Decision**:
Create LangGraph as a separate framework for building stateful, graph-based agent workflows. LangGraph complements LCEL rather than replacing it.

**LangGraph Features**:
- Explicit state management
- Graph-based workflow definition
- Cycles and loops
- Conditional edges
- Persistence (checkpointing)
- Human-in-the-loop support
- Time travel debugging

**Rationale**:
1. **Explicit State**: Agents need to maintain state across steps
2. **Complex Control Flow**: Loops, branches, and cycles are common in agents
3. **Debuggability**: Graph visualization helps understand agent behavior
4. **Persistence**: Long-running agents need to save/resume state
5. **Human Oversight**: Production agents often need human approval
6. **Separation of Concerns**: Keep LCEL simple, use LangGraph for complexity

**Consequences**:
- **Positive**:
  - Clear mental model for complex workflows
  - Built-in persistence and resumability
  - Better debugging tools (graph visualization)
  - Explicit state management (easier to reason about)
  - Human-in-the-loop patterns built-in
  - Production-ready features (checkpointing, error recovery)

- **Negative**:
  - Another framework to learn
  - More boilerplate for simple agents
  - Potential confusion about when to use LCEL vs LangGraph
  - Separate documentation and examples needed

**When to Use Each**:
- **LCEL**: Linear or parallel workflows, simple chains, RAG pipelines
- **LangGraph**: Agents with loops, complex state, human-in-the-loop, persistence

**Alternatives Considered**:
1. **Extend LCEL**: Add state and cycles to Runnables
   - Rejected: Would complicate LCEL's simple model
2. **Use Existing Workflow Engine**: Airflow, Prefect, etc.
   - Rejected: Not designed for LLM-specific patterns
3. **Agent Executors Only**: Stick with simple agent executors
   - Rejected: Insufficient for production use cases

**Integration**:
- LangGraph nodes can be LCEL chains
- LangGraph is built on Runnable protocol
- Seamless interoperability between frameworks

---

## ADR-005: uv as Package Manager

**Status**: Accepted

**Context**:
Python package management has historically been slow and complex. The monorepo structure requires efficient dependency resolution and installation.

**Decision**:
Use `uv` as the primary package manager for the LangChain monorepo, replacing pip and poetry.

**uv Features**:
- Written in Rust (10-100x faster than pip)
- Compatible with pip and pyproject.toml
- Efficient dependency resolution
- Lockfile support (uv.lock)
- Workspace support for monorepos
- Drop-in replacement for pip

**Rationale**:
1. **Speed**: Dramatically faster installs (seconds vs minutes)
2. **Monorepo Support**: Native workspace support for multi-package repos
3. **Compatibility**: Works with existing pyproject.toml and requirements.txt
4. **Reliability**: Deterministic builds with lockfiles
5. **Developer Experience**: Faster CI/CD and local development
6. **Modern**: Active development, modern Python packaging standards

**Consequences**:
- **Positive**:
  - Much faster dependency installation
  - Faster CI/CD pipelines
  - Better developer experience (less waiting)
  - Reliable, reproducible builds
  - Native monorepo support
  - Compatible with existing tools

- **Negative**:
  - Requires uv installation (not in Python stdlib)
  - Newer tool (less mature than pip)
  - Contributors need to learn uv commands
  - Potential compatibility issues with some packages

**Migration**:
```bash
# Old (pip)
pip install -e .

# New (uv)
uv sync

# Old (pip with requirements)
pip install -r requirements.txt

# New (uv)
uv pip install -r requirements.txt
```

**Alternatives Considered**:
1. **pip**: Standard Python package manager
   - Rejected: Too slow for monorepo development
2. **poetry**: Popular alternative to pip
   - Rejected: Slower than uv, less monorepo support
3. **pdm**: Modern package manager
   - Rejected: uv is faster and more compatible

**Fallback**:
- pip still works for end users
- uv is primarily for development
- Published packages work with any package manager

---

## Summary of Key Decisions

| Decision | Rationale | Trade-off |
|----------|-----------|-----------|
| **Monorepo** | Coordinated development, shared tooling | Larger repo, complex build |
| **Core Separation** | Stable API, minimal dependencies | Two packages to maintain |
| **LCEL/Runnables** | Universal interface, composability | Learning curve, breaking change |
| **LangGraph** | Complex workflows, state management | Another framework to learn |
| **uv Package Manager** | Speed, monorepo support | Requires uv installation |

## Principles Behind Decisions

1. **Composability First**: Components should compose easily
2. **Stable Abstractions**: Core interfaces change rarely
3. **Developer Experience**: Fast, intuitive, well-documented
4. **Production Ready**: Built for real-world use cases
5. **Backward Compatibility**: Minimize breaking changes
6. **Performance**: Fast execution and development cycles
7. **Type Safety**: Leverage Python's type system
8. **Extensibility**: Easy to add new components

## Future Considerations

- **Streaming Protocol**: Standardize streaming across all components
- **Observability**: Built-in tracing and monitoring
- **Multi-modal**: Support for images, audio, video
- **Distributed Execution**: Run chains across multiple machines
- **Edge Deployment**: Support for edge/mobile deployment
- **Cost Optimization**: Automatic caching and batching

---

**Last Updated**: 2026-02-05