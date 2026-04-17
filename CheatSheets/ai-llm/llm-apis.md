# Building with LLM APIs: A Complete Progressive Tutorial

---

## 1. What & Why

LLM APIs let you integrate language model capabilities directly into your applications. Instead of building models yourself — an endeavor requiring millions of dollars in compute and months of engineering — you send HTTP requests to hosted endpoints maintained by Anthropic, OpenAI, Google, or other providers and receive generated text, structured data, tool call results, and embeddings in response.

Why use an API rather than a locally-hosted model? For most production use cases: frontier model quality far exceeds what you can run locally, APIs handle infrastructure and scaling, and the economics favor pay-per-token over running GPU servers for variable workloads. Local models make sense for privacy-sensitive data, extremely high volume, latency-critical applications, and offline scenarios.

This tutorial covers the patterns you need to build reliable, production-quality LLM integrations: managing conversation state, handling errors and retries, streaming responses, tool use, structured output, cost estimation, and evaluation.

---

## 2. Mental Model

```
Your application  →  API request  →  LLM provider  →  API response  →  Your app

The request contains:
  - model:          which model to use
  - messages:       the conversation so far (system + user + assistant turns)
  - parameters:     temperature, max_tokens, stop sequences, etc.

The response contains:
  - content:        the model's generated text (or structured data, or tool calls)
  - stop_reason:    why generation stopped (end_turn, max_tokens, tool_use)
  - usage:          token counts (for cost tracking)

CRITICAL: The model has NO MEMORY between API calls.
You must re-send the entire conversation history every time.

Cost = input_tokens × input_price + output_tokens × output_price
     (per 1 million tokens, varies by model and provider)
```

---

## 3. Progressive Examples

### Level 1: First API Calls — OpenAI and Anthropic

```python
# --- OpenAI ---
import os
from openai import OpenAI

# API key read from OPENAI_API_KEY environment variable automatically
client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a concise, technical assistant."},
        {"role": "user", "content": "What is the difference between TCP and UDP?"}
    ],
    temperature=0.3,      # lower = more deterministic
    max_tokens=512,       # cap output length
)

# Extract the response text
text = response.choices[0].message.content
print(text)

# Check why generation stopped
print(response.choices[0].finish_reason)   # "stop" = natural end, "length" = hit max_tokens

# Token usage (for cost tracking)
print(f"Input: {response.usage.prompt_tokens}")
print(f"Output: {response.usage.completion_tokens}")
print(f"Total: {response.usage.total_tokens}")

# --- Anthropic ---
import anthropic

client = anthropic.Anthropic()   # reads ANTHROPIC_API_KEY from environment

message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=512,
    system="You are a concise, technical assistant.",   # system prompt is separate in Anthropic's API
    messages=[
        {"role": "user", "content": "What is the difference between TCP and UDP?"}
    ],
    temperature=0.3,
)

# Extract text from content blocks
text = message.content[0].text
print(text)
print(f"Stop reason: {message.stop_reason}")   # "end_turn" or "max_tokens" or "tool_use"
print(f"Input: {message.usage.input_tokens}")
print(f"Output: {message.usage.output_tokens}")
```

```javascript
// Node.js — OpenAI
import OpenAI from "openai";
const client = new OpenAI();  // reads OPENAI_API_KEY from env

const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "system", content: "You are a concise, technical assistant." },
    { role: "user", content: "What is the difference between TCP and UDP?" }
  ],
  temperature: 0.3,
  max_tokens: 512,
});

console.log(response.choices[0].message.content);
console.log(`Tokens: ${response.usage.total_tokens}`);

// Node.js — Anthropic
import Anthropic from "@anthropic-ai/sdk";
const anthropic = new Anthropic();

const msg = await anthropic.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 512,
  system: "You are a concise, technical assistant.",
  messages: [{ role: "user", content: "What is the difference between TCP and UDP?" }],
});

console.log(msg.content[0].text);
```

### Level 2: Multi-Turn Conversations and Memory Management

