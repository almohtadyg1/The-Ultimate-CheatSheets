# LLM Orchestration Frameworks: A Complete Progressive Tutorial

---

## 1. What & Why

An LLM orchestration framework sits on top of provider APIs (OpenAI, Anthropic, etc.) and provides reusable building blocks for chaining prompts, managing conversation memory, connecting external tools and databases, and building autonomous agents. The two dominant frameworks are LangChain and LlamaIndex.

Why use a framework at all? Because production LLM applications are not single API calls — they are pipelines. Retrieve documents from a vector store, format them into a prompt, send to an LLM, parse the output, call a tool based on the result, send a second LLM call with the tool output. Without a framework, you write this plumbing manually for every application. With a framework, each step is a composable building block.

When NOT to use a framework: simple one-shot applications, when you need full control over the exact HTTP requests being made, when the framework abstractions are leaking and you're fighting them more than using them. For complex applications — agents, multi-step pipelines, RAG with complex retrieval — they save substantial engineering time.

**LangChain**: best for agents, chains, complex pipelines with many LLMs, tools, and data sources.
**LlamaIndex**: best for RAG and document Q&A — its retrieval and indexing capabilities are more powerful.

---

## 2. Mental Model

```
Without a framework:
  You write each connection manually:
  prompt_string → HTTP call → parse JSON → format for next step → HTTP call → ...

With LangChain LCEL (pipe operator):
  prompt | llm | output_parser | next_prompt | llm | ...
  Each | connects a Runnable — anything that takes input and returns output.

LangChain primitives:
  Models    → wrappers around LLM providers (ChatOpenAI, ChatAnthropic)
  Prompts   → templates with variables (ChatPromptTemplate)
  Parsers   → transform LLM output (StrOutputParser, PydanticOutputParser)
  Retrievers → fetch relevant documents (VectorStoreRetriever)
  Agents    → LLMs that decide which tool to call next (use LangGraph)

LlamaIndex primitives:
  Documents  → load from files, databases, APIs
  Nodes      → chunks of documents with metadata
  Index      → searchable structure over nodes (VectorStoreIndex)
  Retriever  → query the index, get relevant nodes
  Engine     → query engine or chat engine combining retriever + LLM
```

---

## 3. Progressive Examples

### Level 1: LangChain — Models, Prompts, and Basic Chains

```python
from langchain_openai import ChatOpenAI
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Initialize models — reads API keys from environment
openai_llm = ChatOpenAI(model="gpt-4o", temperature=0.7, max_tokens=1024)
anthropic_llm = ChatAnthropic(model="claude-sonnet-4-20250514", temperature=0.7)

# Direct invocation
response = openai_llm.invoke("What is the capital of Egypt?")
print(response.content)       # "Cairo"
print(response.usage_metadata)  # token counts

# Prompt templates with variables
translate_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a professional translator. Translate to {language}."),
    ("human", "{text}")
])

# LCEL (LangChain Expression Language): compose with |
parser = StrOutputParser()   # converts LLM response object to plain string

# Build a chain: prompt → model → parser
translate_chain = translate_prompt | openai_llm | parser

# Invoke
result = translate_chain.invoke({
    "language": "Arabic",
    "text": "Hello, how are you?"
})
print(result)   # "مرحباً، كيف حالك؟"

# Stream: receive tokens as they're generated
for chunk in translate_chain.stream({"language": "French", "text": "Good morning!"}):
    print(chunk, end="", flush=True)
print()

# Batch: run multiple inputs in parallel
results = translate_chain.batch([
    {"language": "Spanish", "text": "Hello"},
    {"language": "Japanese", "text": "Hello"},
    {"language": "German", "text": "Hello"},
])
# All three run concurrently, results returned in order
```

### Level 2: Structured Output and Multi-Step Chains

