# Make.com to n8n Migration Toolkit

Toolkit Claude Code pour migrer des workflows Make.com vers n8n (self-hosted).

Fournit des commandes slash, des templates, des patterns de nodes et un processus structure en 6 phases pour migrer chaque workflow de maniere fiable et reproductible.

## Pre-requis

### 1. Claude Code

Installe Claude Code (CLI) :
```bash
npm install -g @anthropic-ai/claude-code
```

### 2. Instance n8n self-hosted

Une instance n8n auto-hebergee (PAS la version cloud n8n.io).
Tu as besoin de :
- L'URL de ton instance (ex: `https://n8n.mon-serveur.com`)
- Un API Key (JWT) genere depuis n8n : Settings > API > Create API Key

### 3. Serveurs MCP a configurer

Les MCP (Model Context Protocol) sont des serveurs que Claude Code utilise pour interagir avec des services externes. Tu dois les configurer dans ton fichier `~/.claude/settings.json` ou dans `.claude/settings.json` du projet.

#### MCP obligatoires

| MCP | Role | Installation |
|-----|------|-------------|
| **n8n-mcp** | Creer/modifier/tester les workflows n8n | `npx @leonardsellem/n8n-mcp-server` |
| **Context7** | Consulter la doc officielle n8n en temps reel | Integre a Claude Code (MCP distant) |

#### MCP recommandes (selon tes integrations)

| MCP | Role | Quand l'utiliser |
|-----|------|-----------------|
| **Airtable** | Lire schemas, records, champs Airtable | Si tes workflows touchent a Airtable |
| **Notion** | Lire pages/bases Notion | Si tes workflows touchent a Notion |
| **Linear** | Interagir avec Linear | Si tes workflows touchent a Linear |

### 4. CLI recommandes

| Outil | Role | Installation |
|-------|------|-------------|
| `dev-browser` | Navigation web, tests visuels, debug webhooks | `npm install -g dev-browser` |
| `curl` | Fallback pour l'API n8n si MCP indisponible | Pre-installe sur macOS/Linux |

---

## Installation

### Etape 1 : Cloner le repo

```bash
git clone <url-du-repo> migration
cd migration
```

### Etape 2 : Configurer les credentials

```bash
cp .env.example env.local
```

Ouvre `env.local` et remplis toutes les valeurs avec tes propres credentials :
- `N8N_BASE_URL` et `N8N_API_KEY` (obligatoires)
- Les credentials des services que tu utilises (Airtable, Freshdesk, Google, etc.)

### Etape 3 : Configurer les MCP dans Claude Code

Edite `~/.claude/settings.json` (global) ou `.claude/settings.json` (projet) :

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["-y", "@leonardsellem/n8n-mcp-server"],
      "env": {
        "N8N_BASE_URL": "https://ton-instance-n8n.com",
        "N8N_API_KEY": "ton_jwt_token"
      }
    }
  }
}
```

> **Note** : Pour Context7 et les MCP Airtable/Notion, voir la doc Claude Code sur les MCP distants (remote MCP) ou les configurer via l'interface Claude.

### Etape 4 : Verifier que tout fonctionne

Lance Claude Code dans le dossier du projet :
```bash
cd migration
claude
```

Puis teste :
- `/project:migration-status` — doit afficher le tableau de bord (vide au debut)
- Demande a Claude : "Liste les workflows dans n8n" — doit se connecter via MCP

---

## Utilisation

### Commandes disponibles

| Commande | Description |
|----------|-------------|
| `/project:migrate <dossier>` | Migrer un workflow Make vers n8n (processus complet en 6 phases) |
| `/project:check-workflow <dossier>` | Verifier et tester un workflow n8n migre |
| `/project:migration-status` | Afficher le tableau de bord d'avancement |

### Preparer un workflow a migrer

Avant de lancer une migration, tu dois deposer le blueprint Make.com dans le bon dossier :

1. **Cree un dossier** dans `workflows/` avec un nom court et descriptif (sans espaces, en kebab-case) :
   ```bash
   mkdir -p workflows/mon-workflow
   ```

2. **Exporte le blueprint** depuis Make.com :
   - Ouvre ton scenario dans Make.com
   - Menu "..." (trois points) > **Export Blueprint**
   - Sauvegarde le fichier JSON

3. **Depose le blueprint** dans le dossier :
   ```bash
   cp ~/Downloads/blueprint.json workflows/mon-workflow/make-blueprint.json
   ```
   > Le fichier **doit** s'appeler `make-blueprint.json` — c'est le nom que les commandes Claude attendent.

4. **Lance la migration** :
   ```
   /project:migrate mon-workflow
   ```

### Workflow typique complet

1. **Deposer le blueprint** (voir ci-dessus)

2. **Lancer la migration** :
   ```
   /project:migrate <nom>
   ```
   Claude va :
   - Analyser le blueprint
   - Te poser des questions (migration fidele vs ameliorations)
   - Proposer un plan d'implementation
   - Creer le workflow dans n8n via MCP
   - Tester et iterer

3. **Verifier** :
   ```
   /project:check-workflow <nom>
   ```

4. **Suivre l'avancement** :
   ```
   /project:migration-status
   ```

---

## Structure du projet

```
.
├── .claude/
│   └── commands/           # Commandes slash Claude Code
│       ├── migrate.md
│       ├── check-workflow.md
│       └── migration-status.md
├── .env.example            # Template credentials (a copier en env.local)
├── .gitignore
├── CLAUDE.md               # Instructions pour Claude Code
├── README.md               # Ce fichier
├── docs/
│   ├── credentials-registry.md   # Registre des credentials
│   ├── migration-tracker.md      # Suivi d'avancement
│   ├── n8n-instance.md           # Infos techniques instance n8n
│   └── n8n-node-patterns.md      # Patterns de config des nodes (REFERENCE)
├── templates/
│   ├── implementation_plan.md    # Template plan de migration
│   └── verification_checklist.md # Template checklist de tests
└── workflows/
    └── <nom-workflow>/           # Un dossier par workflow
        ├── make-blueprint.json   # Blueprint Make (NON committe)
        ├── implementation_plan.md
        ├── verification_checklist.md
        └── n8n-workflow.json     # Export n8n (NON committe)