```python
from openai import OpenAI
from anthropic import Anthropic

# The model has NO memory between calls.
# You must maintain and re-send the full conversation history every time.

class ConversationOpenAI:
    def __init__(self, system_prompt: str, model: str = "gpt-4o"):
        self.client = OpenAI()
        self.model = model
        self.messages = [{"role": "system", "content": system_prompt}]

    def chat(self, user_message: str) -> str:
        self.messages.append({"role": "user", "content": user_message})

        response = self.client.chat.completions.create(
            model=self.model,
            messages=self.messages,
            max_tokens=1024,
        )

        assistant_reply = response.choices[0].message.content
        # Append assistant reply so the model sees it in the next turn
        self.messages.append({"role": "assistant", "content": assistant_reply})

        return assistant_reply

    def token_count(self) -> int:
        # Rough estimate: 4 chars ≈ 1 token
        return sum(len(m["content"]) // 4 for m in self.messages)

    def trim_history(self, keep_last_n: int = 10):
        """Remove old messages to stay within context limits."""
        system = self.messages[0]
        recent = self.messages[-(keep_last_n * 2):]  # keep N user+assistant pairs
        self.messages = [system] + recent


# Context window management — critical for long conversations
class ConversationWithTrimming:
    MAX_TOKENS = 100_000   # conservative limit for gpt-4o's 128k context

    def __init__(self, system: str):
        self.client = OpenAI()
        self.system = system
        self.history = []

    def _estimate_tokens(self, messages):
        return sum(len(str(m)) // 4 for m in messages)

    def chat(self, user_message: str) -> str:
        self.history.append({"role": "user", "content": user_message})

        # Trim history if approaching context limit
        messages = [{"role": "system", "content": self.system}] + self.history
        while self._estimate_tokens(messages) > self.MAX_TOKENS and len(self.history) > 2:
            # Remove oldest user+assistant pair (keep at least one)
            self.history.pop(0)
            if self.history:
                self.history.pop(0)
            messages = [{"role": "system", "content": self.system}] + self.history

        response = self.client.chat.completions.create(
            model="gpt-4o", messages=messages, max_tokens=1024
        )
        reply = response.choices[0].message.content
        self.history.append({"role": "assistant", "content": reply})
        return reply
```

### Level 3: Streaming Responses

```python
# Streaming: receive tokens as they're generated instead of waiting for the full response.
# Critical for good UX — users see output immediately.

from openai import OpenAI
import anthropic

# --- OpenAI streaming ---
client = OpenAI()

def stream_openai(prompt: str):
    with client.chat.completions.stream(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}],
        max_tokens=1024,
    ) as stream:
        for chunk in stream:
            delta = chunk.choices[0].delta
            if delta.content:
                print(delta.content, end="", flush=True)
    print()  # newline at end

# Collect the full text while streaming
def stream_and_collect(prompt: str) -> str:
    full_text = []
    with client.chat.completions.stream(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}],
    ) as stream:
        for text in stream.text_stream:  # convenience iterator
            print(text, end="", flush=True)
            full_text.append(text)
    print()
    return "".join(full_text)

# --- Anthropic streaming ---
anthropic_client = anthropic.Anthropic()

def stream_anthropic(prompt: str) -> str:
    full_text = []
    with anthropic_client.messages.stream(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}],
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
            full_text.append(text)
    print()
    return "".join(full_text)

# --- FastAPI streaming endpoint ---
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

@app.post("/chat")
async def chat_endpoint(prompt: str):
    def generate():
        with client.chat.completions.stream(
            model="gpt-4o",
            messages=[{"role": "user", "content": prompt}],
        ) as stream:
            for text in stream.text_stream:
                yield f"data: {text}\n\n"   # Server-Sent Events format
        yield "data: [DONE]\n\n"

    return StreamingResponse(generate(), media_type="text/event-stream")
```

### Level 4: Tool Use (Function Calling)

