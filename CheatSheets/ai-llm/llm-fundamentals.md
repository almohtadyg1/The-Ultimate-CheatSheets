# LLM Fundamentals: A Complete Progressive Tutorial

---

## 1. What & Why

A Large Language Model (LLM) is a neural network trained on massive amounts of text to predict the next token — a word fragment — given the tokens that preceded it. From this deceptively simple objective, trained on trillions of tokens from the internet, books, and code, these models develop the ability to answer questions, write code, summarize documents, translate languages, and reason through complex problems.

Why understand LLMs at a technical level rather than just using them? Because the mechanics explain the behavior. Why does the same prompt sometimes give different answers? Because output is sampled probabilistically. Why does "hallucination" happen? Because the model is completing text patterns, not retrieving facts. Why does providing more context improve accuracy? Because the model has access to that context when generating each token. Understanding these fundamentals makes you a better engineer of systems that use LLMs.

---

## 2. Mental Model

Think of an LLM as an extremely sophisticated autocomplete engine.

```
Input (prompt):   "The capital of France is"
                           ↓
                    Model processes all
                    preceding tokens
                           ↓
              Probability distribution over
              all possible next tokens:
              
              "Paris"       → 89.3%
              "Paris,"      → 4.1%
              "Lyon"        → 0.8%
              "a"           → 0.6%
              ...thousands more

                           ↓
              Sample from distribution → "Paris"
              
Output so far: "The capital of France is Paris"
              Repeat for next token, and next...
```

The model doesn't "know" facts — it has learned statistical patterns from its training data that make certain continuations far more probable than others. "Hallucination" is what happens when the model generates a statistically plausible continuation that isn't factually grounded.

---

## 3. Progressive Examples

### Level 1: Tokens — The Fundamental Unit

Everything the model processes is tokens. Understanding tokenization explains many otherwise mysterious behaviors.

```python
# Install: pip install tiktoken (OpenAI's tokenizer library)
import tiktoken

enc = tiktoken.encoding_for_model("gpt-4")

def show_tokens(text):
    tokens = enc.encode(text)
    pieces = [enc.decode([t]) for t in tokens]
    print(f"Text:   {repr(text)}")
    print(f"Tokens: {pieces}")
    print(f"Count:  {len(tokens)}\n")

show_tokens("Hello, world!")
# Text:   'Hello, world!'
# Tokens: ['Hello', ',', ' world', '!']
# Count:  4

show_tokens("unbelievable")
# Text:   'unbelievable'
# Tokens: ['un', 'bel', 'iev', 'able']
# Count:  4

show_tokens("1234567890")
# Text:   '1234567890'
# Tokens: ['123', '456', '789', '0']
# Count:  4

# This explains why LLMs struggle with character counting:
show_tokens("How many r's in strawberry?")
# "strawberry" might be: ['str', 'awberry'] — the model never sees individual chars!

# Token counting for cost estimation
def estimate_cost(text, price_per_1k_tokens=0.01):
    n_tokens = len(enc.encode(text))
    cost = (n_tokens / 1000) * price_per_1k_tokens
    return n_tokens, cost

# Rule of thumb conversions:
# ~4 characters = 1 token (English text)
# ~3/4 of a word = 1 token
# 1 page (~500 words) ≈ 650 tokens
# 1 novel (~100k words) ≈ 130k tokens
```

### Level 2: The Context Window and Attention

```python
# The context window is the maximum number of tokens the model can "see" at once.
# This includes: system prompt + conversation history + your message + model's response.

# Context window sizes (as of 2024-2025):
context_windows = {
    "GPT-3.5":           4_096,    # ~3 pages
    "GPT-4":           128_000,    # ~96 pages
    "GPT-4o":          128_000,
    "Claude 3.5 Sonnet": 200_000,  # ~150 pages
    "Claude 3 Opus":   200_000,
    "Gemini 1.5 Pro": 1_000_000,   # ~750 pages
    "Llama 3.1 70B":   128_000,
}

# What goes in the context window:
def estimate_context_usage(system_prompt, conversation, user_message, expected_response):
    enc = tiktoken.encoding_for_model("gpt-4")
    
    system_tokens   = len(enc.encode(system_prompt))
    conv_tokens     = sum(len(enc.encode(m["content"])) for m in conversation)
    message_tokens  = len(enc.encode(user_message))
    response_tokens = len(enc.encode(expected_response))  # estimate
    
    total = system_tokens + conv_tokens + message_tokens + response_tokens
    
    print(f"System prompt:  {system_tokens:,} tokens")
    print(f"Conversation:   {conv_tokens:,} tokens")
    print(f"User message:   {message_tokens:,} tokens")
    print(f"Expected output:{response_tokens:,} tokens")
    print(f"Total:          {total:,} tokens")
    return total

# The "lost in the middle" problem:
# Models process context through attention mechanisms.
# Information at the BEGINNING and END of context gets more "attention" 
# than information in the middle.
# For long contexts, put critical instructions at the start AND end.

# Practical implication: RAG (Retrieval-Augmented Generation)
# Instead of putting an entire document in context, retrieve only relevant chunks.
# This improves accuracy and reduces cost.
```

