# AI Guardrails & Security — Notes

## 1. Recommended Architecture

```text
User
  ↓
Presidio
  ↓
NeMo Guardrails
  ├── Input Rails
  │    ├── Prompt injection
  │    ├── Jailbreak
  │    └── Other input safety checks
  ↓
LLM
  ↓
NeMo Guardrails
  ├── Output Rails
  │    ├── Harmful content
  │    ├── Toxicity
  │    ├── Unsafe response
  │    └── Policy violations
  ↓
Guardrails AI (when needed)
  ├── Schema validation
  ├── JSON validation
  ├── Data/field validation
  └── Custom validators
  ↓
Final Response
  ↓
User
```

## 2. What Each Tool Does

### Presidio — PII Protection

**Main purpose:** Detect and protect personally identifiable information (PII).

Examples:
- Names
- Email addresses
- Phone numbers
- Addresses
- Credit card numbers
- Other sensitive personal information

Typical use:

```text
User Input
   ↓
Presidio
   ↓
Detect / Mask / Anonymize PII
   ↓
LLM
```

Presidio is mainly focused on **sensitive data protection**, not general AI safety.

---

## 3. NeMo Guardrails — AI Safety & Conversation Controls

**Main purpose:** Control what goes into and comes out of the LLM and enforce safety policies around the AI application.

### Input Rails

Can be used for checks such as:

- Prompt injection
- Jailbreak attempts
- Unsafe requests
- Topic restrictions
- Other input safety policies

### Output Rails

Can be used to check the generated LLM response before it reaches the user.

Examples:

- Harmful content
- Toxicity
- Unsafe responses
- Policy violations
- Other application-specific safety rules

Important:

> NeMo Guardrails is the preferred choice for the **overall AI safety layer**, but it does not automatically detect every type of harmful content simply by installing it. Appropriate rails, flows, classifiers/models, or other checks need to be configured for the required safety policy.

---

## 4. Guardrails AI — Output/Data Validation

**Main purpose:** Validate that an LLM-generated response meets a required structure, schema, or set of constraints.

Examples:

- Is the output valid JSON?
- Are required fields present?
- Are data types correct?
- Is a value within an allowed range?
- Does the output match a defined schema?
- Does it pass custom validators?

Example:

```text
LLM Response
     ↓
Guardrails AI
     ↓
Validate structure / schema / constraints
     ↓
Accept, reject, or handle invalid output
```

### Example

Expected:

```json
{
  "vehicle_status": "moving",
  "speed": 72
}
```

Guardrails AI can validate that:

- `vehicle_status` exists
- `speed` is a number
- `speed` is within an expected range
- The response follows the expected schema

---

## 5. NeMo Guardrails vs Guardrails AI

| Requirement | Recommended Tool |
|---|---|
| PII detection/masking | **Presidio** |
| Prompt injection | **NeMo Guardrails** |
| Jailbreak detection | **NeMo Guardrails** |
| Harmful/unsafe response | **NeMo Guardrails** |
| Toxicity/safety | **NeMo Guardrails** |
| Conversation safety | **NeMo Guardrails** |
| Tool/action safety | **NeMo Guardrails** |
| JSON/schema validation | **Guardrails AI** |
| Data/field validation | **Guardrails AI** |
| Custom output validators | **Guardrails AI** |

## 6. Which One Should Be Used for the Final LLM Response?

If the question is:

> "Is the final LLM response acceptable? Does it contain harmful or unsafe content?"

Use **NeMo Guardrails** as the primary safety layer.

```text
LLM
 ↓
NeMo Output Rails
 ↓
Is response safe?
 ├── YES → Send to user
 └── NO  → Block / modify / handle according to policy
```

If the question is:

> "Is the response in the correct format and does its data satisfy my rules?"

Use **Guardrails AI**.

```text
LLM
 ↓
Guardrails AI
 ↓
Is output structurally valid?
 ├── YES → Continue
 └── NO  → Reject / retry / handle invalid output
```

## 7. Recommended Combination

You do **not** need to use both tools everywhere.

A clean architecture is:

```text
                    USER
                      ↓
               ┌─────────────┐
               │  Presidio   │
               │ PII Safety  │
               └──────┬──────┘
                      ↓
          ┌──────────────────────┐
          │  NeMo Guardrails    │
          │     Input Rails     │
          │                      │
          │ • Prompt Injection   │
          │ • Jailbreak          │
          │ • Input Safety       │
          └──────────┬───────────┘
                     ↓
                    LLM
                     ↓
          ┌──────────────────────┐
          │  NeMo Guardrails    │
          │    Output Rails     │
          │                      │
          │ • Harmful Content    │
          │ • Toxicity           │
          │ • Unsafe Response    │
          └──────────┬───────────┘
                     ↓
          ┌──────────────────────┐
          │    Guardrails AI    │
          │  Optional Validation │
          │                      │
          │ • Schema             │
          │ • JSON               │
          │ • Data Constraints   │
          │ • Custom Validators  │
          └──────────┬───────────┘
                     ↓
                   USER
```

### Key takeaway

**Presidio → Protect sensitive personal data**

**NeMo Guardrails → Protect the AI interaction and enforce safety**

**Guardrails AI → Validate the structure and correctness of generated data/output**

For an Agentic AI application, start with **Presidio + NeMo Guardrails** as the core security/safety layer. Add **Guardrails AI** when you have specific structured-output or data-validation requirements.
