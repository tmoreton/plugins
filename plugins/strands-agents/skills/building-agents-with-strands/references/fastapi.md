# Adding Strands to a FastAPI app

## Install

```bash
pip install strands-agents
# optional community tools:
pip install strands-agents-tools
```

The default model provider is Amazon Bedrock; configure AWS credentials (and enable model access in the target region), or select another provider via its model class.

## Minimal integration

```python
from fastapi import FastAPI
from pydantic import BaseModel
from strands import Agent

app = FastAPI()
agent = Agent(system_prompt="You are a helpful assistant.")

class Ask(BaseModel):
    prompt: str

@app.post("/ask")
async def ask(body: Ask):
    result = agent(body.prompt)
    return {"response": str(result)}
```

Notes:
- `Agent` is not async by default; calling `agent(...)` blocks the event loop worker. For high-concurrency endpoints, run the call in a threadpool (`run_in_executor` / `anyio.to_thread.run_sync`) or use the streaming API with an async generator.
- Create the `Agent` once at module level (or per-request if you need isolated sessions) — construction is cheap but sessions/state live on the instance.

## Streaming endpoint

Use Strands' streaming API and yield events from a `StreamingResponse`:

```python
from fastapi.responses import StreamingResponse
from strands import Agent

agent = Agent(system_prompt="You are a helpful assistant.")

@app.post("/ask/stream")
async def ask_stream(body: Ask):
    async def generate():
        async for event in agent.stream_async(body.prompt):
            if "data" in event:
                yield event["data"]
    return StreamingResponse(generate(), media_type="text/plain")
```

Verify the exact streaming event shape against the current docs (`search_docs` tool or https://strandsagents.com/docs/user-guide/) — event keys have changed between SDK versions.

## Tools

```python
from strands import Agent, tool

@tool
def word_count(text: str) -> int:
    """Count words in text."""
    return len(text.split())

agent = Agent(tools=[word_count])
```

The docstring becomes the tool description the model sees. For MCP tools, see the main SKILL.md.

## References

- Docs: https://strandsagents.com/docs/user-guide/quickstart/
- Python SDK source: https://github.com/strands-agents/harness-sdk/tree/main/strands-py