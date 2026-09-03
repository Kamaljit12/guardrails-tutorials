## NeMo Guardrails

NeMo Guardrails is an open-source framework developed by NVIDIA that acts as a programmable control and safety layer between user applications and Large Language Models (LLMs).

Rather than relying purely on prompt engineering to enforce rules, NeMo Guardrails intercepts inputs, intermediate states, and outputs to ensure models remain safe, accurate, on-topic, and aligned with predefined business logic.

## How It Works

NeMo Guardrails processes every interaction through a multi-stage pipeline using Colang (a domain-specific modeling language for conversation flows) paired with standard configuration files (config.yml).

It evaluates interactions across five distinct rails:
```text
User Query ──▶ [Input Rails] ──▶ [Dialog Rails] ──▶ [Retrieval / Execution Rails] ──▶ LLM ──▶ [Output Rails] ──▶ User
```

1. **Input Rails:** Inspect the user's prompt before it ever reaches the core model. They detect `prompt injection`, `jailbreak attempts`, `toxic language`, and `off-topic questions`, modifying or rejecting the request if a rule is violated.

2. **Dialog Rails:** Guide conversational flow using state-machine-like logic written in `Colang`. They map user utterances to canonical intents and determine whether the bot should invoke a **tool**, execute custom `logic`, or `steer` the conversation in a specific direction.

3. **Retrieval Rails:** Used in Retrieval-Augmented Generation (RAG) pipelines to `filter`, `sanitize`, or `re-rank retrieved knowledge chunks`, preventing `data leakage` or `untrusted context injection`.

4. **Execution Rails:** Intercept calls to custom `tools`, `APIs`, or `Python actions` to validate that parameters passed to and returned from tools **meet security constraints**.

5. **Output Rails:** Inspect the `generated text` before returning it to the user. They perform `hallucination` and `fact-checking` checks against retrieved documents, `scrub` **Personally Identifiable Information* `(PII)`, and block harmful content.

## Key Components
- **Colang (.co files):** A lightweight `modeling language` used to `declare canonical user intents`, bot intents, and interaction flows.  
- **Actions (actions.py):** Custom Python code `invoked within flows` to call external databases, `verification APIs` (e.g., Presidio for PII scrubbing), or enterprise endpoints.
- **Config (config.yml):** Specifies the underlying LLM provider (OpenAI, Anthropic, Hugging Face, local models via vLLM), `active rails`, and moderation models.

## Minimal Example

1. Define the dialogue flow in Colang `(config/main.co):`
```code
define user ask off_topic
  "Can you write a poem about flowers?"
  "What is the capital of France?"

define flow off_topic
  user ask off_topic
  bot refuse off_topic

define bot refuse off_topic
  "I am a financial assistant and can only help with your banking inquiries."
```

2. Run in Python:

```python
from nemoguardrails import LLMRails, RailsConfig

config = RailsConfig.from_path("./config")
rails = LLMRails(config)

response = rails.generate(messages=[{
    "role": "user",
    "content": "Can you write a poem about flowers?"
}])

print(response["content"])
# Output: "I am a financial assistant and can only help with your banking inquiries."
```

## When to Use NeMo Guardrails
- Domain-Specific Enterprise Bots: When your assistant should only answer questions within a strict domain (e.g., banking, insurance, internal HR) and reject chit-chat or competitors' inquiries.
- Production RAG Systems: When you need automated fact-checking and hallucination reduction by cross-referencing model assertions directly against source documents.
- Compliance & Data Privacy: When you must ensure sensitive information (passwords, credit card numbers, SSNs) never leaves the server or reaches user chat logs.
- Adversarial Security: When your application requires a dedicated defense layer against jailbreaks, prompt leaking, and system prompt overrides.

## When Not to Use It
- Open-Ended Creative Writing: If your app relies on broad, unrestricted generation across random topics, rigid dialog rails will hinder performance.
- Extreme Latency Constraints: Running verification checks and self-check calls adds token overhead and round-trip latency to each turn. Simple apps may be better served by standard regex or lightweight classification APIs.
