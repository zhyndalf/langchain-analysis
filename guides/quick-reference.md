# LangChain Quick Reference Guide

## LCEL Syntax Cheat Sheet

### Basic Composition

```python
# Sequential chain (pipe operator)
chain = component1 | component2 | component3

# Equivalent to
chain = RunnableSequence(first=component1, middle=[component2], last=component3)

# Parallel execution (dict literal)
parallel = {
    "key1": component1,
    "key2": component2
}

# Equivalent to
parallel = RunnableParallel(steps={"key1": component1, "key2": component2})
```

### Common Patterns

```python
# Simple chain: Prompt → Model → Parser
chain = prompt | model | StrOutputParser()

# RAG chain: Retriever → Format → Prompt → Model
chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# Multi-output chain
chain = prompt | model | {
    "summary": summarize_parser,
    "sentiment": sentiment_parser
}

# Conditional routing
from langchain_core.runnables import RunnableBranch

chain = RunnableBranch(
    (lambda x: x["type"] == "A", chain_a),
    (lambda x: x["type"] == "B", chain_b),
    default_chain
)
```

### Data Flow Control

```python
# Pass through input unchanged
RunnablePassthrough()

# Pass through and add new keys
RunnablePassthrough.assign(
    new_key=some_runnable,
    another_key=lambda x: x["field"] * 2
)

# Extract specific fields
from operator import itemgetter
itemgetter("field_name")

# Wrap arbitrary function
from langchain_core.runnables import RunnableLambda
RunnableLambda(lambda x: x.upper())
```

---

## Runnable Methods Reference

### Invocation Methods

```python
# Synchronous single input
result = runnable.invoke(input, config=config)

# Asynchronous single input
result = await runnable.ainvoke(input, config=config)

# Synchronous batch
results = runnable.batch([input1, input2, input3], config=config)

# Asynchronous batch
results = await runnable.abatch([input1, input2, input3], config=config)

# Synchronous streaming
for chunk in runnable.stream(input, config=config):
    print(chunk)

# Asynchronous streaming
async for chunk in runnable.astream(input, config=config):
    print(chunk)

# Detailed event streaming
async for event in runnable.astream_events(input, version="v2"):
    print(event)

# Log streaming (deprecated, use astream_events)
async for log in runnable.astream_log(input):
    print(log)
```

### Configuration Methods

```python
# Bind configuration
configured = runnable.with_config(
    tags=["production"],
    metadata={"user_id": "123"},
    run_name="my_chain"
)

# Add retry logic
reliable = runnable.with_retry(
    stop_after_attempt=3,
    wait_exponential_jitter=True
)

# Add fallbacks
robust = runnable.with_fallbacks([fallback1, fallback2])

# Add lifecycle listeners
monitored = runnable.with_listeners(
    on_start=lambda run: print("Started"),
    on_end=lambda run: print("Ended"),
    on_error=lambda run: print("Error")
)

# Bind kwargs (for models)
model_with_params = model.bind(temperature=0.7, max_tokens=100)

# Bind tools (for chat models)
model_with_tools = model.bind_tools([tool1, tool2])
```

### Schema Methods

```python
# Get input schema (Pydantic model)
input_schema = runnable.input_schema

# Get output schema (Pydantic model)
output_schema = runnable.output_schema

# Get config schema
config_schema = runnable.config_schema()

# Get JSON schema
input_json = runnable.input_schema.model_json_schema()
output_json = runnable.output_schema.model_json_schema()
```

### Composition Methods

```python
# Pipe to another runnable
chained = runnable.pipe(other_runnable)

# Same as
chained = runnable | other_runnable

# Pick specific output fields
from langchain_core.runnables import RunnablePick
picked = runnable | RunnablePick("field_name")

# Assign new fields to output
assigned = runnable.assign(new_field=other_runnable)
```

---

## Prompt Templates Reference

### String Templates

```python
from langchain_core.prompts import PromptTemplate

# Basic template
prompt = PromptTemplate.from_template("Tell me about {topic}")

# With multiple variables
prompt = PromptTemplate.from_template(
    "Write a {length} {style} about {topic}"
)

# With custom template format
prompt = PromptTemplate(
    template="Tell me about $topic",
    input_variables=["topic"],
    template_format="string"  # or "jinja2", "f-string"
)

# Partial variables
prompt = PromptTemplate.from_template("Today is {date}. {question}")
prompt_partial = prompt.partial(date="2024-01-01")
```

### Chat Templates

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

# From messages
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("human", "{input}")
])

# With message placeholder
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])

# From template
prompt = ChatPromptTemplate.from_template(
    "You are a {role}. Answer: {question}"
)

# Multiple message types
from langchain_core.prompts import HumanMessagePromptTemplate, SystemMessagePromptTemplate

