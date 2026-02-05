# LangChain Learning Path: 4-Week Guide

This comprehensive guide will take you from beginner to proficient in LangChain over 4 weeks.

## Prerequisites

- Python 3.9+ installed
- Basic understanding of Python (functions, classes, async/await)
- Familiarity with APIs and HTTP requests
- API key from at least one LLM provider (OpenAI, Anthropic, etc.)

## Setup

```bash
# Install LangChain and a provider
pip install langchain langchain-openai

# Or for Anthropic
pip install langchain langchain-anthropic

# Set your API key
export OPENAI_API_KEY="your-key-here"
# or
export ANTHROPIC_API_KEY="your-key-here"
```

---

## Week 1: Foundations

### Day 1-2: Understand Runnables and LCEL

**Concepts to Learn**:
- What is a Runnable?
- The LCEL (LangChain Expression Language) protocol
- Core methods: invoke(), stream(), batch()
- Pipe operator (`|`) for composition

**Reading**:
- `/home/zhyndalf/vibeCoding/langchain-fork/libs/core/langchain_core/runnables/base.py:124-250`
- LangChain docs: LCEL concepts

**Hands-on Exercise**:
```python
from langchain_core.runnables import RunnableLambda

# Create simple runnables
add_one = RunnableLambda(lambda x: x + 1)
multiply_two = RunnableLambda(lambda x: x * 2)

# Compose with pipe operator
chain = add_one | multiply_two

# Test different invocation methods
print(chain.invoke(5))  # (5 + 1) * 2 = 12
print(chain.batch([1, 2, 3]))  # [4, 6, 8]

# Inspect schemas
print(chain.input_schema.model_json_schema())
print(chain.output_schema.model_json_schema())
```

**Key Takeaways**:
- Runnables are the building blocks of LangChain
- All components implement the same protocol
- Composition is declarative and type-safe
- Chains automatically support sync, async, batch, and streaming

---

### Day 3-4: Learn Prompts and Chat Models

**Concepts to Learn**:
- PromptTemplate vs ChatPromptTemplate
- Message types (Human, AI, System, Tool)
- Variable substitution in prompts
- Chat model invocation

**Reading**:
- `/home/zhyndalf/vibeCoding/langchain-fork/libs/core/langchain_core/prompts/`
- `/home/zhyndalf/vibeCoding/langchain-fork/libs/core/langchain_core/language_models/chat_models.py`

**Hands-on Exercise**:
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

# Create a prompt template
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant that explains {topic} simply."),
    ("human", "{question}")
])

# Initialize model
model = ChatOpenAI(model="gpt-4", temperature=0.7)

# Compose and invoke
chain = prompt | model
response = chain.invoke({
    "topic": "quantum physics",
    "question": "What is superposition?"
})

print(response.content)
```

**Key Takeaways**:
- Prompts are templates with variables
- Chat models work with message objects
- System messages set behavior, human messages are user input
- Models are runnables and can be chained

---

### Day 5-7: Build Simple Chains and Practice Composition

**Concepts to Learn**:
- RunnableSequence (sequential chains)
- RunnableParallel (parallel execution)
- RunnablePassthrough (data flow control)
- Output parsers

**Hands-on Exercise**:
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# Build a chain with multiple steps
prompt = ChatPromptTemplate.from_template(
    "Write a {length} sentence about {topic}"
)
model = ChatOpenAI()
output_parser = StrOutputParser()

# Sequential chain
chain = prompt | model | output_parser

result = chain.invoke({"length": "short", "topic": "AI"})
print(result)

# Parallel chain
parallel_chain = {
    "short": ChatPromptTemplate.from_template("One sentence about {topic}") | model | output_parser,
    "long": ChatPromptTemplate.from_template("Three sentences about {topic}") | model | output_parser
}

results = parallel_chain.invoke({"topic": "machine learning"})
print(results)  # {"short": "...", "long": "..."}

# Using RunnablePassthrough
chain_with_passthrough = (
    RunnablePassthrough.assign(
        response=prompt | model | output_parser
    )
)

result = chain_with_passthrough.invoke({"length": "short", "topic": "AI"})
print(result)  # {"length": "short", "topic": "AI", "response": "..."}
```

**Practice Projects**:
1. Build a translation chain (English → Spanish → French)
2. Create a summarization chain with different lengths
3. Build a parallel chain that generates both a title and summary

**Key Takeaways**:
- Pipe operator creates sequential chains
- Dict literals create parallel chains
- RunnablePassthrough preserves input while adding new keys
- Output parsers extract structured data from responses

---

## Week 2: Core Components

### Day 1-2: Output Parsers and Structured Output

**Concepts to Learn**:
- StrOutputParser, JsonOutputParser
- PydanticOutputParser for validation
- Structured output with function calling
- Error handling in parsing