### Level 3: Temperature and Sampling Parameters

```python
# Temperature controls how "creative" (random) the model's output is.
# 
# High temperature → more random, creative, unpredictable
# Low temperature  → more deterministic, focused, repetitive
# Temperature = 0  → always picks the most probable token (near-deterministic)

import openai

def compare_temperatures(prompt, temperatures=[0.0, 0.5, 1.0, 2.0]):
    client = openai.OpenAI()
    for temp in temperatures:
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": prompt}],
            temperature=temp,
            max_tokens=50,
        )
        print(f"Temperature {temp}: {response.choices[0].message.content}")

# Temperature recommendations by task:
temp_guide = {
    "Factual Q&A":            0.0,  # deterministic, no creativity needed
    "Code generation":        0.0,  # one correct answer
    "Data extraction":        0.0,  # structured, predictable
    "Summarization":          0.3,  # slight variation acceptable
    "Customer service":       0.5,  # natural but consistent
    "Essay writing":          0.7,  # some creativity
    "Creative brainstorming": 1.0,  # diversity is the goal
    "Poetry/fiction":         1.2,  # high creativity
}

# Other key parameters:
params_explained = {
    "temperature":   "Controls randomness (0 = deterministic, 2 = very random)",
    "max_tokens":    "Maximum tokens to generate (affects cost and length)",
    "top_p":         "Nucleus sampling: only sample from top p probability mass (alternative to temperature)",
    "top_k":         "Only sample from top k tokens (used in some APIs)",
    "stop":          "Stop generation when these sequences appear",
    "n":             "Number of completions to generate",
    "seed":          "For reproducible outputs (when supported)",
    "presence_penalty":   "Penalize tokens already in the output (reduces repetition)",
    "frequency_penalty":  "Penalize tokens proportional to how often they've appeared",
}

# Using stop sequences for structured output:
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "List 3 items:"}],
    stop=["4."],   # stop after the third item is complete
    max_tokens=100,
)
```

### Level 4: Training Pipeline — Pre-training, Fine-tuning, RLHF

```
THE THREE TRAINING STAGES:

1. PRE-TRAINING
   ─────────────────────────────────────────────────────────
   Task:       Predict the next token on a massive corpus
   Data:       Internet text, books, code, academic papers
   Scale:      Trillions of tokens, months of compute
   Result:     Model learns language patterns, facts, reasoning
   Cost:       $50M-$100M+ for frontier models
   
   What the model learns:
   - Language structure and grammar
   - Factual knowledge encoded in training data
   - Code patterns and syntax
   - Reasoning patterns observed in text
   
   What it doesn't learn:
   - How to follow instructions
   - How to have helpful conversations
   - Safety behaviors
   - User intent

2. SUPERVISED FINE-TUNING (SFT)
   ─────────────────────────────────────────────────────────
   Task:       Train on curated (prompt, ideal response) pairs
   Data:       Human-written examples of good assistant behavior
   Scale:      Thousands to millions of examples
   Result:     Model that can follow instructions and have conversations
   
   Without SFT, the base model responds to "How do I make pasta?" by
   continuing the text as if it were training data, not a question.
   SFT teaches the model that it's an assistant that should answer.

3. REINFORCEMENT LEARNING FROM HUMAN FEEDBACK (RLHF)
   ─────────────────────────────────────────────────────────
   Process:
   a. Generate multiple responses to the same prompt
   b. Human raters rank the responses (A better than B)
   c. Train a reward model to predict human preferences
   d. Use RL (PPO algorithm) to optimize the LLM toward higher reward
   
   Result:     Model that is helpful, harmless, and honest
   Used by:    OpenAI (ChatGPT), Anthropic (Claude), Google (Gemini)
   
   Alternative: DPO (Direct Preference Optimization) — simpler, often better
   
   This is why the same base model (e.g., LLaMA) can behave very differently
   depending on which alignment training was applied.
```