```python
# Tool use: the model can decide to call your functions when it needs external data.
# Pattern: send tools definition → model returns tool_call → you execute → send result back

import json
from openai import OpenAI

client = OpenAI()

# Define the tools the model can call
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "City name, e.g. 'Cairo' or 'London, UK'"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "description": "Temperature unit"
                    }
                },
                "required": ["city"]
            }
        }
    }
]

# Your actual function implementations
def get_weather(city: str, unit: str = "celsius") -> dict:
    """Replace with a real weather API call."""
    return {"city": city, "temperature": 22, "unit": unit, "condition": "sunny"}

def run_tool(name: str, arguments: dict):
    if name == "get_weather":
        return get_weather(**arguments)
    raise ValueError(f"Unknown tool: {name}")

# Agentic loop: runs until the model stops requesting tools
def run_with_tools(user_message: str) -> str:
    messages = [{"role": "user", "content": user_message}]

    while True:
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=tools,
            tool_choice="auto",   # model decides when to use tools
        )

        choice = response.choices[0]
        messages.append(choice.message)   # append full message object (includes tool_calls)

        if choice.finish_reason == "stop":
            # Model is done — return the final text response
            return choice.message.content

        if choice.finish_reason == "tool_calls":
            # Model wants to call tools — execute each one
            for tool_call in choice.message.tool_calls:
                function_name = tool_call.function.name
                arguments = json.loads(tool_call.function.arguments)

                print(f"Calling tool: {function_name}({arguments})")
                result = run_tool(function_name, arguments)

                # Send tool result back to the model
                messages.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": json.dumps(result),
                })
            # Loop continues: model will process the tool results and respond

response = run_with_tools("What's the weather like in Cairo and London?")
print(response)
```

### Level 5: Structured Output and Error Handling

```python
import json
import time
import random
from typing import TypeVar, Type
from pydantic import BaseModel, ValidationError
from openai import OpenAI, RateLimitError, APITimeoutError, APIConnectionError

client = OpenAI()

# --- Structured output with Pydantic ---
class ExtractedContact(BaseModel):
    name: str | None = None
    email: str | None = None
    company: str | None = None
    phone: str | None = None

def extract_contact(text: str) -> ExtractedContact:
    """Extract contact information from unstructured text."""
    response = client.chat.completions.create(
        model="gpt-4o",
        response_format={"type": "json_object"},  # force JSON output
        messages=[
            {
                "role": "system",
                "content": """Extract contact information from the text.
Return ONLY a JSON object with these exact keys: name, email, company, phone.
Use null for any field not mentioned.
Example: {"name": "Alice", "email": "alice@co.com", "company": "Acme", "phone": null}"""
            },
            {"role": "user", "content": text}
        ],
        temperature=0,
    )

    try:
        data = json.loads(response.choices[0].message.content)
        return ExtractedContact(**data)
    except (json.JSONDecodeError, ValidationError) as e:
        raise ValueError(f"Failed to parse structured output: {e}")

# --- Robust retry with exponential backoff ---
def call_with_retry(
    func,
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
):
    """Retry an API call with exponential backoff on transient errors."""
    for attempt in range(max_retries):
        try:
            return func()
        except RateLimitError as e:
            if attempt == max_retries - 1:
                raise
            # Respect retry-after header if present
            retry_after = getattr(e, "retry_after", None)
            delay = retry_after or min(base_delay * (2 ** attempt) + random.uniform(0, 1), max_delay)
            print(f"Rate limited. Retrying in {delay:.1f}s (attempt {attempt + 1}/{max_retries})")
            time.sleep(delay)
        except (APITimeoutError, APIConnectionError) as e:
            if attempt == max_retries - 1:
                raise
            delay = min(base_delay * (2 ** attempt), max_delay)
            print(f"Connection error: {e}. Retrying in {delay:.1f}s")
            time.sleep(delay)

# Usage
result = call_with_retry(
    lambda: client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": "Hello"}],
    )
)

# --- Cost tracking ---
PRICING = {  # Per 1M tokens (approximate, check provider for latest)
    "gpt-4o":                {"input": 2.50,  "output": 10.00},
    "gpt-4o-mini":           {"input": 0.15,  "output": 0.60},
    "claude-sonnet-4-20250514": {"input": 3.00,  "output": 15.00},
    "claude-haiku-4-5-20251001": {"input": 0.80,  "output": 4.00},
}

class CostTracker:
    def __init__(self):
        self.total_input_tokens = 0
        self.total_output_tokens = 0
        self.requests = 0

    def record(self, model: str, usage):
        self.total_input_tokens += usage.prompt_tokens
        self.total_output_tokens += usage.completion_tokens
        self.requests += 1

    def estimate_cost(self, model: str) -> float:
        if model not in PRICING:
            return 0
        rates = PRICING[model]
        return (self.total_input_tokens / 1_000_000 * rates["input"] +
                self.total_output_tokens / 1_000_000 * rates["output"])

    def report(self, model: str):
        print(f"Requests:  {self.requests}")
        print(f"Input tokens:  {self.total_input_tokens:,}")
        print(f"Output tokens: {self.total_output_tokens:,}")
        print(f"Estimated cost: ${self.estimate_cost(model):.4f}")
```