**Hands-on Exercise**:
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field

# Define output schema
class MovieReview(BaseModel):
    title: str = Field(description="Movie title")
    rating: int = Field(description="Rating from 1-10")
    summary: str = Field(description="Brief summary")
    pros: list[str] = Field(description="Positive aspects")
    cons: list[str] = Field(description="Negative aspects")

# Create parser
parser = PydanticOutputParser(pydantic_object=MovieReview)

# Build prompt with format instructions
prompt = ChatPromptTemplate.from_template(
    "Review the movie {movie_name}.\\n{format_instructions}"
)

# Chain it together
chain = (
    prompt.partial(format_instructions=parser.get_format_instructions())
    | ChatOpenAI(temperature=0)
    | parser
)

result = chain.invoke({"movie_name": "The Matrix"})
print(result.model_dump_json(indent=2))
```

**Key Takeaways**:
- Pydantic models define expected output structure
- Parsers validate and type-check LLM outputs
- Format instructions guide the LLM to produce correct format
- Structured output is more reliable than string parsing

---

### Day 3-4: Tools and Function Calling

**Concepts to Learn**:
- Defining tools with @tool decorator
- StructuredTool for complex tools
- Binding tools to models
- Tool calling and execution

**Hands-on Exercise**:
```python
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage, ToolMessage

# Define tools
@tool
def calculator(operation: str, a: float, b: float) -> float:
    """Perform basic arithmetic operations.

    Args:
        operation: One of 'add', 'subtract', 'multiply', 'divide'
        a: First number
        b: Second number
    """
    if operation == "add":
        return a + b
    elif operation == "subtract":
        return a - b
    elif operation == "multiply":
        return a * b
    elif operation == "divide":
        return a / b if b != 0 else "Error: Division by zero"

@tool
def get_weather(location: str) -> str:
    """Get current weather for a location."""
    # Mock implementation
    return f"The weather in {location} is sunny, 72°F"

# Bind tools to model
tools = [calculator, get_weather]
model = ChatOpenAI(model="gpt-4").bind_tools(tools)

# Invoke with a query that needs tools
response = model.invoke([HumanMessage(content="What is 25 * 17?")])

# Check if tool was called
if response.tool_calls:
    tool_call = response.tool_calls[0]
    print(f"Tool: {tool_call['name']}")
    print(f"Args: {tool_call['args']}")

    # Execute tool
    result = calculator.invoke(tool_call['args'])
    print(f"Result: {result}")
```

**Practice Projects**:
1. Create a tool that searches Wikipedia
2. Build a unit converter tool
3. Create a tool that validates email addresses

**Key Takeaways**:
- Tools extend LLM capabilities with external functions
- @tool decorator converts functions to tools
- Models with function calling can decide when to use tools
- Tool schemas are automatically generated from docstrings

---

### Day 5-7: Vector Stores and Embeddings

**Concepts to Learn**:
- What are embeddings?
- Vector similarity search
- Document loaders and text splitters
- VectorStore interface

**Hands-on Exercise**:
```python
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from langchain_core.documents import Document
from langchain_text_splitters import RecursiveCharacterTextSplitter

# Create sample documents
docs = [
    Document(page_content="LangChain is a framework for building LLM applications.",
             metadata={"source": "intro"}),
    Document(page_content="LCEL is the LangChain Expression Language for composing chains.",
             metadata={"source": "concepts"}),
    Document(page_content="Runnables are the core abstraction in LangChain.",
             metadata={"source": "architecture"}),
]

# Split documents
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=100,
    chunk_overlap=20
)
splits = text_splitter.split_documents(docs)

# Create embeddings and vector store
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(
    documents=splits,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# Similarity search
query = "What is LCEL?"
results = vectorstore.similarity_search(query, k=2)

for doc in results:
    print(f"Content: {doc.page_content}")
    print(f"Source: {doc.metadata['source']}")
    print("---")

# Create retriever
retriever = vectorstore.as_retriever(search_kwargs={"k": 2})
retrieved_docs = retriever.invoke(query)
```

**Practice Projects**:
1. Load a PDF and create a searchable vector store
2. Build a document Q&A system
3. Implement semantic search over your notes

**Key Takeaways**:
- Embeddings convert text to numerical vectors
- Similar texts have similar vectors
- Vector stores enable semantic search
- Text splitters chunk documents for better retrieval

---

## Week 3: Advanced Patterns

### Day 1-3: Agents and LangGraph Basics

**Concepts to Learn**:
- Agent reasoning patterns (ReAct)
- Agent executors
- Introduction to LangGraph
- State management in agents

**Hands-on Exercise**:
```python
from langchain_openai import ChatOpenAI
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool

# Define tools
@tool
def search(query: str) -> str:
    """Search for information."""
    return f"Search results for: {query}"

@tool
def calculator(expression: str) -> str:
    """Evaluate a mathematical expression."""
    try:
        return str(eval(expression))
    except:
        return "Invalid expression"

# Create agent prompt
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant. Use tools when needed."),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}"),
])