```

---

## Configuration Claude Code detaillee

### CLAUDE.md

Le fichier `CLAUDE.md` a la racine contient toutes les instructions pour Claude. Il est deja configure. Tu peux l'adapter a ton contexte :
- Ajouter/retirer des systemes externes dans le tableau
- Modifier le processus de migration si besoin
- Ajouter des regles specifiques a ton instance

### Settings Claude Code

Cree `.claude/settings.local.json` pour tes permissions locales (ce fichier est gitignore) :

```json
{
  "permissions": {
    "allow": [
      "mcp__n8n-mcp__n8n_list_workflows",
      "mcp__n8n-mcp__n8n_get_workflow",
      "mcp__n8n-mcp__n8n_create_workflow",
      "mcp__n8n-mcp__n8n_update_partial_workflow",
      "mcp__n8n-mcp__n8n_update_full_workflow",
      "mcp__n8n-mcp__n8n_test_workflow",
      "mcp__n8n-mcp__n8n_executions",
      "mcp__n8n-mcp__n8n_manage_credentials",
      "mcp__n8n-mcp__n8n_delete_workflow",
      "mcp__n8n-mcp__get_node",
      "mcp__n8n-mcp__search_nodes",
      "mcp__n8n-mcp__validate_node",
      "mcp__context7__resolve-library-id",
      "mcp__context7__query-docs",
      "mcp__airtable__describe_table",
      "mcp__airtable__list_tables",
      "mcp__airtable__search_records",
      "mcp__airtable__get_record",
      "Bash(ls:*)",
      "Bash(curl:*)",
      "Bash(python3:*)"
    ]
  }
}
```

### Global CLAUDE.md (optionnel)

Si tu utilises `dev-browser`, ajoute dans `~/.claude/CLAUDE.md` :

```markdown
## Dev Browser

`dev-browser` est installe globalement sur cette machine. Utilise-le pour toute tache de navigation web, test d'interface, screenshot ou extraction de contenu depuis un site.

Pour decouvrir l'API complete, lance `dev-browser --help`.
```

---

## Mise a jour du doc n8n-node-patterns

Au fur et a mesure des migrations, enrichis `docs/n8n-node-patterns.md` avec les patterns de nodes que tu decouvres. Ce fichier est la reference que Claude consulte avant de creer chaque node. Plus il est complet, plus les migrations sont fiables.

Pour extraire les patterns d'un workflow existant :
```
Demande a Claude : "Extrais les patterns de nodes du workflow [ID] et mets a jour docs/n8n-node-patterns.md"
```

---

## Troubleshooting

| Probleme | Solution |
|----------|----------|
| MCP n8n ne se connecte pas | Verifier `N8N_BASE_URL` et `N8N_API_KEY` dans la config MCP |
| "Community node" erreur | Remplacer par un node natif ou HTTP Request |
| Credential manquante dans n8n | Creer la credential dans l'interface n8n, puis ajouter l'ID dans `docs/n8n-node-patterns.md` |
| Context7 ne repond pas | Utiliser le MCP distant Claude.ai, ou fallback sur `curl` vers la doc n8n |
| Blueprint Make introuvable | Exporter depuis Make.com > Scenario > "..." > Export Blueprint |