prompt = ChatPromptTemplate.from_messages([
    SystemMessagePromptTemplate.from_template("You are a {role}"),
    HumanMessagePromptTemplate.from_template("{question}")
])
```

### Few-Shot Templates

```python
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate

# Define examples
examples = [
    {"input": "happy", "output": "sad"},
    {"input": "tall", "output": "short"},
]

# Example prompt
example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="Input: {input}\\nOutput: {output}"
)

# Few-shot prompt
prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    prefix="Give the opposite of the word:",
    suffix="Input: {input}\\nOutput:",
    input_variables=["input"]
)
```

---

## Tool Definition Reference

### Simple Tool (Decorator)

```python
from langchain_core.tools import tool

@tool
def search(query: str) -> str:
    """Search for information.

    Args:
        query: The search query
    """
    return f"Results for {query}"

# Use it
result = search.invoke({"query": "LangChain"})
```

### Structured Tool

```python
from langchain_core.tools import StructuredTool
from pydantic import BaseModel, Field

class SearchInput(BaseModel):
    query: str = Field(description="The search query")
    max_results: int = Field(default=5, description="Maximum results")

def search_function(query: str, max_results: int = 5) -> str:
    return f"Top {max_results} results for {query}"

search_tool = StructuredTool.from_function(
    func=search_function,
    name="search",
    description="Search for information",
    args_schema=SearchInput
)
```

### BaseTool Subclass

```python
from langchain_core.tools import BaseTool
from pydantic import Field

class CustomTool(BaseTool):
    name: str = "custom_tool"
    description: str = "A custom tool"
    api_key: str = Field(exclude=True)  # Private field

    def _run(self, query: str) -> str:
        """Synchronous implementation."""
        return f"Result for {query}"

    async def _arun(self, query: str) -> str:
        """Asynchronous implementation."""
        return f"Async result for {query}"

tool = CustomTool(api_key="secret")
```

### Tool Binding

```python
from langchain_openai import ChatOpenAI

# Bind tools to model
model = ChatOpenAI(model="gpt-4")
model_with_tools = model.bind_tools([tool1, tool2, tool3])

# Invoke
response = model_with_tools.invoke("Use the search tool")

# Check for tool calls
if response.tool_calls:
    for tool_call in response.tool_calls:
        print(f"Tool: {tool_call['name']}")
        print(f"Args: {tool_call['args']}")
```

---

## Agent Patterns Reference

### ReAct Agent

```python
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}"),
])

agent = create_tool_calling_agent(
    llm=ChatOpenAI(model="gpt-4"),
    tools=[tool1, tool2],
    prompt=prompt
)

agent_executor = AgentExecutor(
    agent=agent,
    tools=[tool1, tool2],
    verbose=True,
    max_iterations=5,
    handle_parsing_errors=True
)

result = agent_executor.invoke({"input": "Your question"})
```

### Structured Output Agent

```python
from langchain_openai import ChatOpenAI
from pydantic import BaseModel, Field

class Response(BaseModel):
    answer: str = Field(description="The answer")
    confidence: float = Field(description="Confidence 0-1")
    sources: list[str] = Field(description="Sources used")

model = ChatOpenAI()
structured_model = model.with_structured_output(Response)

result = structured_model.invoke("Your question")
print(result.answer)
print(result.confidence)
```

### Custom Agent Loop

```python
from langchain_core.messages import HumanMessage, AIMessage, ToolMessage

def run_agent(query: str, max_iterations: int = 5):
    messages = [HumanMessage(content=query)]

    for i in range(max_iterations):
        # Call model
        response = model_with_tools.invoke(messages)
        messages.append(response)

        # Check if done
        if not response.tool_calls:
            return response.content

        # Execute tools
        for tool_call in response.tool_calls:
            tool = tool_map[tool_call["name"]]
            result = tool.invoke(tool_call["args"])
            messages.append(ToolMessage(
                content=str(result),
                tool_call_id=tool_call["id"]
            ))

    return "Max iterations reached"
```

---

## Vector Store Operations Reference

### Initialize Vector Store

```python
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma, FAISS, Pinecone

# Chroma (local)
vectorstore = Chroma(
    embedding_function=OpenAIEmbeddings(),
    persist_directory="./chroma_db"
)

# FAISS (local)
vectorstore = FAISS.from_documents(
    documents=docs,
    embedding=OpenAIEmbeddings()
)

# Pinecone (cloud)
vectorstore = Pinecone.from_documents(
    documents=docs,
    embedding=OpenAIEmbeddings(),
    index_name="my-index"
)
```

### Add Documents

```python
from langchain_core.documents import Document

