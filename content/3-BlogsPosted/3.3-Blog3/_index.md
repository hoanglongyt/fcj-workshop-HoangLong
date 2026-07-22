---
title: "Blog 3"
date: 2026-06-30
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# AMAZON BEDROCK GUARDRAILS - ENTERPRISE SAFETY AND PRIVACY FOR GENERATIVE AI APPLICATIONS

Amazon Bedrock Guardrails provides multi-layered safety controls for Generative AI applications, enabling enterprises to implement customized safeguards aligned with organizational security policies and compliance requirements.

### Key points to know:

* **Content & Denied Topic Filtering**: Automatically detects and blocks prompts or responses containing hate speech, sensitive content, or restricted business topics.
* **PII Redaction & Privacy**: Identifies and masks Personally Identifiable Information (PII) such as emails, phone numbers, credit cards, or national IDs during AI interactions.
* **Prompt Injection & Jailbreak Protection**: Strengthens defenses against malicious prompt injection and jailbreaking attempts targeting Foundation Models.
* **Model-Agnostic Application**: Guardrails can be consistently applied across multiple Foundation Models in Amazon Bedrock without reconfiguring safety rules for each individual model.

---

### Facebook Post Link
- **Post Link**: [https://www.facebook.com/groups/awsstudygroupfcj/permalink/2220585502039743/#](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2220585502039743/#)

---

### Step-by-Step Guide: Deploying Amazon Bedrock Guardrails

#### 1. Create a Guardrail via AWS CLI
Define denied topics and sensitive PII masking policies:

```bash
aws bedrock create-guardrail \
  --name "enterprise-ai-guardrail" \
  --description "Guardrail for Enterprise Generative AI Application" \
  --topic-policy-config '{"topicsConfig": [{"name": "FinancialAdvice", "definition": "Providing direct investment advice", "examples": ["Which stock should I buy today?"], "type": "DENY"}]}' \
  --sensitive-information-policy-config '{"piiEntitiesConfig": [{"type": "EMAIL", "action": "ANONYMIZE"}, {"type": "PHONE", "action": "ANONYMIZE"}]}' \
  --blocked-input-messaging "Your request violates content safety policy." \
  --blocked-outputs-messaging "The response was blocked as it contained restricted content."
```

#### 2. Integrate Guardrail with Bedrock Model Invocations
When invoking Bedrock models via the Python Boto3 SDK, pass `guardrailIdentifier` and `guardrailVersion`:

```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

response = bedrock.invoke_model(
    modelId='anthropic.claude-3-sonnet-20240229-v1:0',
    guardrailIdentifier='<your-guardrail-id>',
    guardrailVersion='1',
    body=json.dumps({
        "prompt": "Please recommend the best stock to buy today.",
        "max_tokens": 300
    })
)
```

#### 3. Verify Filtered Responses
The Guardrail automatically intercepts violating inputs or outputs and returns pre-configured safety responses.