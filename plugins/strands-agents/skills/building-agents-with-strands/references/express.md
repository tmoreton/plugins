# Adding Strands to an Express app

## Install

```bash
npm install @strands-agents/sdk
```

Requires Node.js 22+. The default model provider is Amazon Bedrock; configure AWS credentials, or select another provider via its model class.

## Minimal integration

```typescript
import express from 'express'
import { Agent } from '@strands-agents/sdk'

const app = express()
app.use(express.json())
const agent = new Agent({ systemPrompt: 'You are a helpful assistant.' })

app.post('/ask', async (req, res) => {
  try {
    const result = await agent.invoke(req.body.prompt)
    res.json({ response: result })
  } catch (err) {
    res.status(500).json({ error: String(err) })
  }
})

app.listen(3000)
```

## Streaming

Stream agent events to the client with Server-Sent Events:

```typescript
app.post('/ask/stream', async (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream')
  res.setHeader('Cache-Control', 'no-cache')
  res.setHeader('Connection', 'keep-alive')

  for await (const event of agent.stream(req.body.prompt)) {
    res.write(`data: ${JSON.stringify(event)}\n\n`)
  }
  res.end()
})
```

Verify the exact streaming API name and event shape against the current docs — the TypeScript SDK's streaming surface has changed between versions.

## Tools (Zod schema)

```typescript
import { Agent, tool } from '@strands-agents/sdk'
import { z } from 'zod'

const weather = tool({
  name: 'get_weather',
  description: 'Get current weather for a location.',
  inputSchema: z.object({ location: z.string() }),
  callback: (input) => `Weather in ${input.location}: sunny`,
})

const agent = new Agent({ tools: [weather] })
```

## References

- Docs: https://strandsagents.com/docs/user-guide/quickstart/
- TypeScript SDK source: https://github.com/strands-agents/harness-sdk/tree/main/strands-ts