# Add documents
docs = [
    Document(page_content="Content 1", metadata={"source": "doc1"}),
    Document(page_content="Content 2", metadata={"source": "doc2"}),
]
vectorstore.add_documents(docs)

# Add texts directly
vectorstore.add_texts(
    texts=["Text 1", "Text 2"],
    metadatas=[{"source": "t1"}, {"source": "t2"}]
)
```

### Search Operations

```python
# Similarity search
results = vectorstore.similarity_search(
    query="search query",
    k=4  # top 4 results
)

# Similarity search with scores
results = vectorstore.similarity_search_with_score(
    query="search query",
    k=4
)
for doc, score in results:
    print(f"Score: {score}, Content: {doc.page_content}")

# MMR (Maximum Marginal Relevance) search
results = vectorstore.max_marginal_relevance_search(
    query="search query",
    k=4,
    fetch_k=20,  # fetch more, then diversify
    lambda_mult=0.5  # diversity vs relevance
)

# Filtered search
results = vectorstore.similarity_search(
    query="search query",
    k=4,
    filter={"source": "doc1"}
)
```

### Create Retriever

```python
# Basic retriever
retriever = vectorstore.as_retriever()

# Configured retriever
retriever = vectorstore.as_retriever(
    search_type="similarity",  # or "mmr", "similarity_score_threshold"
    search_kwargs={
        "k": 4,
        "score_threshold": 0.5,  # for similarity_score_threshold
        "fetch_k": 20,  # for mmr
        "lambda_mult": 0.5,  # for mmr
        "filter": {"source": "doc1"}
    }
)

# Use retriever
docs = retriever.invoke("query")

# In a chain
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
)
```

### Delete and Update

```python
# Delete by ID
vectorstore.delete(ids=["id1", "id2"])

# Update (delete + add)
vectorstore.delete(ids=["id1"])
vectorstore.add_documents([updated_doc])

# Clear all
vectorstore.delete(delete_all=True)
```

---

## Configuration Reference

### RunnableConfig

```python
from langchain_core.runnables import RunnableConfig

config = RunnableConfig(
    tags=["production", "v1"],
    metadata={"user_id": "123", "session_id": "abc"},
    callbacks=[callback_handler],
    run_name="my_chain_run",
    max_concurrency=5,
    recursion_limit=25,
    configurable={"model": "gpt-4", "temperature": 0.7}
)

result = runnable.invoke(input, config=config)
```

### Common Config Patterns

```python
# Development config
dev_config = {
    "tags": ["dev"],
    "callbacks": [ConsoleCallbackHandler()],
    "configurable": {"temperature": 1.0}
}

# Production config
prod_config = {
    "tags": ["prod"],
    "metadata": {"version": "1.0"},
    "max_concurrency": 10,
    "configurable": {"temperature": 0.3}
}

# With LangSmith tracing
traced_config = {
    "tags": ["experiment"],
    "metadata": {"experiment_id": "exp_123"},
    "callbacks": [LangChainTracer()]
}
```

---

## Common Imports

```python
# Core
from langchain_core.runnables import (
    Runnable,
    RunnableSequence,
    RunnableParallel,
    RunnableLambda,
    RunnablePassthrough,
    RunnableBranch,
)
from langchain_core.prompts import (
    PromptTemplate,
    ChatPromptTemplate,
    MessagesPlaceholder,
    FewShotPromptTemplate,
)
from langchain_core.output_parsers import (
    StrOutputParser,
    JsonOutputParser,
    PydanticOutputParser,
)
from langchain_core.messages import (
    HumanMessage,
    AIMessage,
    SystemMessage,
    ToolMessage,
)
from langchain_core.documents import Document
from langchain_core.tools import tool, BaseTool, StructuredTool

# Models (choose your provider)
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_anthropic import ChatAnthropic
from langchain_ollama import ChatOllama

# Vector stores
from langchain_community.vectorstores import Chroma, FAISS

# Text splitters
from langchain_text_splitters import (
    RecursiveCharacterTextSplitter,
    CharacterTextSplitter,
)

# Agents
from langchain.agents import create_tool_calling_agent, AgentExecutor

# Memory
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_community.chat_message_histories import ChatMessageHistory
```

---

## Quick Tips

1. **Always use type hints** - They help with debugging and IDE support
2. **Use streaming for better UX** - Users see progress in real-time
3. **Add retries for reliability** - LLM APIs can be flaky
4. **Use callbacks for monitoring** - Essential for production
5. **Leverage caching** - Save costs on repeated queries
6. **Test with small models first** - Faster iteration, lower costs
7. **Use structured output** - More reliable than string parsing
8. **Implement proper error handling** - LLMs can fail unpredictably
9. **Monitor token usage** - Costs can add up quickly
10. **Use async when possible** - Better performance for I/O operations