### Level 5: Embeddings and Semantic Search

```python
# Embeddings: dense vector representations of text that capture semantic meaning
# Similar meanings → nearby vectors in the embedding space

import openai
import numpy as np

client = openai.OpenAI()

def get_embedding(text: str, model="text-embedding-3-small") -> list[float]:
    response = client.embeddings.create(input=text, model=model)
    return response.data[0].embedding

def cosine_similarity(a: list[float], b: list[float]) -> float:
    a, b = np.array(a), np.array(b)
    return float(np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b)))

# Embeddings capture semantic meaning, not just word overlap
phrases = [
    "The dog ran quickly",
    "The canine sprinted fast",   # semantically similar
    "I enjoy programming",         # different topic
    "Paris is in France",          # completely different
]

base = get_embedding(phrases[0])
for phrase in phrases[1:]:
    emb = get_embedding(phrase)
    sim = cosine_similarity(base, emb)
    print(f"Similarity to '{phrases[0]}':")
    print(f"  '{phrase}': {sim:.3f}")

# Output shows "The canine sprinted fast" has high similarity to 
# "The dog ran quickly" despite sharing no keywords.

# --- Semantic search (the core of RAG) ---
class SimpleVectorStore:
    def __init__(self):
        self.documents = []
        self.embeddings = []

    def add(self, text: str):
        self.documents.append(text)
        self.embeddings.append(get_embedding(text))

    def search(self, query: str, top_k: int = 3) -> list[str]:
        query_emb = get_embedding(query)
        similarities = [cosine_similarity(query_emb, e) for e in self.embeddings]
        ranked = sorted(enumerate(similarities), key=lambda x: -x[1])
        return [self.documents[i] for i, _ in ranked[:top_k]]

# Usage:
store = SimpleVectorStore()
store.add("Python was created by Guido van Rossum in 1991.")
store.add("JavaScript runs in web browsers and was created by Brendan Eich.")
store.add("Rust prioritizes memory safety without garbage collection.")
store.add("The Eiffel Tower was built in 1889 in Paris.")

results = store.search("Who invented Python?")
print(results[0])  # "Python was created by Guido van Rossum in 1991."
```

### Level 6: LLM Evaluation and Limitations

