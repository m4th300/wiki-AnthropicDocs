# Mode Streaming Input vs Mode Message Unique

> Comprendre les deux modes d'entrée du Claude Agent SDK et quand utiliser chacun

## Mode Streaming Input (recommandé)

Session persistante et interactive. Accès complet aux capacités de l'agent.

### Avantages

- Téléchargements d'images directement dans les messages
- Messages en file d'attente (traitement séquentiel)
- Retours en temps réel
- Persistance du contexte sur plusieurs tours
- Intégration complète des outils et serveurs MCP

### Exemple TypeScript

```typescript
async function* generateMessages(): AsyncGenerator<SDKUserMessage> {
  yield {
    type: "user",
    message: { role: "user", content: "Analyze this codebase for security issues" },
    parent_tool_use_id: null
  };

  await new Promise((resolve) => setTimeout(resolve, 2000));

  // Avec image
  yield {
    type: "user",
    message: {
      role: "user",
      content: [
        { type: "text", text: "Review this architecture diagram" },
        {
          type: "image",
          source: {
            type: "base64",
            media_type: "image/png",
            data: await readFile("diagram.png", "base64")
          }
        }
      ]
    },
    parent_tool_use_id: null
  };
}

for await (const message of query({
  prompt: generateMessages(),
  options: { maxTurns: 10, allowedTools: ["Read", "Grep"] }
})) { ... }
```

### Exemple Python (ClaudeSDKClient)

```python
async def message_generator():
    yield {
        "type": "user",
        "message": {"role": "user", "content": "Analyze this codebase..."},
    }
    await asyncio.sleep(2)

    with open("diagram.png", "rb") as f:
        image_data = base64.b64encode(f.read()).decode()

    yield {
        "type": "user",
        "message": {
            "role": "user",
            "content": [
                {"type": "text", "text": "Review this architecture diagram"},
                {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": image_data}},
            ],
        },
    }

async with ClaudeSDKClient(options) as client:
    await client.query(message_generator())
    async for message in client.receive_response():
        ...
```

## Mode Message Unique

Requêtes ponctuelles, plus simple mais plus limité.

### Quand utiliser

- Réponse ponctuelle
- Pas d'attachements d'images
- Environnement sans état (fonction lambda)

### Limitations

- Pas d'images directes
- Pas de mise en file d'attente dynamique
- Pas d'interruption en temps réel
- Pas de conversations multi-tours naturelles

### Exemple

```python
# Requête simple
async for message in query(
    prompt="Explain the authentication flow",
    options=ClaudeAgentOptions(max_turns=1, allowed_tools=["Read", "Grep"]),
):
    if isinstance(message, ResultMessage):
        print(message.result)

# Continuer la conversation
async for message in query(
    prompt="Now explain the authorization process",
    options=ClaudeAgentOptions(continue_conversation=True, max_turns=1),
):
    if isinstance(message, ResultMessage):
        print(message.result)
```

```typescript
// Simple
for await (const message of query({ prompt: "Explain...", options: { maxTurns: 1 } })) { ... }

// Continuer
for await (const message of query({ prompt: "Now...", options: { continue: true, maxTurns: 1 } })) { ... }
```