```python
from langchain_core.output_parsers import JsonOutputParser
from pydantic import BaseModel, Field
from langchain_core.runnables import RunnablePassthrough
from typing import List

# Pydantic model for typed, validated output
class ArticleSummary(BaseModel):
    title: str = Field(description="Article title (max 10 words)")
    key_points: List[str] = Field(description="3-5 most important points")
    sentiment: str = Field(description="positive, negative, or neutral")
    word_count_estimate: int = Field(description="Estimated words in original")

parser = JsonOutputParser(pydantic_object=ArticleSummary)

summary_prompt = ChatPromptTemplate.from_messages([
    ("system", "You extract structured information from text. {format_instructions}"),
    ("human", "Analyze this article:\n\n{article}")
]).partial(format_instructions=parser.get_format_instructions())

summary_chain = summary_prompt | openai_llm | parser

article = """
Python 3.12 was released with significant performance improvements,
including a 25% speed increase over Python 3.11. The new release
includes improved error messages, new typing features, and experimental
features for the upcoming free-threaded mode.
"""

summary = summary_chain.invoke({"article": article})
print(summary.title)        # "Python 3.12 Brings 25% Performance Boost"
print(summary.key_points)   # ["25% speed increase", "improved error messages", ...]
print(summary.sentiment)    # "positive"

# Multi-step chain: topic → outline → full post
llm = ChatOpenAI(model="gpt-4o")
str_parser = StrOutputParser()

outline_chain = (
    ChatPromptTemplate.from_template("Write a 5-point outline for: {topic}")
    | llm
    | str_parser
)

expand_chain = (
    ChatPromptTemplate.from_template("Write a full post from this outline:\n{outline}")
    | llm
    | str_parser
)

# Chain them: pass output of first into second
full_chain = (
    outline_chain
    | (lambda outline: {"outline": outline})   # wrap in dict for next template
    | expand_chain
)

result = full_chain.invoke({"topic": "vector databases for beginners"})

# Parallel execution: run multiple LLM calls simultaneously
from langchain_core.runnables import RunnableParallel

analysis_chain = RunnableParallel(
    summary=ChatPromptTemplate.from_template("Summarize in 1 sentence: {text}") | llm | str_parser,
    keywords=ChatPromptTemplate.from_template("List 5 keywords from: {text}") | llm | str_parser,
    sentiment=ChatPromptTemplate.from_template("Sentiment of (one word): {text}") | llm | str_parser,
)

results = analysis_chain.invoke({"text": article})
print(results["summary"])    # one sentence
print(results["keywords"])   # comma-separated keywords
print(results["sentiment"])  # positive/negative/neutral
```

### Level 3: Memory and RAG Chains

```python
from langchain.memory import ConversationBufferMemory, ConversationSummaryMemory
from langchain_core.prompts import MessagesPlaceholder
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_core.runnables import RunnablePassthrough
from langchain_core.messages import HumanMessage, AIMessage

# --- Conversation Memory ---
# Buffer memory: keeps full conversation history
memory = ConversationBufferMemory(return_messages=True)

# Summary memory: LLM summarizes old turns to stay within context limits
summary_memory = ConversationSummaryMemory(
    llm=ChatOpenAI(model="gpt-4o-mini"),  # cheap model for summarization
    return_messages=True
)

# Chat chain with persistent memory
chat_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder(variable_name="history"),   # injected message list
    ("human", "{question}")
])

# Maintain history as a list
history = []

def chat_with_memory(question: str) -> str:
    messages = chat_prompt.format_messages(history=history, question=question)
    response = openai_llm.invoke(messages)
    # Update history
    history.append(HumanMessage(content=question))
    history.append(AIMessage(content=response.content))
    return response.content

print(chat_with_memory("My name is Alice and I live in Cairo."))
print(chat_with_memory("What city did I say I live in?"))   # remembers Cairo

# --- RAG Chain ---
# Build a vector store
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Chroma.from_texts(
    texts=[
        "Returns are accepted within 30 days of purchase.",
        "Shipping takes 3-5 business days.",
        "Premium members receive free shipping on all orders.",
        "Contact support@company.com for return assistance.",
    ],
    embedding=embeddings,
    metadatas=[{"source": "policy.txt"}] * 4,
)

retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

rag_prompt = ChatPromptTemplate.from_messages([
    ("system", """Answer the question using ONLY the following context.
If the answer is not in the context, say "I don't have that information."

Context:
{context}"""),
    ("human", "{question}")
])

def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | rag_prompt
    | openai_llm
    | str_parser
)

answer = rag_chain.invoke("Can I return something after 3 weeks?")
print(answer)   # "Yes, returns are accepted within 30 days..."
```

### Level 4: LangChain Agents with LangGraph

