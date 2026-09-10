# Adding Strands to a Next.js app

## Install

```bash
npm install @strands-agents/sdk
```

Requires Node.js 22+. The default model provider is Amazon Bedrock; configure AWS credentials, or select another provider via its model class.

## Route handler (App Router)

Create `app/api/agent/route.ts`:

```typescript
import { Agent } from '@strands-agents/sdk'

export const runtime = 'nodejs' // required: the SDK needs the Node.js runtime, not edge
export const maxDuration = 60 // optional; adjust for your deployment

const agent = new Agent({ systemPrompt: 'You are a helpful assistant.' })

export async function POST(req: Request) {
  const { prompt } = await req.json()
  try {
    const result = await agent.invoke(prompt)
    return Response.json({ response: result })
  } catch (err) {
    return Response.json({ error: String(err) }, { status: 500 })
  }
}
```

Key points:
- **`export const runtime = 'nodejs'` is required.** The TypeScript SDK targets Node.js 22+ and will not run in the edge runtime.
- Module-level `Agent` instances are reused across invocations in the same server process; create per-request only if you need isolated session state.

## Streaming

For a streaming response compatible with `useChat`-style clients, adapt Strands' async event stream:

```typescript
export async function POST(req: Request) {
  const { prompt } = await req.json()

  const stream = new ReadableStream({
    async start(controller) {
      for await (const event of agent.stream(prompt)) {
        controller.enqueue(new TextEncoder().encode(`data: ${JSON.stringify(event)}\n\n`))
      }
      controller.close()
    },
  })

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      Connection: 'keep-alive',
    },
  })
}
```

Verify the exact streaming API against the current docs — the TypeScript SDK's streaming surface has changed between versions.

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