### Level 6: Production Patterns — Caching, Logging, Evaluation

```python
import hashlib
import json
import sqlite3
from functools import wraps
from openai import OpenAI

client = OpenAI()

# --- Semantic caching ---
class LLMCache:
    """Cache LLM responses in SQLite to avoid redundant API calls."""

    def __init__(self, db_path: str = "llm_cache.db"):
        self.conn = sqlite3.connect(db_path, check_same_thread=False)
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS cache (
                key TEXT PRIMARY KEY,
                response TEXT,
                created_at REAL,
                model TEXT
            )
        """)
        self.conn.commit()

    def _key(self, messages: list, model: str, **params) -> str:
        payload = json.dumps({"messages": messages, "model": model, **params}, sort_keys=True)
        return hashlib.sha256(payload.encode()).hexdigest()

    def get(self, messages, model, **params):
        key = self._key(messages, model, **params)
        row = self.conn.execute("SELECT response FROM cache WHERE key = ?", (key,)).fetchone()
        return json.loads(row[0]) if row else None

    def set(self, messages, model, response, **params):
        key = self._key(messages, model, **params)
        self.conn.execute(
            "INSERT OR REPLACE INTO cache VALUES (?, ?, ?, ?)",
            (key, json.dumps(response), __import__("time").time(), model)
        )
        self.conn.commit()

cache = LLMCache()

def cached_completion(model: str, messages: list, **kwargs):
    cached = cache.get(messages, model, **kwargs)
    if cached:
        print("[CACHE HIT]")
        return cached

    response = client.chat.completions.create(
        model=model, messages=messages, **kwargs
    )
    response_dict = response.model_dump()
    cache.set(messages, model, response_dict, **kwargs)
    return response_dict

# --- Evaluation harness ---
class LLMEval:
    """Simple evaluation harness for LLM outputs."""

    def __init__(self, model: str = "gpt-4o"):
        self.model = model
        self.client = OpenAI()

    def llm_judge(self, question: str, answer: str, rubric: str) -> dict:
        """Use a powerful model to score a weaker model's output."""
        response = self.client.chat.completions.create(
            model="gpt-4o",
            temperature=0,
            messages=[{
                "role": "user",
                "content": f"""Evaluate the following answer.
Question: {question}
Answer: {answer}
Rubric: {rubric}

Respond with JSON: {{"score": 1-5, "reasoning": "brief explanation", "pass": true/false}}
Score 1 = completely wrong, 5 = perfect."""
            }],
            response_format={"type": "json_object"},
        )
        return json.loads(response.choices[0].message.content)

    def run_benchmark(self, test_cases: list[dict]) -> dict:
        """Run evaluation on a list of test cases."""
        results = []
        for case in test_cases:
            response = self.client.chat.completions.create(
                model=self.model,
                messages=[{"role": "user", "content": case["prompt"]}],
                temperature=0,
            )
            answer = response.choices[0].message.content
            score = self.llm_judge(case["prompt"], answer, case["rubric"])
            results.append({**case, "answer": answer, **score})

        avg_score = sum(r["score"] for r in results) / len(results)
        pass_rate = sum(1 for r in results if r["pass"]) / len(results)
        return {"avg_score": avg_score, "pass_rate": pass_rate, "results": results}

# Usage
eval_harness = LLMEval(model="gpt-4o-mini")   # evaluate a cheaper model
test_cases = [
    {
        "prompt": "What is 15% of 240?",
        "rubric": "Answer must be exactly 36 or show correct calculation."
    },
    {
        "prompt": "Sort this list: [3, 1, 4, 1, 5, 9, 2, 6]",
        "rubric": "Must return [1, 1, 2, 3, 4, 5, 6, 9] in some form."
    },
]
results = eval_harness.run_benchmark(test_cases)
print(f"Average score: {results['avg_score']:.1f}/5")
print(f"Pass rate: {results['pass_rate']:.0%}")
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Hardcoding API keys in source code**

```python
# WRONG: key in code → committed to git → leaked forever
client = OpenAI(api_key="sk-abc123")