```python
# Agents: LLMs that decide which tool to call next, in a loop.
# LangGraph provides explicit control over the agent loop.

from langgraph.prebuilt import create_react_agent
from langchain_core.tools import tool
import json
import math

# Define tools
@tool
def search_docs(query: str) -> str:
    """Search the company knowledge base for information."""
    # Replace with actual vector search
    results = vectorstore.similarity_search(query, k=2)
    return "\n".join(doc.page_content for doc in results)

@tool
def calculate(expression: str) -> str:
    """Evaluate a mathematical expression. Example: '2 * 3.14 * 5'"""
    try:
        # Safe evaluation for math expressions only
        result = eval(expression, {"__builtins__": {}},
                     {"sqrt": math.sqrt, "pi": math.pi, "abs": abs})
        return str(result)
    except Exception as e:
        return f"Error: {e}"

@tool
def get_order_status(order_id: str) -> str:
    """Get the current status of an order by ID."""
    # Replace with actual database lookup
    statuses = {
        "ORD-001": "shipped — expected delivery Jan 20",
        "ORD-002": "processing — ships within 2 business days",
    }
    return statuses.get(order_id, f"Order {order_id} not found.")

# Create a ReAct agent (Reason + Act loop)
agent = create_react_agent(
    model=openai_llm,
    tools=[search_docs, calculate, get_order_status],
    state_modifier="You are a helpful customer service agent. Use the available tools to answer questions accurately.",
)

# Run the agent
result = agent.invoke({
    "messages": [{"role": "user", "content": "What is your return policy and what's the status of order ORD-001?"}]
})
print(result["messages"][-1].content)
# Agent will: call search_docs for return policy + call get_order_status for ORD-001, then synthesize both
```

### Level 5: LlamaIndex — Document Indexing and Query Engines

```python
# LlamaIndex excels at RAG with complex document structures
from llama_index.core import SimpleDirectoryReader, VectorStoreIndex, Settings
from llama_index.llms.openai import OpenAI
from llama_index.embeddings.openai import OpenAIEmbedding
from llama_index.core.node_parser import SentenceSplitter
from llama_index.core.retrievers import VectorIndexRetriever
from llama_index.core.query_engine import RetrieverQueryEngine

# Configure globally
Settings.llm = OpenAI(model="gpt-4o", temperature=0)
Settings.embed_model = OpenAIEmbedding(model="text-embedding-3-small")
Settings.node_parser = SentenceSplitter(chunk_size=512, chunk_overlap=64)

# Load documents from a directory (supports PDF, DOCX, TXT, HTML, etc.)
documents = SimpleDirectoryReader("./docs/").load_data()

# Build a searchable index (handles chunking + embedding automatically)
index = VectorStoreIndex.from_documents(documents)

# Simple query engine
query_engine = index.as_query_engine(similarity_top_k=3)
response = query_engine.query("What is the refund policy for digital products?")
print(response.response)      # synthesized answer
for node in response.source_nodes:
    print(f"Source: {node.metadata.get('file_name')} (score: {node.score:.3f})")

# Chat engine: maintains conversation context
chat_engine = index.as_chat_engine(
    chat_mode="condense_plus_context",  # rephrases follow-up questions using history
    verbose=True,
)

resp1 = chat_engine.chat("What is the return window?")
resp2 = chat_engine.chat("What about digital downloads?")  # uses context from resp1

# --- Advanced: Metadata filtering ---
from llama_index.core.vector_stores import MetadataFilter, MetadataFilters

# Build index with metadata
from llama_index.core.schema import TextNode

nodes = [
    TextNode(
        text="Returns accepted within 30 days.",
        metadata={"category": "returns", "product_type": "physical"}
    ),
    TextNode(
        text="Digital products are non-refundable.",
        metadata={"category": "returns", "product_type": "digital"}
    ),
]
index = VectorStoreIndex(nodes)

# Filter to only physical product policies
filters = MetadataFilters(filters=[
    MetadataFilter(key="product_type", value="physical")
])
filtered_engine = index.as_query_engine(filters=filters)
response = filtered_engine.query("Can I return this?")
# Only searches physical product nodes
```

### Level 6: When to Use Frameworks vs Raw APIs

