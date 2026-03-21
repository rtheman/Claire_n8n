# Meet Cl(ai)re: What Building an AI Scheduling Bot Taught Me About Building AI Right

*By Rich | Data & Analytics | March 2026*

---

I've spent the better part of 20 years working with data: building pipelines, designing models, and helping organizations turn numbers into decisions. But lately, the conversation has shifted. Everyone wants to talk about AI agents. So I built one. A small one. Her name is Cl(ai)re.

She answers phone calls, checks my calendar, and books appointments. That's it.

And yet, building her taught me more about responsible AI design than I expected. Here's what I learned.

---

## What Is Cl(ai)re?

Cl(ai)re is a voice-enabled AI assistant that manages my schedule. You call her, she picks up, you tell her you'd like to book a meeting with me, and she handles the rest: checking availability, confirming the details, and writing it into my Google Calendar. No back-and-forth email chains. No human in the loop.

If you'd like to schedule a meeting with me via Cl(ai)re, visit [rtheman.com](https://rtheman.com) — she's in the bottom-right corner of the page. She currently speaks English 🇬🇧 and German 🇩🇪.

Under the hood, she's made up of two layers:

- **The brain**: An [ElevenLabs](https://elevenlabs.io) voice agent, guided by a detailed system prompt that defines her personality, scheduling rules, timezone handling, and how she talks to callers.
- **The hands**: An [n8n](https://n8n.io) automation workflow that exposes four Google Calendar tools — get events, create, update, and delete — which Cl(ai)re calls when she needs to act.

Simple, right? In concept, yes. But once I started thinking about what it would mean to deploy something like this in a professional setting, even at this small scale, a few design principles kept coming up. I now think of these as the foundation for building any AI solution responsibly.

---

## The CTC Framework: Consistency, Traceability, Controllability

These three came up naturally as I thought about what could go wrong.

### 1. Consistency

If Alice asks Cl(ai)re whether I'm free next Tuesday and gets one answer, Bryan should get the same answer. That sounds obvious. But it's not a given with AI.

The reason Cl(ai)re gets this right is simple: she doesn't guess. She calls the calendar. Every time. The system prompt explicitly forbids her from simulating or estimating availability — if she doesn't have a tool result, she doesn't give an answer.

**Resolution**: Ground your AI in a single source of truth. Whatever the agent answers should be traceable back to a real data source, not a generated best guess. If the data isn't available, the agent should say so — and not fill the gap with inference.

---

### 2. Traceability

In machine learning, we talk a lot about explainability — being able to point to which features or data signals drove a model's prediction. The same principle applies to AI agents.

For Cl(ai)re, traceability means she can tell the caller *why* a slot is unavailable — not just that it is. It also means every action she takes — every calendar read, every booking — is a discrete, logged step that can be audited.

This matters far more when the stakes are higher. Think about an AI agent automating a financial reconciliation. If it flags a discrepancy, the team needs to know: *what data led to this flag?* Without that, you're flying blind. The agent's output is only as useful as the reasoning behind it.

**Resolution**: Design every agent action as a logged, discrete step. Use workflow tools (like n8n) that record each step's inputs and outputs. For higher-stakes agents, implement structured audit logs and link every output back to the data that produced it — similar to lineage tracking in data engineering.

---

### 3. Controllability

This is the one I feel most strongly about for high-stakes AI.

For Cl(ai)re, the worst-case scenario is that I miss a meeting. Annoying, not catastrophic. But imagine an AI agent that automates month-end payment runs. The stakes are entirely different.

The right design isn't to let the AI act autonomously. It's to treat the AI as an **interpretation layer**. It processes the information, produces a result with a confidence score, and flags it for human review. High-confidence flags get a light touch. Low-confidence flags go to a subject matter expert (SME) who validates, corrects, and — crucially — whose corrections feed back into the agent as new knowledge.

Over time, the agent learns from these edge cases. The SME's workload decreases. Stakeholder confidence in the AI's output grows. And eventually, the organization earns the right to extend automation further — because they've built trust in the system incrementally.

**Resolution**: Don't design for full autonomy on day one. Implement a human-in-the-loop review layer with confidence-based routing. Capture SME corrections as structured feedback that re-enters the agent as updated knowledge or fine-tuning data. This is the loop that makes AI solutions sustainable — and trusted — in a corporate setting.

---

## Five Pitfalls I Noticed in Practice

Beyond the CTC principles, I ran into recurring issues during development. They're worth naming — and most of them have straightforward fixes once you know to look for them.

---

### 1. Context Drift

Over a long conversation — or over time as an agent accumulates context — it can lose sight of the original goal. This is well-recognised in the LLM space, sometimes called *goal drift* or *instruction following degradation*. As the context window fills, earlier instructions get diluted.

**Resolution**: Keep agents focused and narrow — one agent, one domain. Don't build a master agent that knows everything and does everything (jack of all trades). Reinforce core objectives in the system prompt at structured intervals. For long-running agents, implement periodic context summarization to prune noise while preserving intent.

---

### 2. Hallucination

AI can make things up. This is well-documented, and it appears as OWASP's top risk for LLM-based applications. For Cl(ai)re, the guard is simple: she's not allowed to answer anything she doesn't have a verified tool result to support. No tool call, no answer.

For more complex agents, the design principle extends further. A technique called Retrieval-Augmented Generation (RAG) anchors the agent's responses in a verified knowledge base rather than relying solely on what the model was trained on. Think of it as the difference between a consultant who quotes from a source document versus one who speaks from memory.

**Resolution**: Constrain the agent's answer space to verified data sources. Where domain knowledge is required, use RAG to ground responses. Treat hallucination as a first-class bug category with its own test suite — automated evaluation frameworks (such as LLM-as-judge or RAGAS) can systematically test for it across common scenarios before you ship.

---

### 3. Scoping

Unlike code, which is deterministic, AI agents respond to context. A vague prompt can lead the agent to either over-engineer (adding things nobody asked for) or under-engineer (missing important edge cases). The root cause is usually specification ambiguity — the agent fills in the gaps with assumptions.

**Resolution**: Use a structured planning step before building. In tools like Claude Code:
- **Plan Mode** generates a full implementation plan for your review before any code is written — a natural checkpoint to spot gaps and misalignments.
- **Project-level instruction files** (e.g., `CLAUDE.md`) act as standing guardrails, keeping the agent aligned with design standards across sessions.

The principle is simple: agree on scope before you start, not after.

---

### 4. Non-Determinism

This is the pitfall that catches analytics professionals off guard the most. In traditional software, the same input produces the same output. Every time. AI breaks that assumption.

The same prompt, with the same model, can produce materially different outputs across runs. Primarily due to temperature sampling, model version updates, or subtle variations in how the context is assembled. For a scheduling bot, this might mean slightly different phrasing. For a financial reporting agent, it could mean different numbers. In regulated environments, this creates a direct compliance problem: can you reproduce the exact decision your agent made last Tuesday?

**Resolution**: For production agents where consistency matters, set the model temperature to 0 and pin model versions. Design tools so they produce the same result if run twice — no duplicate bookings, no repeated side effects. Log every intermediate step so that even when outputs vary slightly, the full reasoning chain is traceable. Run automated evaluation suites against a fixed test set on every deployment, with quality threshold gates.

---

### 5. Automation Bias

This is the human-side failure that technical teams consistently underestimate. Research shows that users accept AI-generated outputs even when those outputs contradict available evidence — simply because the advice came from an AI. For analytics professionals, this is especially dangerous: an agent that confidently produces a wrong metric or a fabricated trend will often pass unchallenged because it looks authoritative and arrives quickly.

Here's the counterintuitive part: a 2025 study found that adding explanations to AI outputs *increases* blind trust. Users treat the explanation as a validation signal rather than a prompt for scrutiny. More explainability doesn't automatically mean better oversight.

**Resolution**: Display confidence scores alongside results, but pair them with explicit "low confidence" warnings that interrupt the workflow — not ignorable footnotes. For high-stakes outputs (forecasts, anomaly flags, KPIs), require an explicit human confirmation step before the result propagates downstream. Train stakeholders not just on how to *use* agents, but on how to *challenge* them. And consider a simple anti-anchoring practice: have the analyst form their own hypothesis *before* seeing the agent's output.

---

## Two More Attributes Worth Building In

### Observability

One reason tools like n8n and Zapier have become so popular is that they make the process *visible*. You can watch each step execute in real time, see how long each takes, and immediately spot where something went wrong.

This is observability — and it's often undervalued in AI design. It's the difference between a black box and a system you can actually operate. For AI engineers, it's an early warning system. For stakeholders, it's a window into what the AI is actually doing — which builds trust far faster than any presentation ever will.

---

### Version Control

I store Cl(ai)re's system prompt and n8n workflow in a Git repository. This might seem like overkill for a personal project, but the discipline is worth it.

Version control for AI means:
- You can roll back to a previous version when something breaks
- You have a log of every change made, and why
- Others can understand, review, and contribute to the solution

In corporate AI development, this is especially important. The original engineer rarely stays on the project forever. Without version control, their knowledge walks out the door with them. With it, the next person can pick up where they left off.

---

## Putting It Together: The AI Design Framework

Cl(ai)re is a small project. She books meetings. The consequences of her getting something wrong are low. But I deliberately applied these principles as if the stakes were higher — because the habits you build on small projects are the ones you'll carry into large ones.

If I were building an AI agent to automate financial close processes, or to flag compliance issues, or to generate forecasts that drive business decisions — I'd want these same principles baked in from day one. Not retrofitted later when something goes wrong.

The technology is moving fast. The principles, thankfully, are not.

---

*If you'd like to schedule a meeting with me via Cl(ai)re, visit [rtheman.com](https://rtheman.com) — she's in the bottom-right corner of the page. She currently speaks English 🇬🇧 and German 🇩🇪.*

*The project is also open-source on [GitHub](https://github.com/rtheman/Claire_n8n).*

*Have thoughts on this? I'd love to hear how you're thinking about AI design governance in your own work.*