```python
# Evaluating LLM outputs — critical for production systems

# 1. Exact match (for structured output)
def eval_json_extraction(model_output: str, expected: dict) -> dict:
    import json
    try:
        parsed = json.loads(model_output)
        correct = sum(parsed.get(k) == v for k, v in expected.items())
        return {"valid_json": True, "field_accuracy": correct / len(expected)}
    except json.JSONDecodeError:
        return {"valid_json": False, "field_accuracy": 0.0}

# 2. LLM-as-judge (for open-ended output)
def llm_judge(question: str, model_response: str, criteria: str) -> int:
    """Use a stronger model to evaluate a weaker model's output."""
    judge_prompt = f"""
Score the following response on a scale of 1-5.
Criterion: {criteria}

Question: {question}
Response: {model_response}

Scoring guide:
1 = Completely wrong or irrelevant
2 = Partially correct but major issues
3 = Mostly correct with minor issues  
4 = Correct and clear
5 = Excellent — thorough, accurate, well-structured

Respond with only a number 1-5.
"""
    # Call a capable model for evaluation
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": judge_prompt}],
        temperature=0,
        max_tokens=5,
    )
    return int(response.choices[0].message.content.strip())

# Known limitations to engineer around:

limitations = {
    "Hallucination": {
        "description": "Generates plausible but false information",
        "mitigation": "Use RAG with grounded prompts, include citations, verify critical facts"
    },
    "Lost in the middle": {
        "description": "Ignores information in the middle of long contexts",
        "mitigation": "Put critical info at start and end; use RAG to chunk and retrieve"
    },
    "Reasoning limitations": {
        "description": "Multi-step reasoning fails on complex problems",
        "mitigation": "Use chain-of-thought prompting, break into subtasks, verify steps"
    },
    "Token counting / character tasks": {
        "description": "Can't reliably count letters in words (due to tokenization)",
        "mitigation": "Use code execution for counting tasks"
    },
    "Recency cutoff": {
        "description": "Training data has a cutoff date; doesn't know recent events",
        "mitigation": "Connect to search/retrieval for current information"
    },
    "Context inconsistency": {
        "description": "May contradict itself across a long conversation",
        "mitigation": "Summarize and reinject key facts at intervals"
    },
}
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Treating LLMs as search engines or databases**

LLMs don't retrieve information — they generate text based on statistical patterns. "What is the capital of France?" works because "Paris" is a highly probable completion of that prompt from training. But "What happened yesterday?" fails because the model has no real-world connection. LLMs generate plausible text, not retrieved facts. This is why Retrieval-Augmented Generation (RAG) exists.

**Mistake 2: Assuming temperature=0 gives perfectly deterministic output**

At temperature=0, the model still samples from the probability distribution (always picking the argmax/highest probability token), but floating-point arithmetic on large numbers has implementation variance. Different hardware, batching, and precision settings can produce different outputs even at temperature=0. For true reproducibility, use a seed parameter when the API supports it.

**Mistake 3: Assuming more parameters = better for your task**

A 7B parameter model fine-tuned on your specific domain often outperforms a 70B general model on that domain. Larger models cost more to run, have higher latency, and aren't always better for specialized tasks. Match model size to your actual requirements and measure empirically.

**Mistake 4: Ignoring the "prompt injection" attack surface**

If your system takes user input and places it into a prompt alongside system instructions, users can inject adversarial text that overrides your instructions. Always delimit user content from system instructions using XML tags or similar structural markers. Validate and sanitize critical outputs.

**Mistake 5: Over-relying on temperature for creative tasks**

Very high temperature (>1.5) often degrades coherence — the model starts generating random-looking text rather than creative text. For creative tasks, better approaches include: rich creative prompting, asking for multiple variations and selecting, or using techniques like best-of-N sampling.

---

## 5. The "Why Does This Work" Layer

### How Attention Enables In-Context Learning

The Transformer architecture at the core of every modern LLM uses "attention" — a mechanism that allows every token in the input to "look at" every other token and weight its influence based on learned relevance. This is what enables in-context learning (few-shot prompting): the model attends to your examples and uses them to condition its output.

Unlike traditional machine learning, the model weights don't update during inference. The "learning" from your examples is entirely in the computation happening in the attention layers as they process the full context. This is also why longer contexts are computationally expensive — attention scales quadratically with sequence length (though modern techniques like FlashAttention reduce the constant).

### Why RLHF Produces Helpful and Harmless Models

Pre-trained models optimize for "what text is plausible given the training data." Training data includes harmful, misleading, and unhelpful text. Without additional training, the model will generate such text when it's the statistically most likely completion.

RLHF rewires the model's reward signal from "predict training data" to "maximize human preference scores." Humans rate responses on helpfulness and harmlessness. The reward model learns to predict these ratings. The LLM is then optimized with PPO (Proximal Policy Optimization) to generate text the reward model scores highly.

The limitation: reward models can be "hacked" — the LLM learns to generate text that scores well on the reward model but not actually useful. This is Goodhart's Law applied to AI alignment.

---

## 6. Quick Reference

### Token Estimates

| Content | Tokens |
|---------|--------|
| 1 word (English avg) | ~1.3 |
| 1 sentence | ~15-20 |
| 1 paragraph | ~75-100 |
| 1 page (~500 words) | ~650 |
| 1 book chapter (~5k words) | ~6,500 |

### Key Parameters

| Parameter | Range | Effect |
|-----------|-------|--------|
| `temperature` | 0.0-2.0 | 0=deterministic, 1=normal, 2=chaotic |
| `top_p` | 0.0-1.0 | Limit sampling to top probability mass |
| `max_tokens` | 1-model_max | Cap output length |
| `frequency_penalty` | -2.0-2.0 | Reduce repetition of frequent tokens |
| `presence_penalty` | -2.0-2.0 | Reduce reuse of any token already used |

### Temperature by Task

| Task | Temperature |
|------|------------|
| Factual Q&A, code, data extraction | 0.0-0.2 |
| Summarization, classification | 0.2-0.5 |
| Conversation, explanation | 0.5-0.8 |
| Creative writing, brainstorming | 0.8-1.2 |

### Architectures You'll See

| Architecture | Key Feature | Example Models |
|-------------|------------|----------------|
| Decoder-only Transformer | Autoregressive text generation | GPT, LLaMA, Claude, Gemini |
| Encoder-only | Text classification, embeddings | BERT, RoBERTa |
| Encoder-decoder | Translation, summarization | T5, BART |
| Mixture of Experts | Scale without proportional compute | Mixtral, GPT-4 (rumored) |
