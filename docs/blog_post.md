# Meet Cl(ai)re: What Building an AI Scheduling Bot Taught Me About Building AI Right

*By Rich | Data & Analytics | March 2026*

---

I've spent the better part of 20 years working with data — building pipelines, designing models, and helping organisations turn numbers into decisions. But lately, the conversation has shifted. Everyone wants to talk about AI agents. So I built one. A small one. Her name is Cl(ai)re.

She answers phone calls, checks my calendar, and books appointments. That's it.

And yet, building her taught me more about responsible AI design than I expected. Here's what I learned.

---

## What Is Cl(ai)re?

Cl(ai)re is a voice-enabled AI assistant that manages my schedule. You call her, she picks up, you tell her you'd like to book a meeting with me, and she handles the rest — checking availability, confirming the details, and writing it into my Google Calendar. No back-and-forth email chains. No human in the loop.

```
┌─────────────────────────────────────────────────────────┐
│                   HOW CL(AI)RE WORKS                    │
│                                                         │
│   Caller ──► ElevenLabs Voice Agent ──► MCP Protocol   │
│               (Brain: personality,       (The bridge)   │
│                rules, language)               │         │
│                                               ▼         │
│                                    n8n Workflow Engine  │
│                                    (The hands: create,  │
│                                     read, update,       │
│                                     delete events)      │
│                                               │         │
│                                               ▼         │
│                                    Google Calendar API  │
└─────────────────────────────────────────────────────────┘
```

Under the hood, she's made up of two layers:

