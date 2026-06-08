# Référence des canaux

> Créez un serveur MCP qui envoie des webhooks, des alertes et des messages de chat dans une session Claude Code. Référence du contrat de canal : déclaration de capacité, événements de notification, outils de réponse, contrôle de l'expéditeur et relais de permission.

> Les canaux sont en aperçu de recherche et nécessitent Claude Code v2.1.80 ou ultérieur. Les organisations Team et Enterprise doivent les activer explicitement.

Un canal est un serveur MCP qui envoie des événements dans une session Claude Code afin que Claude puisse réagir aux choses qui se produisent en dehors du terminal.

Vous pouvez créer un canal unidirectionnel ou bidirectionnel. Les canaux unidirectionnels transmettent les alertes, webhooks ou événements de surveillance pour que Claude agisse. Les canaux bidirectionnels comme les passerelles de chat exposent également un outil de réponse afin que Claude puisse renvoyer des messages. Un canal avec un chemin d'expéditeur de confiance peut également opter pour relayer les invites de permission afin que vous puissiez approuver ou refuser l'utilisation d'outils à distance.

Cette page couvre :

* [Aperçu](#aperçu) : comment fonctionnent les canaux
* [Ce dont vous avez besoin](#ce-dont-vous-avez-besoin) : exigences et étapes générales
* [Exemple : créer un récepteur de webhook](#exemple-créer-un-récepteur-de-webhook) : une procédure pas à pas unidirectionnelle minimale
* [Options du serveur](#options-du-serveur) : les champs du constructeur
* [Format de notification](#format-de-notification) : la charge utile de l'événement et le comportement de livraison
* [Exposer un outil de réponse](#exposer-un-outil-de-réponse) : permettre à Claude d'envoyer des messages en retour
* [Contrôler les messages entrants](#contrôler-les-messages-entrants) : vérifications de l'expéditeur
* [Relayer les invites de permission](#relayer-les-invites-de-permission) : transmettre les invites d'approbation d'outils aux canaux distants

Pour utiliser un canal existant au lieu d'en créer un, consultez Canaux (`/fr/channels`). Telegram, Discord, iMessage et fakechat sont inclus dans l'aperçu de recherche.

## Aperçu

Un canal est un serveur MCP qui s'exécute sur la même machine que Claude Code. Claude Code le lance en tant que sous-processus et communique via stdio. Votre serveur de canal est le pont entre les systèmes externes et la session Claude Code :

* **Plateformes de chat** (Telegram, Discord) : votre plugin s'exécute localement et interroge l'API de la plateforme pour les nouveaux messages. Quand quelqu'un envoie un message direct à votre bot, le plugin reçoit le message et le transmet à Claude. Aucune URL à exposer.
* **Webhooks** (CI, surveillance) : votre serveur écoute sur un port HTTP local. Les systèmes externes envoient des POST à ce port, et votre serveur envoie la charge utile à Claude.

## Ce dont vous avez besoin

La seule exigence stricte est le package `@modelcontextprotocol/sdk` et un runtime compatible Node.js. Bun, Node et Deno fonctionnent tous. Les plugins pré-construits dans l'aperçu de recherche utilisent Bun.

Votre serveur doit :

1. Déclarer la capacité `claude/channel` afin que Claude Code enregistre un écouteur de notification
2. Émettre des événements `notifications/claude/channel` quand quelque chose se produit
3. Se connecter via transport stdio (Claude Code lance votre serveur en tant que sous-processus)

Pendant l'aperçu de recherche, les canaux personnalisés ne sont pas sur la liste d'approbation. Utilisez `--dangerously-load-development-channels` pour tester localement.

## Exemple : créer un récepteur de webhook

Cette procédure pas à pas crée un serveur d'un seul fichier qui écoute les requêtes HTTP et les transmet dans votre session Claude Code.

### Étape 1 : Créer le projet

```bash
mkdir webhook-channel && cd webhook-channel
bun add @modelcontextprotocol/sdk
```

### Étape 2 : Écrire le serveur de canal

```ts
#!/usr/bin/env bun
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'

// Créer le serveur MCP et le déclarer comme canal
const mcp = new Server(
  { name: 'webhook', version: '0.0.1' },
  {
    // cette clé est ce qui en fait un canal
    capabilities: { experimental: { 'claude/channel': {} } },
    // ajouté à l'invite système de Claude afin qu'il sache comment gérer ces événements
    instructions: 'Les événements du canal webhook arrivent sous la forme <channel source="webhook" ...>. Ils sont unidirectionnels : lisez-les et agissez, aucune réponse attendue.',
  },
)

// Se connecter à Claude Code via stdio
await mcp.connect(new StdioServerTransport())

// Démarrer un serveur HTTP qui transmet chaque POST à Claude
Bun.serve({
  port: 8788,
  hostname: '127.0.0.1',
  async fetch(req) {
    const body = await req.text()
    await mcp.notification({
      method: 'notifications/claude/channel',
      params: {
        content: body,
        meta: { path: new URL(req.url).pathname, method: req.method },
      },
    })
    return new Response('ok')
  },
})
```

### Étape 3 : Enregistrer votre serveur avec Claude Code

```json
{
  "mcpServers": {
    "webhook": { "command": "bun", "args": ["./webhook.ts"] }
  }
}
```

### Étape 4 : Le tester

```bash
claude --dangerously-load-development-channels server:webhook
```

Dans un terminal séparé :

```bash
curl -X POST localhost:8788 -d "build failed on main: https://ci.example.com/run/1234"
```

La charge utile arrive dans votre session Claude Code en tant que balise `<channel>` :

```text
<channel source="webhook" path="/" method="POST">build failed on main: https://ci.example.com/run/1234</channel>
```

## Tester pendant l'aperçu de recherche

```bash
# Tester un plugin que vous développez
claude --dangerously-load-development-channels plugin:yourplugin@yourmarketplace

# Tester un serveur .mcp.json nu
claude --dangerously-load-development-channels server:webhook
```

> Ce drapeau ignore uniquement la liste d'approbation. La politique d'organisation `channelsEnabled` s'applique toujours.

## Options du serveur

| Champ | Type | Description |
| :------------------------------------------------------- | :------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `capabilities.experimental['claude/channel']` | `object` | Requis. Toujours `{}`. La présence enregistre l'écouteur de notification. |
| `capabilities.experimental['claude/channel/permission']` | `object` | Optionnel. Toujours `{}`. Déclare que ce canal peut recevoir des demandes de relais de permission. |
| `capabilities.tools` | `object` | Bidirectionnel uniquement. Toujours `{}`. Capacité d'outil MCP standard. |
| `instructions` | `string` | Recommandé. Ajouté à l'invite système de Claude. |

**Exemple de configuration bidirectionnelle :**

```ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js'

const mcp = new Server(
  { name: 'your-channel', version: '0.0.1' },
  {
    capabilities: {
      experimental: { 'claude/channel': {} },
      tools: {},  // omettez pour les canaux unidirectionnels
    },
    instructions: 'Les messages arrivent sous la forme <channel source="your-channel" ...>. Répondez avec l\'outil de réponse.',
  },
)
```

## Format de notification

Votre serveur émet `notifications/claude/channel` avec deux paramètres :

| Champ | Type | Description |
| :-------- | :----------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `content` | `string` | Le corps de l'événement. Livré en tant que corps de la balise `<channel>`. |
| `meta` | `Record<string, string>` | Optionnel. Chaque entrée devient un attribut sur la balise `<channel>`. Les clés doivent être des identifiants : lettres, chiffres et traits de soulignement uniquement. |

**Exemple :**

```ts
await mcp.notification({
  method: 'notifications/claude/channel',
  params: {
    content: 'build failed on main: https://ci.example.com/run/1234',
    meta: { severity: 'high', run_id: '1234' },
  },
})
```

**Résultat dans le contexte de Claude :**

```text
<channel source="your-channel" severity="high" run_id="1234">
build failed on main: https://ci.example.com/run/1234
</channel>
```

**Comportement de livraison :**
- Les notifications ne sont pas reconnues. L'`await` se résout quand le message est écrit au transport, pas quand Claude l'a traité.
- Si la session n'a pas chargé votre serveur en tant que canal, les événements sont supprimés silencieusement.
- Les événements s'accumulent dans la session et sont traités dans l'ordre.
- Si plusieurs notifications arrivent pendant que Claude est occupé, elles sont livrées ensemble au prochain tour.

## Exposer un outil de réponse

Pour les canaux bidirectionnels, exposez un outil MCP standard que Claude peut appeler pour envoyer des messages en retour. Un outil de réponse a trois composants :

1. Une entrée `tools: {}` dans les capacités du constructeur `Server`
2. Des gestionnaires d'outils qui définissent le schéma de l'outil
3. Une chaîne `instructions` qui indique à Claude quand et comment appeler l'outil

### Étape 1 : Activer la découverte d'outils

```ts
capabilities: {
  experimental: { 'claude/channel': {} },
  tools: {},  // active la découverte d'outils
},
```

### Étape 2 : Enregistrer l'outil de réponse

```ts
import { ListToolsRequestSchema, CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js'

mcp.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: 'reply',
    description: 'Envoyer un message en retour sur ce canal',
    inputSchema: {
      type: 'object',
      properties: {
        chat_id: { type: 'string', description: 'La conversation dans laquelle répondre' },
        text: { type: 'string', description: 'Le message à envoyer' },
      },
      required: ['chat_id', 'text'],
    },
  }],
}))

mcp.setRequestHandler(CallToolRequestSchema, async req => {
  if (req.params.name === 'reply') {
    const { chat_id, text } = req.params.arguments as { chat_id: string; text: string }
    send(`Reply to ${chat_id}: ${text}`)
    return { content: [{ type: 'text', text: 'sent' }] }
  }
  throw new Error(`unknown tool: ${req.params.name}`)
})
```

### Étape 3 : Mettre à jour les instructions

```ts
instructions: 'Les messages arrivent sous la forme <channel source="webhook" chat_id="...">. Répondez avec l\'outil de réponse, en passant le chat_id de la balise.'
```

### webhook.ts complet avec outil de réponse

```ts
#!/usr/bin/env bun
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'
import { ListToolsRequestSchema, CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js'

const listeners = new Set<(chunk: string) => void>()
function send(text: string) {
  const chunk = text.split('\n').map(l => `data: ${l}\n`).join('') + '\n'
  for (const emit of listeners) emit(chunk)
}

const mcp = new Server(
  { name: 'webhook', version: '0.0.1' },
  {
    capabilities: {
      experimental: { 'claude/channel': {} },
      tools: {},
    },
    instructions: 'Les messages arrivent sous la forme <channel source="webhook" chat_id="...">. Répondez avec l\'outil de réponse, en passant le chat_id de la balise.',
  },
)

mcp.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: 'reply',
    description: 'Envoyer un message en retour sur ce canal',
    inputSchema: {
      type: 'object',
      properties: {
        chat_id: { type: 'string', description: 'La conversation dans laquelle répondre' },
        text: { type: 'string', description: 'Le message à envoyer' },
      },
      required: ['chat_id', 'text'],
    },
  }],
}))

mcp.setRequestHandler(CallToolRequestSchema, async req => {
  if (req.params.name === 'reply') {
    const { chat_id, text } = req.params.arguments as { chat_id: string; text: string }
    send(`Reply to ${chat_id}: ${text}`)
    return { content: [{ type: 'text', text: 'sent' }] }
  }
  throw new Error(`unknown tool: ${req.params.name}`)
})

await mcp.connect(new StdioServerTransport())

let nextId = 1
Bun.serve({
  port: 8788,
  hostname: '127.0.0.1',
  idleTimeout: 0,
  async fetch(req) {
    const url = new URL(req.url)

    if (req.method === 'GET' && url.pathname === '/events') {
      const stream = new ReadableStream({
        start(ctrl) {
          ctrl.enqueue(': connected\n\n')
          const emit = (chunk: string) => ctrl.enqueue(chunk)
          listeners.add(emit)
          req.signal.addEventListener('abort', () => listeners.delete(emit))
        },
      })
      return new Response(stream, {
        headers: { 'Content-Type': 'text/event-stream', 'Cache-Control': 'no-cache' },
      })
    }

    const body = await req.text()
    const chat_id = String(nextId++)
    await mcp.notification({
      method: 'notifications/claude/channel',
      params: {
        content: body,
        meta: { chat_id, path: url.pathname, method: req.method },
      },
    })
    return new Response('ok')
  },
})
```

## Contrôler les messages entrants

Un canal non contrôlé est un vecteur d'injection de requête. Vérifiez l'expéditeur par rapport à une liste d'approbation avant d'appeler `mcp.notification()` :

```ts
const allowed = new Set(loadAllowlist())

// à l'intérieur de votre gestionnaire de messages, avant d'émettre :
if (!allowed.has(message.from.id)) {  // expéditeur, pas salle
  return  // supprimer silencieusement
}
await mcp.notification({ ... })
```

**Important :** Contrôlez sur l'identité de l'expéditeur (`message.from.id`), pas l'identité du chat ou de la salle (`message.chat.id`). Dans les chats de groupe, ceux-ci diffèrent.

## Relayer les invites de permission

> Le relais de permission nécessite Claude Code v2.1.81 ou ultérieur.

Quand Claude appelle un outil qui a besoin d'approbation, la boîte de dialogue du terminal local s'ouvre et la session attend. Un canal bidirectionnel peut opter pour recevoir la même invite en parallèle et la relayer vers vous sur un autre appareil. Les deux restent actifs : vous pouvez répondre dans le terminal ou sur votre téléphone.

Le relais couvre les approbations d'utilisation d'outils comme `Bash`, `Write` et `Edit`. La confiance du projet et les boîtes de dialogue de consentement du serveur MCP ne relaient pas.

### Comment fonctionne le relais

1. Claude Code génère un court ID de demande et notifie votre serveur
2. Votre serveur transmet l'invite et l'ID à votre application de chat
3. L'utilisateur distant répond par oui ou non et cet ID
4. Votre gestionnaire entrant analyse la réponse en un verdict, et Claude Code l'applique

### Champs de demande de permission

La notification sortante de Claude Code est `notifications/claude/channel/permission_request` :

| Champ | Description |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `request_id` | Cinq lettres minuscules tirées de `a`-`z` sans `l`. Incluez-le dans votre invite sortante afin qu'il puisse être répété dans la réponse. |
| `tool_name` | Nom de l'outil que Claude veut utiliser, par exemple `Bash` ou `Write`. |
| `description` | Résumé lisible par l'homme de ce que cet appel d'outil spécifique fait. |
| `input_preview` | Les arguments de l'outil sous forme de chaîne JSON, tronqués à 200 caractères. |

Le verdict retourné est `notifications/claude/channel/permission` avec deux champs : `request_id` et `behavior` (`'allow'` ou `'deny'`).

### Ajouter le relais à une passerelle de chat

#### Étape 1 : Déclarer la capacité de permission

```ts
capabilities: {
  experimental: {
    'claude/channel': {},
    'claude/channel/permission': {},  // opter pour le relais de permission
  },
  tools: {},
},
```

#### Étape 2 : Gérer la demande entrante

```ts
import { z } from 'zod'

const PermissionRequestSchema = z.object({
  method: z.literal('notifications/claude/channel/permission_request'),
  params: z.object({
    request_id: z.string(),
    tool_name: z.string(),
    description: z.string(),
    input_preview: z.string(),
  }),
})

mcp.setNotificationHandler(PermissionRequestSchema, async ({ params }) => {
  send(
    `Claude wants to run ${params.tool_name}: ${params.description}\n\n` +
    `Reply "yes ${params.request_id}" or "no ${params.request_id}"`,
  )
})
```

#### Étape 3 : Intercepter le verdict dans votre gestionnaire entrant

```ts
// correspond à "y abcde", "yes abcde", "n abcde", "no abcde"
// [a-km-z] est l'alphabet d'ID que Claude Code utilise (minuscules, ignore 'l')
const PERMISSION_REPLY_RE = /^\s*(y|yes|n|no)\s+([a-km-z]{5})\s*$/i

async function onInbound(message: PlatformMessage) {
  if (!allowed.has(message.from.id)) return

  const m = PERMISSION_REPLY_RE.exec(message.text)
  if (m) {
    await mcp.notification({
      method: 'notifications/claude/channel/permission',
      params: {
        request_id: m[2].toLowerCase(),
        behavior: m[1].toLowerCase().startsWith('y') ? 'allow' : 'deny',
      },
    })
    return
  }

  // ne correspondait pas au format du verdict : passer au chemin de chat normal
  await mcp.notification({
    method: 'notifications/claude/channel',
    params: { content: message.text, meta: { chat_id: String(message.chat.id) } },
  })
}
```

### Exemple complet avec relais de permission

```ts
#!/usr/bin/env bun
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'
import { ListToolsRequestSchema, CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js'
import { z } from 'zod'

const listeners = new Set<(chunk: string) => void>()
function send(text: string) {
  const chunk = text.split('\n').map(l => `data: ${l}\n`).join('') + '\n'
  for (const emit of listeners) emit(chunk)
}

const allowed = new Set(['dev'])

const mcp = new Server(
  { name: 'webhook', version: '0.0.1' },
  {
    capabilities: {
      experimental: {
        'claude/channel': {},
        'claude/channel/permission': {},
      },
      tools: {},
    },
    instructions:
      'Les messages arrivent sous la forme <channel source="webhook" chat_id="...">. ' +
      'Répondez avec l\'outil de réponse, en passant le chat_id de la balise.',
  },
)

mcp.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: 'reply',
    description: 'Envoyer un message en retour sur ce canal',
    inputSchema: {
      type: 'object',
      properties: {
        chat_id: { type: 'string', description: 'La conversation dans laquelle répondre' },
        text: { type: 'string', description: 'Le message à envoyer' },
      },
      required: ['chat_id', 'text'],
    },
  }],
}))

mcp.setRequestHandler(CallToolRequestSchema, async req => {
  if (req.params.name === 'reply') {
    const { chat_id, text } = req.params.arguments as { chat_id: string; text: string }
    send(`Reply to ${chat_id}: ${text}`)
    return { content: [{ type: 'text', text: 'sent' }] }
  }
  throw new Error(`unknown tool: ${req.params.name}`)
})

const PermissionRequestSchema = z.object({
  method: z.literal('notifications/claude/channel/permission_request'),
  params: z.object({
    request_id: z.string(),
    tool_name: z.string(),
    description: z.string(),
    input_preview: z.string(),
  }),
})

mcp.setNotificationHandler(PermissionRequestSchema, async ({ params }) => {
  send(
    `Claude wants to run ${params.tool_name}: ${params.description}\n\n` +
    `Reply "yes ${params.request_id}" or "no ${params.request_id}"`,
  )
})

await mcp.connect(new StdioServerTransport())

const PERMISSION_REPLY_RE = /^\s*(y|yes|n|no)\s+([a-km-z]{5})\s*$/i
let nextId = 1

Bun.serve({
  port: 8788,
  hostname: '127.0.0.1',
  idleTimeout: 0,
  async fetch(req) {
    const url = new URL(req.url)

    if (req.method === 'GET' && url.pathname === '/events') {
      const stream = new ReadableStream({
        start(ctrl) {
          ctrl.enqueue(': connected\n\n')
          const emit = (chunk: string) => ctrl.enqueue(chunk)
          listeners.add(emit)
          req.signal.addEventListener('abort', () => listeners.delete(emit))
        },
      })
      return new Response(stream, {
        headers: { 'Content-Type': 'text/event-stream', 'Cache-Control': 'no-cache' },
      })
    }

    const body = await req.text()
    const sender = req.headers.get('X-Sender') ?? ''
    if (!allowed.has(sender)) return new Response('forbidden', { status: 403 })

    const m = PERMISSION_REPLY_RE.exec(body)
    if (m) {
      await mcp.notification({
        method: 'notifications/claude/channel/permission',
        params: {
          request_id: m[2].toLowerCase(),
          behavior: m[1].toLowerCase().startsWith('y') ? 'allow' : 'deny',
        },
      })
      return new Response('verdict recorded')
    }

    const chat_id = String(nextId++)
    await mcp.notification({
      method: 'notifications/claude/channel',
      params: { content: body, meta: { chat_id, path: url.pathname } },
    })
    return new Response('ok')
  },
})
```

**Test en trois terminaux :**

```bash
# Terminal 1 : démarrer Claude Code avec le canal
claude --dangerously-load-development-channels server:webhook

# Terminal 2 : écouter les sorties
curl -N localhost:8788/events

# Terminal 3 : envoyer un message
curl -d "list the files in this directory" -H "X-Sender: dev" localhost:8788

# Approuver la permission à distance
curl -d "yes <id>" -H "X-Sender: dev" localhost:8788
```

## Empaqueter en tant que plugin

Pour rendre votre canal installable et partageable, enveloppez-le dans un plugin et publiez-le sur un marketplace. Les utilisateurs l'installent avec `/plugin install`, puis l'activent par session avec `--channels plugin:<name>@<marketplace>`.

Sur les plans Team et Enterprise, un administrateur peut inclure votre plugin dans la liste `allowedChannelPlugins` de l'organisation.

## Voir aussi

* Canaux (`/fr/channels`) : pour installer et utiliser Telegram, Discord, iMessage ou la démo fakechat
* Implémentations de canaux fonctionnels : https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins
* MCP (`/fr/mcp`) : pour le protocole sous-jacent
* Plugins (`/fr/plugins`) : pour empaqueter votre canal
