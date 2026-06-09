---
title: "اجرای عامل‌های LangGraph در پروداکشن: آنچه یاد گرفتم"
date: 2026-06-01
description: "نگاهی عملی به الگوها، مشکلات و تصمیم‌های زیرساختی که یک عامل LangGraph آزمایشگاهی را از یک سیستم پروداکشن واقعی جدا می‌کند."
tags: ["langgraph", "llm", "python", "fastapi", "production"]
categories: ["هوش مصنوعی"]
author: "مصطفی قدیمی"
showToc: true
---

LangGraph is an excellent framework for building stateful, multi-step AI agents. Getting one to work in a Jupyter notebook is fast. Getting it to handle *thousands of real conversations reliably* is a different problem entirely.

Here are the lessons I've collected from shipping LangGraph agents in production.

---

## 1. State is your agent's memory — design it carefully

The `TypedDict` you pass as `AgentState` is not just a data container; it's the single source of truth for every decision node in your graph. Treat it like a database schema:

- Keep it **flat where possible** — nested dicts make conditional edge logic harder to reason about
- Add explicit **status fields** (`conversation_phase`, `escalation_reason`) rather than inferring state from message history
- Version your state shape and write **migration helpers** — you *will* need to replay old checkpoints

```python
class AgentState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
    phase: Literal["intake", "diagnosis", "resolution", "escalated"]
    escalation_reason: str | None
    skill_context: SkillContext | None
    attempts: int
```

## 2. Checkpointing is not optional

LangGraph's `MemorySaver` is fine for local dev. In production you need a persistent checkpointer — PostgreSQL via `langgraph-checkpoint-postgres` or Redis for lower-latency use cases.

Why it matters:

- **Resume interrupted conversations** without re-running expensive LLM calls
- **Replay for debugging** — reproduce exactly what happened in a failed run
- **Human-in-the-loop** patterns require durable state between turns

```python
from langgraph.checkpoint.postgres import PostgresSaver

checkpointer = PostgresSaver.from_conn_string(settings.database_url)
checkpointer.setup()

graph = builder.compile(checkpointer=checkpointer)
```

## 3. Structured output beats prompt engineering

Rather than asking the model to "respond in JSON", use `model.with_structured_output(MySchema)` everywhere a node needs to make a decision. This gives you:

- Pydantic validation at the LLM boundary
- Automatic retries on parse failure
- Cleaner prompts — the schema *is* the contract

```python
class TriageDecision(BaseModel):
    category: Literal["plumbing", "electrical", "hvac", "general"]
    urgency: Literal["emergency", "urgent", "routine"]
    summary: str = Field(description="One sentence summary of the issue")

decision = llm.with_structured_output(TriageDecision).invoke(messages)
```

## 4. Instrument everything before you need it

Add LangSmith tracing from day one, not after your first production incident:

```python
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "my-agent-prod"
```

The trace UI will save hours when you're debugging why a specific conversation took 14 seconds or why a node made an unexpected branch decision.

## 5. Async all the way down

LangGraph supports `async` graph compilation and invocation. Use it. LLM calls are I/O-bound and async gives you real concurrency when your FastAPI endpoint handles multiple conversations simultaneously.

```python
result = await graph.ainvoke(
    {"messages": [HumanMessage(content=user_message)]},
    config={"configurable": {"thread_id": conversation_id}},
)
```