# Create agent
tools = [search, calculator]
model = ChatOpenAI(model="gpt-4", temperature=0)
agent = create_tool_calling_agent(model, tools, prompt)

# Create executor
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=5
)

# Run agent
result = agent_executor.invoke({
    "input": "What is 15% of 240? Then search for information about that number."
})
print(result["output"])
```

**Key Takeaways**:
- Agents can reason about which tools to use
- AgentExecutor manages the agent loop
- Agents iterate until they reach a final answer
- LangGraph provides more control over agent workflows

---

### Day 4-5: Memory and Conversation Management

**Concepts to Learn**:
- ChatMessageHistory
- Conversation buffer memory
- Conversation summary memory
- RunnableWithMessageHistory

**Hands-on Exercise**:
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_community.chat_message_histories import ChatMessageHistory

# In-memory store for demo
store = {}

def get_session_history(session_id: str):
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]

# Create chain with memory
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])

chain = prompt | ChatOpenAI()

# Wrap with message history
chain_with_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history"
)

# Use with session
config = {"configurable": {"session_id": "user123"}}

response1 = chain_with_history.invoke(
    {"input": "My name is Alice"},
    config=config
)
print(response1.content)

response2 = chain_with_history.invoke(
    {"input": "What's my name?"},
    config=config
)
print(response2.content)  # Should remember "Alice"
```

**Key Takeaways**:
- Memory stores conversation history
- Session IDs separate different conversations
- MessagesPlaceholder injects history into prompts
- RunnableWithMessageHistory automates memory management

---

### Day 6-7: Streaming and Callbacks

**Concepts to Learn**:
- Streaming with astream()
- astream_events() for detailed events
- Custom callback handlers
- Tracing with LangSmith

**Hands-on Exercise**:
```python
import asyncio
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.callbacks import BaseCallbackHandler

# Custom callback handler
class MyCallbackHandler(BaseCallbackHandler):
    def on_llm_start(self, serialized, prompts, **kwargs):
        print(f"LLM started with {len(prompts)} prompts")

    def on_llm_end(self, response, **kwargs):
        print(f"LLM finished")

    def on_llm_new_token(self, token: str, **kwargs):
        print(f"Token: {token}", end="", flush=True)

# Create chain
prompt = ChatPromptTemplate.from_template("Tell me a story about {topic}")
model = ChatOpenAI(streaming=True)
chain = prompt | model | StrOutputParser()

# Stream with callback
async def stream_example():
    async for chunk in chain.astream(
        {"topic": "a robot"},
        config={"callbacks": [MyCallbackHandler()]}
    ):
        print(chunk, end="", flush=True)

asyncio.run(stream_example())

# Using astream_events for detailed tracking
async def events_example():
    async for event in chain.astream_events(
        {"topic": "a robot"},
        version="v2"
    ):
        if event["event"] == "on_chat_model_stream":
            print(event["data"]["chunk"].content, end="", flush=True)

asyncio.run(events_example())
```

**Key Takeaways**:
- Streaming provides real-time feedback
- astream() yields output chunks
- astream_events() provides detailed event stream
- Callbacks enable custom logging and monitoring

---

## Week 4: Real Applications

### Day 1-3: Build a RAG Application

**Project**: Document Q&A System

```python
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import DirectoryLoader

# Load documents
loader = DirectoryLoader("./docs", glob="**/*.md")
docs = loader.load()

# Split
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
splits = text_splitter.split_documents(docs)

# Create vector store
vectorstore = Chroma.from_documents(
    documents=splits,
    embedding=OpenAIEmbeddings()
)
retriever = vectorstore.as_retriever()

# Create RAG chain
template = """Answer the question based only on the following context:

{context}

Question: {question}

Answer:"""

prompt = ChatPromptTemplate.from_template(template)

def format_docs(docs):
    return "\\n\\n".join(doc.page_content for doc in docs)

rag_chain = (
    {
        "context": retriever | format_docs,
        "question": RunnablePassthrough()
    }
    | prompt
    | ChatOpenAI()
    | StrOutputParser()
)

# Use it
answer = rag_chain.invoke("What is LangChain?")
print(answer)
```

**Enhancements**:
1. Add source citations
2. Implement conversation memory
3. Add streaming responses
4. Create a web interface with Streamlit

---

### Day 4-5: Build an Agent with Custom Tools

**Project**: Research Assistant Agent

