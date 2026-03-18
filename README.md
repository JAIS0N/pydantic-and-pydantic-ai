Important Blogs : https://dev.to/hamluk/extending-pydantic-ai-agents-with-dependencies-adding-context-to-your-ai-agents-3f8o, https://szeyusim.medium.com/a-comprehensive-guide-on-agent-development-with-pydantic-ai-beginner-to-advanced-12d90e0ba1a6, https://docs.pydantic.dev/latest/



# AI Agent with PydanticAI

A Jupyter notebook demonstrating how to build a structured AI agent using [PydanticAI](https://docs.pydantic.dev/latest/concepts/pydantic_ai/) — a framework that combines Pydantic's powerful data validation with AI model integrations.

---

## Overview

This notebook walks through two main topics:

1. **Pydantic Basics** — how to define validated data models using `BaseModel`
2. **Building an AI Agent** — a full bank support agent example using PydanticAI

---

## Prerequisites

- Python 3.10+
- An OpenAI API key (for the `gpt-4o` model used in the agent)

### Installation

```bash
pip install pydantic-ai pydantic
```

---

## Notebook Structure

### Part 1: Pydantic Basics

Introduces `BaseModel` with a `User` example, covering:

- Defining typed fields (`int`, `str`, `datetime`, `dict`)
- Automatic type coercion (e.g., string `"1"` → integer `1`)
- `ValidationError` handling for invalid inputs

```python
from pydantic import BaseModel, PositiveInt

class User(BaseModel):
    id: int
    name: str = 'John Doe'
    signup_ts: datetime | None
    tastes: dict[str, PositiveInt]
```

---

### Part 2: Building a Bank Support Agent

Implements a full AI-powered customer support agent with the following components:

#### Dependencies (`SupportDependencies`)
A dataclass holding the customer ID and a database connection, injected at runtime.

#### Result Schema (`SupportResult`)
A Pydantic model defining the structured output the agent must return:

| Field | Type | Description |
|---|---|---|
| `support_advice` | `str` | Advice message for the customer |
| `block_card` | `bool` | Whether to block the customer's card |
| `risk` | `int` (0–10) | Risk level of the query |

#### Agent Setup
```python
support_agent = Agent(
    'openai:gpt-4o',
    deps_type=SupportDependencies,
    result_type=SupportResult,
    system_prompt="You are a support agent in our bank..."
)
```

#### Dynamic System Prompt
Fetches the customer's name from the database and injects it into the system prompt at runtime.

#### Tool: `customer_balance`
A registered async tool that queries the database for the customer's current balance, with an option to include pending transactions.

#### Example Usage

```python
# Query: account balance
result = await support_agent.run("What is my balance?", deps=deps)
# → support_advice='Your balance is $123.45.' | block_card=False | risk=1

# Query: lost card
result = await support_agent.run("I just lost my card!", deps=deps)
# → support_advice='We are blocking your card...' | block_card=True | risk=8
```

---

## Key Concepts

- **Structured outputs** — the agent always returns a validated `SupportResult` object, not free-form text
- **Dependency injection** — runtime context (customer ID, DB connection) is passed cleanly via `deps`
- **Dynamic prompts** — system prompts can be async functions that pull live data
- **Tools** — agents can call registered Python functions to fetch real data during a conversation

---

## Notes

- The `bank_database.DatabaseConn` import is a placeholder for your own database layer
- The agent uses `openai:gpt-4o` — swap to any [supported model](https://docs.pydantic.dev/latest/concepts/pydantic_ai/#models) as needed
- All agent methods are `async` and should be run inside an async context (e.g., `asyncio.run(main())`)
