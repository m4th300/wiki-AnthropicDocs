# Outils personnalisés dans le SDK Agent

> Définissez des outils personnalisés avec le serveur MCP en processus du SDK pour que Claude puisse appeler vos fonctions.

## Référence rapide

| Si vous voulez... | Faites ceci |
|-------------------|-------------|
| Définir un outil | `@tool` (Python) ou `tool()` (TypeScript) |
| Enregistrer auprès de Claude | `create_sdk_mcp_server` + passer à `mcpServers` dans `query()` |
| Pré-approuver | Ajouter à `allowedTools` |
| Supprimer un outil intégré | Passer un tableau `tools` listant uniquement ce que vous voulez |
| Appels parallèles | `readOnlyHint: true` dans les annotations |
| Gérer erreurs sans arrêter | Retourner `isError: true` |
| Retourner images | Bloc `image` dans le tableau `content` |
| Résultat JSON structuré | Définir `structuredContent` sur le résultat |

## Créer un outil

Un outil = nom + description + schéma d'entrée + gestionnaire.

### Python

```python
from claude_agent_sdk import tool, create_sdk_mcp_server
import httpx

@tool(
    "get_temperature",
    "Get the current temperature at a location",
    {"latitude": float, "longitude": float},
)
async def get_temperature(args: dict[str, Any]) -> dict[str, Any]:
    async with httpx.AsyncClient() as client:
        response = await client.get(
            "https://api.open-meteo.com/v1/forecast",
            params={
                "latitude": args["latitude"],
                "longitude": args["longitude"],
                "current": "temperature_2m",
            },
        )
        data = response.json()
    return {
        "content": [{"type": "text", "text": f"Temperature: {data['current']['temperature_2m']}°F"}]
    }

weather_server = create_sdk_mcp_server(name="weather", version="1.0.0", tools=[get_temperature])
```

### TypeScript

```typescript
import { tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const getTemperature = tool(
  "get_temperature",
  "Get the current temperature at a location",
  {
    latitude: z.number().describe("Latitude coordinate"),
    longitude: z.number().describe("Longitude coordinate")
  },
  async (args) => {
    const response = await fetch(`...`);
    const data: any = await response.json();
    return {
      content: [{ type: "text", text: `Temperature: ${data.current.temperature_2m}°F` }]
    };
  }
);

const weatherServer = createSdkMcpServer({ name: "weather", version: "1.0.0", tools: [getTemperature] });
```

## Appeler un outil personnalisé

```python
options = ClaudeAgentOptions(
    mcp_servers={"weather": weather_server},
    allowed_tools=["mcp__weather__get_temperature"],  # Format: mcp__{server}__{tool}
)

async for message in query(
    prompt="What's the temperature in San Francisco?",
    options=options,
):
    if isinstance(message, ResultMessage) and message.subtype == "success":
        print(message.result)
```

## Format du nom d'outil

`mcp__{server_name}__{tool_name}` → `mcp__weather__get_temperature`

Wildcard: `mcp__weather__*` pour tous les outils du serveur.

## Schémas d'entrée

### Python — dict simple
```python
{"latitude": float, "longitude": float}
```

### Python — JSON Schema complet (pour énumérations, optionnels)
```python
{
    "type": "object",
    "properties": {
        "unit_type": {
            "type": "string",
            "enum": ["length", "temperature", "weight"],
        },
        "value": {"type": "number"},
    },
    "required": ["unit_type", "value"],
}
```

### TypeScript — Zod
```typescript
{
  unit_type: z.enum(["length", "temperature", "weight"]).describe("Category"),
  value: z.number(),
  hours: z.number().int().min(1).max(24).default(12)  // .default() = optionnel
}
```

Paramètre optionnel Python: omettre du schéma, mentionner dans la description, lire avec `args.get()`.

## Annotations d'outils

```python
@tool("get_temperature", "...", {"latitude": float}, annotations=ToolAnnotations(readOnlyHint=True))
```

```typescript
tool("get_temperature", "...", { latitude: z.number() }, handler, { annotations: { readOnlyHint: true } })
```

| Champ | Par défaut | Signification |
|-------|-----------|---------------|
| `readOnlyHint` | `false` | Outil en lecture seule (peut appels parallèles) |
| `destructiveHint` | `true` | Peut effectuer des mises à jour destructrices |
| `idempotentHint` | `false` | Appels répétés sans effet supplémentaire |
| `openWorldHint` | `true` | Atteint des systèmes externes |

## Gérer les erreurs

```python
async def fetch_data(args):
    try:
        response = await client.get(args["endpoint"])
        if response.status_code != 200:
            return {
                "content": [{"type": "text", "text": f"API error: {response.status_code}"}],
                "is_error": True,
            }
        return {"content": [{"type": "text", "text": response.json()}]}
    except Exception as e:
        return {
            "content": [{"type": "text", "text": f"Failed: {str(e)}"}],
            "is_error": True,
        }
```

Exception non capturée → boucle d'agent s'arrête. `is_error: True` → boucle continue, Claude peut réessayer.

## Retourner des images

```python
return {
    "content": [{
        "type": "image",
        "data": base64.b64encode(image_bytes).decode("ascii"),  # Base64 brut, pas de préfixe
        "mimeType": "image/png",  # image/png, image/jpeg, image/webp, image/gif
    }]
}
```

## Retourner des ressources

```python
return {
    "content": [{
        "type": "resource",
        "resource": {
            "uri": "file:///tmp/report.md",  # Libellé pour Claude
            "mimeType": "text/markdown",
            "text": "# Report\n...",  # Contenu réel inline
        },
    }]
}
```

## Retourner des données structurées

```typescript
return {
  content: [{ type: "image", data: chartPngBuffer.toString("base64"), mimeType: "image/png" }],
  structuredContent: {
    series: "temperature_2m",
    points: [62.1, 63.4, 65.0]
  }
};
```

Note Python: `@tool` ne transmet que `content` et `is_error`. Pour `structuredContent`, utiliser un serveur MCP autonome.

## Contrôler l'accès aux outils

| Option | Couche | Effet |
|--------|--------|-------|
| `tools: ["Read", "Grep"]` | Disponibilité | Seuls ces outils intégrés dans le contexte |
| `tools: []` | Disponibilité | Supprimer tous les outils intégrés |
| `allowedTools` | Permission | Auto-approuvés, les autres passent au flux de permission |
| `disallowedTools: ["Bash"]` | Disponibilité | Supprime entièrement du contexte |
| `disallowedTools: ["Bash(rm *)"]` | Permission | Refuse les appels correspondants, Bash reste visible |