```python
from langchain_openai import ChatOpenAI
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool
import requests

@tool
def search_wikipedia(query: str) -> str:
    """Search Wikipedia for information."""
    url = f"https://en.wikipedia.org/api/rest_v1/page/summary/{query}"
    response = requests.get(url)
    if response.status_code == 200:
        data = response.json()
        return data.get("extract", "No information found")
    return "Error searching Wikipedia"

@tool
def calculate(expression: str) -> str:
    """Evaluate a mathematical expression."""
    try:
        result = eval(expression)
        return str(result)
    except Exception as e:
        return f"Error: {str(e)}"

@tool
def save_note(content: str) -> str:
    """Save a note to a file."""
    with open("notes.txt", "a") as f:
        f.write(content + "\\n")
    return "Note saved successfully"

# Create agent
tools = [search_wikipedia, calculate, save_note]
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a research assistant. Use tools to help answer questions."),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}"),
])

agent = create_tool_calling_agent(
    ChatOpenAI(model="gpt-4", temperature=0),
    tools,
    prompt
)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True
)

# Use it
result = agent_executor.invoke({
    "input": "Research the population of Tokyo and save it to notes"
})
```

**Enhancements**:
1. Add web scraping tool
2. Implement file reading/writing tools
3. Add database query tool
4. Create a chat interface

---

### Day 6-7: Production Considerations

**Topics to Cover**:
- Error handling and retries
- Rate limiting
- Caching responses
- Monitoring with LangSmith
- Cost optimization
- Security best practices

**Error Handling Example**:
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

# Add retry logic
model = ChatOpenAI().with_retry(
    stop_after_attempt=3,
    wait_exponential_jitter=True
)

# Add fallback models
primary_model = ChatOpenAI(model="gpt-4")
fallback_model = ChatOpenAI(model="gpt-3.5-turbo")

chain = (
    ChatPromptTemplate.from_template("Answer: {question}")
    | primary_model.with_fallbacks([fallback_model])
)

# Use it
try:
    result = chain.invoke({"question": "What is AI?"})
except Exception as e:
    print(f"Error: {e}")
```

**Production Checklist**:
- [ ] Implement proper error handling
- [ ] Add retry logic for transient failures
- [ ] Set up fallback models
- [ ] Configure rate limiting
- [ ] Enable caching for repeated queries
- [ ] Set up monitoring and alerting
- [ ] Implement cost tracking
- [ ] Secure API keys (use environment variables)
- [ ] Add input validation
- [ ] Implement output sanitization
- [ ] Set up logging
- [ ] Create health check endpoints

---

## Recommended Reading Order

1. **Start Here**: `/home/zhyndalf/vibeCoding/langchain-fork/README.md`
2. **Architecture**: `/home/zhyndalf/vibeCoding/langchain-fork/CLAUDE.md`
3. **Core Concepts**:
   - `libs/core/langchain_core/runnables/base.py`
   - `libs/core/langchain_core/prompts/`
   - `libs/core/langchain_core/language_models/`
4. **Partner Integrations**: `libs/partners/*/` (choose your provider)
5. **Standard Tests**: `libs/standard-tests/` (for usage patterns)

## Additional Resources

- **Official Docs**: https://docs.langchain.com/oss/python/langchain/overview
- **API Reference**: https://reference.langchain.com/python
- **LangChain Academy**: https://academy.langchain.com/
- **Community Forum**: https://forum.langchain.com

## Tips for Success

1. **Start Simple**: Begin with basic chains before building complex agents
2. **Read the Code**: The LangChain codebase is well-documented
3. **Use Type Hints**: They help catch errors early
4. **Test Incrementally**: Test each component before composing
5. **Monitor Costs**: LLM calls can be expensive
6. **Use Callbacks**: They're essential for debugging
7. **Leverage Streaming**: Better user experience
8. **Think in Chains**: Decompose problems into composable steps
9. **Experiment**: Try different models and parameters
10. **Join the Community**: Ask questions, share learnings

## Next Steps After 4 Weeks

- Explore LangGraph for advanced agent workflows
- Learn about LangSmith for production monitoring
- Study Deep Agents for complex multi-agent systems
- Contribute to LangChain open source
- Build and deploy a production application
- Explore specialized integrations (your domain)

## Common Pitfalls to Avoid

1. **Over-engineering**: Start simple, add complexity as needed
2. **Ignoring Costs**: Monitor token usage
3. **Skipping Error Handling**: LLMs can fail unpredictably
4. **Not Using Streaming**: Users expect real-time feedback
5. **Hardcoding Prompts**: Use templates for flexibility
6. **Forgetting Async**: Async improves performance
7. **Neglecting Testing**: Test with various inputs
8. **Ignoring Security**: Validate inputs, sanitize outputs
9. **Not Monitoring**: Use callbacks and LangSmith
10. **Reinventing Wheels**: Check if LangChain already has it

---

**Good luck on your LangChain journey!**
