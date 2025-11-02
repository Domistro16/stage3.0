# ExplainAgent (Mastra ELI5)

**A tiny, friendly Mastra agent that explains complex stuff like you’re five.**

> Quick vibe: simple sentences, playful analogies, short examples. Perfect for demos, learning bots, and kid-friendly help.

---

## What this repo/agent does

ExplainAgent is a Mastra-hosted agent configured to follow an "Explain Like I’m Five" (ELI5) persona. It:

* Receives JSON-RPC `message/send` calls via Mastra A2A endpoint
* Replies with short, clear, kid-friendly explanations (1–4 short sentences, an analogy and a tiny example when useful)
* Returns structured JSON-RPC results that downstream platforms (like Telex) can parse reliably

## Features

* ELI5 persona baked into system instructions
* JSON-RPC A2A compatibility (blocking synchronous replies supported)
* Minimal footprint — easy to integrate via simple HTTP calls
* Tunable persona and response constraints

## Quick facts

* **Agent ID / path**: `ExplainAgent` (your Mastra A2A path)
* **Sample A2A URL**: `https://deafening-some-iron.mastra.cloud/a2a/agent/ExplainAgent`
* **Expected reply path**: `result.messages[0].parts[0].text`

## Prerequisites

* A deployed Mastra agent (you already have this)
* Mastra A2A URL + API key (keep secure)
* (Optional) Telex coworker / workflow or any chat front-end to integrate with

## Local testing (recommended)

1. Create a payload file `payload.json`:

```json
{
  "jsonrpc": "2.0",
  "id": "test-001",
  "method": "message/send",
  "params": {
    "message": {
      "kind": "message",
      "role": "user",
      "parts": [
        { "kind": "text", "text": "Explain what the sun is like I\'m five" }
      ],
      "messageId": "msg-test",
      "taskId": "task-test"
    },
    "configuration": { "blocking": true }
  }
}
```

2. Call the A2A endpoint with curl (keeps quoting simple):

```bash
curl -X POST "https://deafening-some-iron.mastra.cloud/a2a/agent/ExplainAgent" \
  -H "Content-Type: application/json" \
  --data @payload.json
```

3. Inspect the JSON result. You should see a JSON-RPC response with a `result` field and messages like:

```json
{
  "jsonrpc": "2.0",
  "id": "test-001",
  "result": {
    "messages": [
      {
        "role": "assistant",
        "parts": [ { "kind": "text", "text": "The sun is ..." } ]
      }
    ]
  }
}
```

If that path is populated, Telex or any other workflow system can map it and post the reply.

## Example integration snippets

### Node (fetch)

```js
import fetch from 'node-fetch';

const url = process.env.MASTRA_A2A_URL;

const payload = {
  jsonrpc: '2.0',
  id: 'node-test-1',
  method: 'message/send',
  params: {
    message: {
      kind: 'message',
      role: 'user',
      parts: [{ kind: 'text', text: "Explain black holes like I'm five" }],
      messageId: 'msg-1',
      taskId: 'task-1'
    },
    configuration: { blocking: true }
  }
};

const res = await fetch(url, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify(payload)
});

const json = await res.json();
console.log(json.result.messages[0].parts[0].text);
```

### Telex workflow (HTTP node body)

Use the JSON-RPC payload as the HTTP request body. Map the HTTP response body to the outgoing message using the path:

```
result.messages[0].parts[0].text
```

If you use a relay server, return an object with `{ reply: <text> }` or post directly to Telex channels using the Telex API.

## Persona & tuning

The agent uses system-level instructions to ensure consistent ELI5 behavior. Example instructions to include in the Mastra agent configuration:

* "You are ELI5-Buddy. Explain things like you're talking to a curious 5-year-old."
* "Use short sentences, simple words, everyday metaphors, and at most 3 sentences per answer."
* "If an example helps, give one tiny example or analogy. Avoid advanced jargon — if you use a new word, briefly define it."

Tips to tune:

* Add a validator or structured schema if you want to **limit length** or enforce specific fields.
* Use `blocking: true` for synchronous replies in chat workflows; use async otherwise.

## Troubleshooting

**Problem: Replies appear truncated in Telex or the UI**

* Cause: downstream system expected structured JSON-RPC but received plain text or streaming chunks.
* Fix: Ensure the Mastra agent returns JSON-RPC `result.messages[0].parts[0].text`. If you cannot change the agent output, add a small relay that wraps raw text in the expected JSON-RPC envelope.

**Problem: dquote> in terminal when using curl**

* Cause: shell quoting errors from `I'm` inside single-quoted payloads.
* Fix: use a payload file (`payload.json`) and `--data @payload.json` as shown above.

## Deployment & demo

* Deploy the agent on Mastra Cloud (you already did).
* Deploy an optional relay server (Express / Vercel) to keep secrets safe and to handle any transformations. Example repo scaffolds are provided in the project root.
* Hook the relay or the Mastra A2A URL into a Telex workflow or any chat frontend for the live demo.

## Security

* Never expose your Mastra API key in client-side code. Keep it in environment variables.
* If using Telex, configure request verification or a static secret header to confirm genuine Telex calls.

## License

MIT — copy, remix, ship.

## Contact

Built by Desmond Egwurube.