```python
# Decision framework:

# USE A FRAMEWORK when:
# - You're building a multi-step pipeline (retrieve → generate → act)
# - You need conversation memory with history management
# - You're integrating multiple data sources (vector DB + SQL + APIs)
# - You're building an agent that uses tools
# - You want built-in observability (LangSmith, Arize Phoenix)

# USE RAW API CALLS when:
# - Simple, single-step LLM calls
# - You need precise control over every HTTP parameter
# - The framework abstractions are fighting you more than helping
# - You're optimizing for performance in a tight loop
# - You're building a custom abstraction layer

# HYBRID: raw calls for the core logic, framework for specific integrations

# Example: use raw API for the main LLM call, use framework only for the vector store
from openai import OpenAI
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings

# Framework just for the vector store integration
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Chroma.from_texts(texts=[...], embedding=embeddings)

# Raw API for everything else (more control)
client = OpenAI()

def answer_question(question: str) -> str:
    # Use framework for retrieval
    docs = vectorstore.similarity_search(question, k=3)
    context = "\n".join(d.page_content for d in docs)

    # Use raw API for generation (full control over the request)
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": f"Answer using this context:\n{context}"},
            {"role": "user", "content": question},
        ],
        temperature=0,
        max_tokens=512,
        timeout=30,
    )
    return response.choices[0].message.content
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Starting with a framework before understanding the fundamentals**

Building a LangChain agent before understanding how the raw API works is like learning to drive in a simulator. When something breaks (and it will), you won't know what layer the problem is in — the framework, the provider API, your prompt, or your data. Learn the raw API first, then use frameworks to eliminate boilerplate.

**Mistake 2: Misusing LCEL by over-chaining**

```python
# WRONG: chain is so long it's unreadable and hard to debug
result = (
    step1_prompt | llm | parser
    | step2_prompt | llm | parser
    | step3_prompt | llm | parser
    | formatter
    | validator
    | step4_prompt | llm | parser
)

# CORRECT: name each step, use functions for complex logic
outline = (step1_prompt | llm | parser).invoke({"topic": topic})
draft = (step2_prompt | llm | parser).invoke({"outline": outline})
validated = validate_draft(draft)   # plain Python function
final = (step3_prompt | llm | parser).invoke({"draft": validated})
```

**Mistake 3: Not setting `similarity_top_k` appropriately**

```python
# WRONG: default k=4 may be too many (noisy) or too few (missing key info)
query_engine = index.as_query_engine()

# CORRECT: tune based on your use case
# k=2-3 for precise factual Q&A (fewer but more relevant chunks)
# k=5-10 for broad research queries (cast wider net)
query_engine = index.as_query_engine(similarity_top_k=3, similarity_cutoff=0.7)
# similarity_cutoff: reject chunks below this relevance score
```

**Mistake 4: Using BufferMemory without trimming for long conversations**

```python
# WRONG: BufferMemory keeps ALL history — grows without bound
memory = ConversationBufferMemory(return_messages=True)
# After 100 turns: prompt has 10,000+ tokens — expensive and may hit context limit

# CORRECT: use windowed or summary memory
from langchain.memory import ConversationBufferWindowMemory
memory = ConversationBufferWindowMemory(k=10, return_messages=True)
# Only keeps last 10 exchange pairs
```

---

## 5. Quick Reference

### LangChain LCEL Patterns

```python
# Basic chain
chain = prompt | llm | parser

# With context from another source
chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt | llm | parser
)

# Parallel execution
chain = RunnableParallel(
    a=prompt_a | llm | parser,
    b=prompt_b | llm | parser,
)

# Branching
chain = RunnableBranch(
    (condition_fn, branch_chain_a),
    default_chain_b,
)
```

### Framework Decision Matrix

| Situation | Use |
|-----------|-----|
| Single LLM call | Raw API |
| RAG over documents | LlamaIndex or raw API + vector DB |
| Multi-step chain (3+ steps) | LangChain LCEL |
| Agent with tools | LangGraph |
| Document Q&A, complex retrieval | LlamaIndex |
| Prototype / learning | Raw API first |
| Production with observability | Add LangSmith |

### Common Runnables

| Class | Purpose |
|-------|---------|
| `ChatOpenAI`, `ChatAnthropic` | LLM wrappers |
| `ChatPromptTemplate` | Message templates with variables |
| `StrOutputParser` | LLM response → string |
| `JsonOutputParser` | LLM response → dict |
| `PydanticOutputParser` | LLM response → typed Pydantic model |
| `RunnablePassthrough` | Pass input through unchanged |
| `RunnableParallel` | Run multiple chains simultaneously |
| `RunnableBranch` | Conditional routing |
| `VectorStoreRetriever` | Query a vector database |