- **The brain**: An [ElevenLabs](https://elevenlabs.io) voice agent, guided by a detailed system prompt that defines her personality, scheduling rules, timezone handling, and how she talks to callers.
- **The hands**: An [n8n](https://n8n.io) automation workflow that exposes four Google Calendar tools — get events, create, update, and delete — which Cl(ai)re calls when she needs to act.

Simple, right? In concept, yes. But once I started thinking about what it would mean to deploy something like this in a professional setting — even at this small scale — five design principles kept coming up. I now think of these as the foundation for building any AI solution responsibly.

---

## The CTC Framework: Consistency, Traceability, Controllability

These three came up naturally as I thought about what could go wrong.

### 1. Consistency

```
┌─────────────────────────────────────────────────────────┐
│                     CONSISTENCY                         │
│                                                         │
│  Person A asks:        Person B asks:                   │
│  "Is Rich free         "Is Rich free                    │
│   next Tuesday?"        next Tuesday?"                  │
│        │                      │                         │
│        └──────────┬───────────┘                         │
│                   ▼                                     │
│           Same answer. Always.                          │
│         ✓ Ground truth = Google Calendar               │
└─────────────────────────────────────────────────────────┘
```

If Person A asks Cl(ai)re whether I'm free next Tuesday and gets one answer, Person B should get the same answer. That sounds obvious. But it's not a given with AI.

The reason Cl(ai)re gets this right is simple: she doesn't guess. She calls the calendar. Every time. The system prompt explicitly forbids her from simulating or estimating availability — if she doesn't have a tool result, she doesn't give an answer.

The lesson? **Ground your AI in a single source of truth.** Whatever the agent answers should be traceable back to a real data source, not a best guess.

---

### 2. Traceability

```
┌─────────────────────────────────────────────────────────┐
│                    TRACEABILITY                         │
│                                                         │
│  ✗ "Rich is not available Tuesday at 3pm."             │
│                                                         │
│  ✓ "Rich is not available Tuesday at 3pm — there's     │
│     already a meeting in that slot, booked by           │
│     another caller."                                    │
│                                                         │
│  Why it matters: The caller understands the reason.    │
│  The engineer can audit the result.                     │
└─────────────────────────────────────────────────────────┘
```

In machine learning, we talk a lot about explainability — being able to point to which features or data signals drove a model's prediction. The same principle applies to AI agents.

For Cl(ai)re, traceability means she can (in principle) tell the caller *why* a slot is unavailable — not just that it is. It also means every action she takes — every calendar read, every booking — is a discrete step that can be audited.

This matters far more when the stakes are higher. Think about an AI agent automating a financial reconciliation. If it flags a discrepancy, the team needs to know: *what data led to this flag?* Without that, you're flying blind. The agent's output is only as useful as the reasoning behind it.

---

### 3. Controllability

```
┌─────────────────────────────────────────────────────────┐
│                  CONTROLLABILITY                        │
│                                                         │
│         AI Agent Output                                 │
│               │                                         │
│               ▼                                         │
│    ┌─────────────────────┐                              │
│    │   Confidence Score  │                              │
│    └──────────┬──────────┘                              │
│               │                                         │
│    ┌──────────┴──────────┐                              │
│    │                     │                              │
│  High confidence       Low confidence                   │
│    │                     │                              │
│    ▼                     ▼                              │
│ Light review ──────► Human (SME) review                 │
│    │                     │                              │
│    ▼                     ▼                              │
│  Approve            Validate, correct,                  │
│                     feed back into agent                │
└─────────────────────────────────────────────────────────┘
```

This is the one I feel most strongly about for high-stakes AI.

For Cl(ai)re, the worst-case scenario is that I miss a meeting. Annoying, not catastrophic. But imagine an AI agent that automates month-end payment runs. The stakes are entirely different.

The right design isn't to let the AI act autonomously. It's to treat the AI as an **interpretation layer** — it processes the information, produces a result *with a confidence score*, and flags it for human review. High-confidence flags get a light touch. Low-confidence flags go to a subject matter expert (SME) who validates, corrects, and — crucially — whose corrections feed back into the agent as new knowledge.

Over time, the agent learns from these edge cases. The SME's workload decreases. Stakeholder confidence in the AI's output grows. And eventually, the organisation earns the right to extend automation further — because they've built trust in the system incrementally.

This is the loop that makes AI solutions sustainable in a corporate setting.

---

## Three Pitfalls I Noticed in Practice

Beyond the CTC principles, I ran into three recurring issues during development. They're worth naming.

### Context Drift

```
┌─────────────────────────────────────────────────────────┐
│                   CONTEXT DRIFT                         │
│                                                         │
│  Start of session:  "Book a meeting for Tuesday"        │
│                              │                          │
│                    [many exchanges later]               │
│                              │                          │
│                              ▼                          │
│  Agent is now discussing: timezone politics in Europe   │
│                                                         │
│  Fix: Scope agents narrowly. One agent, one job.        │
└─────────────────────────────────────────────────────────┘
```

Over a long conversation — or over time as an agent accumulates context — it can lose sight of the original goal. I call this context drift.

The fix isn't complicated: **keep agents focused**. Don't build a master agent that knows everything and does everything. Instead, build narrow agents with clearly scoped responsibilities. Proper labelling of skills and knowledge helps too — it lets the agent identify and use the right tools for the right task, without getting distracted.

---

### Hallucination

AI can make things up. This is well-documented. For Cl(ai)re, the guard against hallucination is straightforward: she's not allowed to answer anything she doesn't have a tool call to support. There's no room for improvisation.

For more complex agents, the design principle extends further: build in testing and QA processes. Treat hallucination as a bug category with its own test suite. If you're working with domain-specific knowledge, a well-structured knowledge base (sometimes called RAG — Retrieval-Augmented Generation) can anchor the agent's responses in verified information rather than generated assumptions.

---

### Scoping

```
┌──────────────────────────────────────────────────────────┐
│                      SCOPING                             │
│                                                          │
│   Over-engineered:                                       │
│   Agent adds multilingual support, analytics dashboard,  │
│   Slack integration, and a mobile app. Nobody asked.     │
│                                                          │
│   Under-engineered:                                      │
│   Agent books meetings but ignores timezone differences. │
│                                                          │
│   Right-sized:                                           │
│   Agent does exactly what was asked, well.              │
│                                                          │
│   How to get there: Plan Mode → review → confirm gaps   │
│   before building.                                       │
└──────────────────────────────────────────────────────────┘
```

Unlike code, which is deterministic, AI agents respond to context. This means they can either over-engineer (adding things nobody asked for) or under-engineer (missing important edge cases) depending on what context they have access to.

I found that using **Plan Mode** in tools like Claude Code — where the AI outlines its intended approach before acting — is a valuable safeguard. It creates a moment to review, spot gaps, fill in missing context, and align before development starts. It turns an ambiguous prompt into a shared understanding.

---

## Two More Attributes Worth Building In

### Observability

```
┌─────────────────────────────────────────────────────────┐
│                   OBSERVABILITY                         │
│                                                         │
│  Step 1: Receive call          ✓ 0.3s                  │
│  Step 2: Check calendar        ✓ 1.2s                  │
│  Step 3: Confirm availability  ✓ 0.8s                  │
│  Step 4: Write event           ⚠ 4.1s  ← flag this     │
│  Step 5: Confirm to caller     ✓ 0.4s                  │
│                                                         │
│  Without observability, you'd never know Step 4 was     │
│  slow — until it became a problem.                      │
└─────────────────────────────────────────────────────────┘
```

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

```
┌─────────────────────────────────────────────────────────────────────┐
│                     AI DESIGN FRAMEWORK                             │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                  FOUNDATION: CTC                            │  │
│   │   Consistency   │  Traceability  │  Controllability         │  │
│   │   Same answer   │  Show your     │  Human in the loop       │  │
│   │   every time    │  working       │  with feedback           │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                  WATCH OUT FOR                              │  │
│   │  Context Drift  │  Hallucination │  Scoping                 │  │
│   │  Narrow agents  │  Test it like  │  Plan before             │  │
│   │  per domain     │  any feature   │  you build               │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                  MAINTENANCE                                │  │
│   │        Observability         │     Version Control          │  │
│   │        See what's happening  │     Log what changed         │  │
│   │        in real time          │     and why                  │  │
│   └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

Cl(ai)re is a small project. She books meetings. The consequences of her getting something wrong are low. But I deliberately applied these principles as if the stakes were higher — because the habits you build on small projects are the ones you'll carry into large ones.

If I were building an AI agent to automate financial close processes, or to flag compliance issues, or to generate forecasts that drive business decisions — I'd want these same principles baked in from day one. Not retrofitted later when something goes wrong.

The technology is moving fast. The principles, thankfully, are not.

---

*Cl(ai)re is open-source. You can find the project on [GitHub](https://github.com/rtheman/Claire_n8n).*

*Have thoughts on this? I'd love to hear how you're thinking about AI design governance in your own work.*
