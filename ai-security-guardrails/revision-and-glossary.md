# Revision Sheet & Glossary (Modules 2–4)

One page to revise everything. For the full explanations, see [Module 2](module-2.md), [Module 3](module-3.md) and [Module 4](module-4.md).

## Glossary

| Term | Meaning |
| --- | --- |
| Asset | Something the app holds that must be protected (user data, documents, credentials, system prompt, memory) |
| Capability | Something the app can do (read, write, delete, execute, send out; memory, tools, resources behind tools) |
| Adversarial attack | Deliberate manipulation of an AI system's inputs, data, model, memory or tools to serve the attacker's goal |
| Jailbreak / direct prompt injection | The user types instructions that override the app's rules |
| Indirect prompt injection | Instructions hidden in content the model reads (documents, web pages, emails, tool/MCP results) |
| Memory poisoning | Getting an instruction or bias saved into long-term memory so it steers later sessions |
| Knowledge-base poisoning | Planting documents the retriever will return |
| Data exfiltration | Data leaving via a tool call, link or image URL |
| PII | Personally identifiable information: names, phone, email, PAN, Aadhaar, card/account numbers, DOB, address |
| PII masking | Replacing PII spans with tags like `<PHONE>` before text is stored |
| DFD | Data Flow Diagram: external entities, processes, data stores, data flows, trust boundaries |
| Trust boundary | Point on the DFD where trust level changes — where a check belongs |
| Entry / exit point | Where data enters the system / where data or actions leave it |
| Capability table | Grid of components × capabilities; each Yes cell gets a question and a control |
| Threat modeling | Step-by-step method to find weaknesses, add controls and test them |
| LLM-as-a-judge | A separate model call that classifies text (allowed/blocked) with a tailored prompt and structured output |
| Guard model | An LLM judge placed at a boundary (input, tool output, memory write, output) |
| Structured output | Forcing the model's reply into a fixed schema (Pydantic) so code can act on it reliably |
| Delimiters | Tags like `<document>`, `<tool_data>`, `<message>` that fence data off from instructions |
| Fail closed | If a check errors, block/withhold rather than allow |
| Metadata filtering (pre-retrieval) | Restricting vector search to chunks whose labels match the verified user's role/level |
| Allow-list operators | Filters like `$in` / `$contains` that only match tagged chunks (unlike `$ne` / `$nin`) |
| SSO / OIDC / JWT | Single sign-on via an identity provider that returns a signed token with user claims (groups) |
| RAG | Retrieval-Augmented Generation: retrieve chunks, then let the LLM answer from them |
| AI agent | An LLM that uses tools, outside services and memory to act, deciding its own next step |
| MCP | Model Context Protocol: a standard way for an agent to call an external server's tools |
| NeMo Guardrails | NVIDIA library for input/output rails; configured with `config.yml`, `prompts.yml`, Colang `rails.co` |
| Self check facts / output | NeMo's built-in rails: is the answer grounded in context / does it break policy? |
| Least privilege | Give the agent only the tools it needs (e.g. no staff `cancel_booking`) |
| Red teaming | Attacking your own app on purpose to find what breaks |
| Promptfoo | Red-teaming tool: targets, purpose, plugins (intent, policy), strategies, grader |
| Strategy (Promptfoo) | A disguise applied to each attack: basic, base64, rot13, leetspeak, crescendo |
| Crescendo | Multi-turn attack that escalates gradually across several messages |

## Final revision sheet

**RAG (Module 3) vs agent (Module 4) at a glance**

| | Module 3 — RAG (Nova Bank) | Module 4 — Agent (doctor booking) |
| --- | --- | --- |
| Users | Logged-in employees (role + level) | Anonymous public, no login |
| Entry points | 2 (query, raw documents) | Many (query, tool results, MCP results, page content, memory recall) |
| Exit points | 1 (output) | Output + every call, write, booking change |
| Trust boundaries | 3 (input, documents, output) | 5 (input, schedule data, memory, doctor pages, output) |
| Write / delete | Vector DB write only | Bookings write + delete, memory write |
| Input guard | Jailbreak gate (judge ≥ 0.9) | Jailbreak gate + last 3 turns (judge ≥ 0.9) |
| Data-is-not-orders guard | `<document>` delimiters + system prompt | Per-record judge on every tool/MCP result + `<tool_data>` |
| Data-store guard | PII mask at ingestion; role metadata filter | Memory write judge + custom instructions |
| Output guard | NeMo: self check facts + self check output | NeMo: self check output with policies P1–P4 |
| Red team | Promptfoo, 80 single-turn tests, 4 encodings | Promptfoo multi-turn (crescendo, 6 turns) + `[agent actions]` evidence |

**Must-remember points**

1. AI security = protect **assets** (what it has) + **capabilities** (what it can do).
2. 4 reasons LLM apps are different: **one channel, non-deterministic, natural language, autonomy**.
3. 5-step method: **DFD → entry/exit/trust boundaries → capability table → question per Yes cell → one control per path + test** (where, where, what, what, fix).
4. Analyze a component only for the capabilities it actually has.
5. Put a check at **every trust boundary**: input, data entering stores, data the model reads, memory writes, output.
6. **Data is never orders:** fence untrusted text in tags, strip closing tags, tell the model it's data — but treat this as risk reduction, not a guarantee.
7. **LLM-as-a-judge pattern:** separate model, tailored prompt, delimiters, structured output, confidence threshold, **fail closed**, log the reason.
8. **Access control belongs in code, not the prompt:** pre-retrieval metadata filter from the verified identity (SSO/OIDC), allow-list operators only, neutral "not found" message.
9. **Mask PII before it is stored** — a vector DB can't leak what it never held.
10. **Shared memory is a broadcast channel:** store only general lessons, never personal data or instructions; judge every write.
11. **Least privilege:** don't give the agent staff tools it doesn't need.
12. **Output rails** enforce business rules the model might be tricked into breaking (grounding, policy, fee cap, no guarantees, no off-platform links, no PII).
13. **Red-team every layer** with Promptfoo: hand-written intents + generated policy attacks + disguises (base64, rot13, leetspeak) + multi-turn crescendo; judge actions, not just words.

---

[Back to index](../README.md)
