# Guardrail Types in LLM Systems

Guardrails are categorized primarily by **where they intervene in the execution pipeline** and by **the specific domain risks they address**.

---

## 1. Architectural Taxonomy (By Pipeline Stage)


```
[Input Rails] ──▶ [Retrieval & Tool Rails] ──▶ [Dialog Rails] ──▶ [Output Rails]
```

* **Input Guardrails (Pre-Processing)**
  * **Role:** Evaluate user prompts before they reach the foundation model.
  * **Key Targets:** Direct prompt injection, jailbreaks, malicious payloads, system prompt extraction, PII ingestion, and off-topic requests.
  * **Enforcement:** Rejecting the request, redacting inputs, or sanitizing text.

* **Dialog & Orchestration Rails (State Control)**
  * **Role:** Govern conversational flow, multi-turn states, and intent routing.
  * **Key Targets:** Conversational drift, adherence to business processes, mandatory disclaimers, and topic boundaries.
  * **Enforcement:** Directing to canned responses, state-machine transitions, or deterministic branch execution.

* **Retrieval & Tool Guardrails (Context & Execution)**
  * **Role:** Secure intermediate operations like RAG retrieval and API calls.
  * **Key Targets:** Indirect prompt injection within retrieved chunks, data leakage from unauthorized source documents, and invalid tool arguments.
  * **Enforcement:** Filtering context chunks, privilege boundaries, and function schema validation.

* **Output Guardrails (Post-Processing)**
  * **Role:** Inspect model-generated responses before delivering them to the client.
  * **Key Targets:** Hallucinations, factual inconsistency, PII leakage, toxic tone, and brand reputation risks.
  * **Enforcement:** Blocking responses, falling back to safe defaults, or masking sensitive terms.

---

## 2. Functional Taxonomy (By Target Risk)

| Guardrail Domain | Primary Threats | Typical Enforcement Techniques |
| :--- | :--- | :--- |
| **Security & Adversarial** | Direct/indirect prompt injections, jailbreaks, data exfiltration | Semantic embeddings, classifier models (e.g., Llama Guard), heuristic pattern matching |
| **Topical & Scope** | Out-of-domain questions, competitors, politics, casual chit-chat | Vector similarity to banned intent clusters, zero-shot classification, rule engines |
| **Privacy & Compliance** | SSNs, API tokens, passwords, PHI, financial records | Regex, Named Entity Recognition (NER), Microsoft Presidio, automated token masking |
| **Factuality & Grounding** | Hallucinations, fabricated citations, unsupported claims | Natural Language Inference (NLI), claim extraction, reference context self-checking |
| **Safety & Moderation** | Toxicity, harassment, hate speech, self-harm, sexual content | Multi-label classification models, OpenAI Moderation API, toxicity scoring |
| **Structural & Schema** | Invalid JSON, missing parameters, malformed tool arguments | Constrained decoding (e.g., Outlines, Guidance), Pydantic parsing |

---

## 3. Implementation Approaches

* **Deterministic / Rules:** Fast, cheap, and pattern-based (e.g., Regex, string matching, schema validation).
* **Classifier & Embedding-Based:** Specialized, low-latency models evaluating embeddings or class labels (e.g., DeBERTa, Llama Guard).
* **LLM-as-a-Judge:** Secondary LLM passes running verification prompts (e.g., self-checking for hallucination or topical drift).
* **Constrained Decoding:** Logit biasing at inference time to guarantee strict adherence to context-free grammars or schemas.

```
