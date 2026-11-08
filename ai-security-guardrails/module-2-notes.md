# Module 2 Notes — AI Security & Guardrails

## Overview

Module 1 showed *why* AI security matters; Module 2 gives the *what*: a definition, why LLM apps are uniquely hard to secure, and one method (threat modeling) that every later module uses. Source: 22 slides in `Module_2/module2_slides.pdf`.

**Recap of Module 1 — three apps, six attacks**

| Application | Attacks demonstrated |
| --- | --- |
| RAG application | 1. Unauthorized users accessing confidential documents · 2. Customer personal data leaked via RAG |
| Agentic application | 3. CEO's private data accessed via an injected harmful email · 4. Harmful email causing memory poisoning |
| Multimodal agent | 5. Agent responding to harmful text hidden inside an image · 6. Harmful instructions entering context via the web-fetch tool |

## 1. What is AI security?

AI security = protecting everything an LLM-based application **has** (assets) and everything it **can do** (capabilities) from unauthorized and malicious access.

| Type | Item | What it means |
| --- | --- | --- |
| Asset | User data | Personal data, chat history, uploaded files |
| Asset | Company documents | Knowledge-base content, with per-user access rules |
| Asset | Credentials + system prompt | API keys, tokens, hidden rules, business logic in the prompt |
| Capability | Memory | Long-term notes the agent reads in later sessions |
| Capability | Tools, scopes, permissions | What the agent is allowed to call, and with which rights |
| Capability | Resources behind tools | Mailboxes, databases, file systems, payment systems |

References on the slides: OWASP Top 10 for LLM Applications 2025 (assets) and OWASP Top 10 for Agentic Applications, Dec 2025 (capabilities).

## 2. Why study AI security separately?

Four things make LLM apps different from traditional software:

1. **One channel** — instructions and data travel in the same text. There is no parameterized query for prompts (unlike SQL).
2. **Not deterministic** — the same input can give different outputs, so a test that passes once proves little.
3. **Natural language** — an attacker can phrase one attack in unlimited forms, so a block list can't catch them all.
4. **Autonomy** — an agent picks tools and acts without a human check at each step.

## 3. Adversarial attacks

**Definition:** deliberate attempts by an attacker to manipulate an AI system's inputs, data, model, memory or connected tools so it serves the attacker's goal instead of its intended goal.

**In simple words:** someone purposely gives or plants something harmful so the AI leaks information, follows the wrong instruction, or takes an action it shouldn't. (Refs: NIST AI 100-2e2025; OWASP LLM01:2025 Prompt Injection.)

| Attack | How it works |
| --- | --- |
| Direct prompt injection / jailbreak | The user types instructions that override the rules |
| Indirect prompt injection | Instructions hide in documents, web pages, emails, images |
| System prompt extraction | The attacker makes the model reveal its hidden rules |
| Data exfiltration | Data leaves via a tool call, a link, or an image URL |
| Memory poisoning | The attacker writes instructions into long-term memory |
| Knowledge base poisoning | The attacker adds documents the retriever will return |

## 4. Case study: the banking RAG app

A RAG app built for a banking client leaks customer data because retrieval has no access check.

1. **Request** — a user asks: "Give me a list of all customers who applied for a loan but got rejected."
2. **Missing access check** — the app searches the knowledge base without checking which records this user may see.
3. **Unauthorized retrieval** — confidential customer records are returned and passed to the LLM as context.
4. **Personal data leak** — the LLM answers with private customer details the user should never see.

The task set by the slides: *you are asked to fix this.* The two tools below are how you analyze it.

## 5. Tool 1 — the Data Flow Diagram (DFD)

A DFD shows **how data moves** — not code, classes or servers. It has five elements (per Shostack's threat-model notation):

| Element | Symbol | LLM-app examples |
| --- | --- | --- |
| External entity | Rectangle | User, attacker, 3rd-party API |
| Process | Rounded box | LLM, orchestrator, retriever, tool |
| Data store | Two parallel lines | Vector DB, memory, logs |
| Data flow | Arrow | Prompt, chunks, tool call |
| Trust boundary | Dashed line | Where the trust level changes |

RAG app flow: User → Retriever ⇄ Vector DB → LLM → Output to the user.

## 6. Tool 2 — the capability lens

Components are not equal, so each is analyzed **only for the capabilities it actually has**. Example: a retriever only reads chunks from the vector DB (can't write, delete or send out), so you don't analyze it for write risks. An email tool reads the inbox, writes drafts, sends to any address, and acts as the user — far more to analyze.

Each capability comes with one question:

| Capability | Question to ask |
| --- | --- |
| Read | What can it read that this user must not see? |
| Write / update | What can a wrong write change, and can we undo it? |
| Delete | What is lost if it deletes the wrong item? |
| Execute | What can the code reach: files, network, secrets? |
| Send out | Can it send private data to an outside address? |
| Untrusted input | Can an attacker control what it receives? |

## 7. Threat modeling framework — five steps

Threat modeling = a step-by-step way to find all the weaknesses in an application, put security measures in place, then test them.

1. Draw the DFD of the application. *(where)*
2. Mark entry points, exit points and trust boundaries. *(where)*
3. Make a capability table for each component. *(what)*
4. Ask the question for each capability. *(what)*
5. Write one control for each path and test it. *(fix)*

## Glossary

| Term | Meaning |
| --- | --- |
| Asset | Something the app holds that must be protected (user data, documents, credentials, system prompt) |
| Capability | Something the app can do (memory, tools and their permissions, resources behind tools) |
| Adversarial attack | Deliberate manipulation of an AI system's inputs, data, model, memory or tools to serve the attacker's goal |
| Prompt injection | Instructions that override the app's rules — direct (typed by the user) or indirect (hidden in content) |
| Jailbreak | A direct prompt injection aimed at bypassing safety rules |
| Exfiltration | Data leaving the system via a tool call, link or image URL |
| RAG | Retrieval-Augmented Generation: retrieve chunks from a knowledge base, then pass them to the LLM as context |
| DFD | Data Flow Diagram: entities, processes, data stores, flows and trust boundaries |
| Trust boundary | A line on the DFD where the trust level of data changes |
| Threat modeling | Step-by-step framework to find weaknesses, add controls, and test them |

## Quick revision

- AI security = protect **assets** + **capabilities** from unauthorized/malicious access.
- 4 unique challenges: **one channel, non-deterministic, natural language, autonomy**.
- 6 adversarial attacks: direct injection/jailbreak, indirect injection, system prompt extraction, data exfiltration, memory poisoning, knowledge-base poisoning.
- Banking RAG leak root cause: **no per-user access check before retrieval**.
- DFD = 5 elements; it shows data movement, not code or servers.
- Capability lens: analyze each component only for what it can do; 6 capabilities → 6 questions (read, write, delete, execute, send out, untrusted input).
- 5-step method: DFD → entry/exit/trust boundaries → capability table → ask questions → one control per path + test. Mnemonic: **where, where, what, what, fix**.