# CORRECT: read from environment
import os
client = OpenAI()   # reads OPENAI_API_KEY automatically from environment
# Set in shell: export OPENAI_API_KEY="sk-..."
# Or use python-dotenv with a .gitignored .env file
```

**Mistake 2: Not handling `finish_reason == "length"`**

```python
# WRONG: assuming all output is complete
text = response.choices[0].message.content
process(text)   # text may be mid-sentence!

# CORRECT: check finish reason
choice = response.choices[0]
if choice.finish_reason == "length":
    # Output was truncated — increase max_tokens or handle partial output
    raise ValueError("Response truncated — increase max_tokens")
if choice.finish_reason == "content_filter":
    return "[Content filtered by safety system]"
text = choice.message.content
```

**Mistake 3: Sequential API calls when parallel is possible**

```python
import asyncio
from openai import AsyncOpenAI

# WRONG: sequential (slow — 3 API calls × latency each)
texts = []
for prompt in prompts:
    response = client.chat.completions.create(model="gpt-4o", messages=[...])
    texts.append(response.choices[0].message.content)

# CORRECT: parallel async calls
async_client = AsyncOpenAI()

async def call_async(prompt: str) -> str:
    response = await async_client.chat.completions.create(
        model="gpt-4o", messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content

async def batch_calls(prompts: list[str]) -> list[str]:
    return await asyncio.gather(*[call_async(p) for p in prompts])

texts = asyncio.run(batch_calls(prompts))   # ~same latency as one call
```

**Mistake 4: Not tracking costs in production**

At $10/million output tokens (GPT-4o), a pipeline that generates 500-token responses 10,000 times per day costs $50/day — $18,250/year. This catches teams by surprise. Track token usage from day one and alert on anomalies.

---

## 5. The "Why Does This Work" Layer

### Why You Must Re-send the Full Conversation History

LLM APIs are stateless: the model has no persistent memory between calls. Each call is completely independent. To maintain a conversation, you must include all prior turns in the `messages` array — the model "sees" everything you send and generates a continuation.

This design has implications: the API has no concept of a session. Every API call costs tokens for both the new user message and all prior conversation history. For long conversations, history management (trimming, summarizing) becomes essential to control costs and stay within context limits.

### Why Context Windows Limit What You Can Include

A language model processes the entire `messages` array through its attention mechanism simultaneously. The attention matrix is O(n²) in sequence length — doubling the context quadruples the computation. Modern architectures use optimizations (Flash Attention, sliding windows), but context length still fundamentally limits what you can include in a single call.

Practical implication: you cannot stuff an entire document and a long conversation into the prompt without hitting limits or dramatically increasing cost and latency. Retrieval-Augmented Generation (RAG) solves this by fetching only the relevant passages.

---

## 6. Quick Reference

### Response Extraction

```python
# OpenAI
text = response.choices[0].message.content
finish = response.choices[0].finish_reason   # "stop", "length", "tool_calls"
tokens_in = response.usage.prompt_tokens
tokens_out = response.usage.completion_tokens

# Anthropic
text = message.content[0].text
stop = message.stop_reason   # "end_turn", "max_tokens", "tool_use"
tokens_in = message.usage.input_tokens
tokens_out = message.usage.output_tokens
```

### Key Parameters

| Parameter | Purpose | Typical Range |
|-----------|---------|---------------|
| `temperature` | Randomness | 0 (deterministic) to 1 (creative) |
| `max_tokens` | Output length cap | 256–4096 typical |
| `top_p` | Nucleus sampling | 0.9–1.0 |
| `stop` | Stop sequences | `["\n", "END"]` |
| `response_format` | Force JSON | `{"type": "json_object"}` |

### Cost Reference (Approximate, check provider for current prices)

| Model | Input $/1M | Output $/1M |
|-------|-----------|------------|
| GPT-4o | $2.50 | $10.00 |
| GPT-4o-mini | $0.15 | $0.60 |
| Claude Sonnet | $3.00 | $15.00 |
| Claude Haiku | $0.80 | $4.00 |

### Production Checklist

```
□ API keys in environment variables, never in code
□ Retry with exponential backoff for rate limits and timeouts
□ Check finish_reason before using output
□ Track token usage and cost per call
□ Cache deterministic calls (temperature=0)
□ Use streaming for interactive UIs
□ Set appropriate max_tokens (not too low = truncation, not too high = waste)
□ Validate structured output with Pydantic
□ Run evaluation benchmarks before model upgrades
```
