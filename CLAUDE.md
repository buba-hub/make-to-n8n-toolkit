# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Mission

Migrer les workflows d'automatisation Make.com vers n8n (auto-heberge) et les ameliorer/optimiser si le client le souhaite.

## Architecture du projet

```
workflows/<nom-workflow>/
  make-blueprint.json          # Blueprint Make.com source (reference) — NON COMMITTE
  implementation_plan.md       # Plan de migration (cree pendant le processus)
  verification_checklist.md    # Checklist de verification (cree pendant le processus)
  n8n-workflow.json            # Export n8n si applicable (reference uniquement) — NON COMMITTE
```

## Instance n8n

- **Type** : n8n auto-hebergee sur VPS (PAS la version cloud)
- **URL** : voir `env.local` → `N8N_BASE_URL`
- **API** : REST API v1, authentification par JWT dans X-N8N-API-KEY header

> **IMPORTANT** : L'URL, les IDs de workflows et les credentials sont dans `env.local`.
> Chaque utilisateur doit creer son propre `env.local` a partir de `.env.example`.

## Regles strictes de migration

### Creation de workflows
- **TOUJOURS** creer/modifier les workflows directement dans n8n via le MCP n8n — ne JAMAIS generer un fichier JSON a importer manuellement
- Si le MCP n8n est indisponible, utiliser l'API REST n8n via curl en dernier recours

### Nodes et compatibilite
- **INTERDIT** d'utiliser des community nodes — uniquement des nodes natifs n8n
- **TOUJOURS** verifier la compatibilite des nodes avec la derniere version de l'instance n8n via Context7 avant utilisation
- Si aucun node natif ne correspond au besoin, utiliser un node **HTTP Request** plutot qu'un node communautaire ou personnalise
- Verifier que les parametres et options des nodes correspondent a la version actuelle (les APIs de nodes evoluent entre versions)
- **TOUJOURS** consulter `docs/n8n-node-patterns.md` avant de creer un workflow — ce fichier contient les configurations exactes (type, typeVersion, format des parametres, credentials keys) extraites des workflows existants
- **ATTENTION** : Il n'y a PAS de node `n8n-nodes-base.sftp` — utiliser `n8n-nodes-base.ftp` avec `protocol: "sftp"`
- Les champs Airtable base/table utilisent le format `__rl: true` avec `mode: "id"` et la credential key est `airtableTokenApi`
- Utiliser `mcp__n8n-mcp__search_nodes` avec `includeExamples: true` et `includeOperations: true` pour decouvrir les bons parametres d'un node
- Utiliser `mcp__n8n-mcp__get_node` pour obtenir la specification complete d'un node avant de l'utiliser
- En cas de doute sur la configuration d'un node, recuperer un workflow existant qui utilise le meme node via `mcp__n8n-mcp__n8n_get_workflow` (mode full) et reproduire le pattern

### S'inspirer de l'existant dans l'instance n8n
- **TOUJOURS** inspecter les workflows existants dans l'instance n8n qui utilisent des nodes/services similaires avant de configurer un nouveau workflow
- Recuperer via MCP n8n les workflows actifs qui partagent les memes integrations pour en extraire les patterns de configuration (auth, body format, custom fields, credentials ID)
- Reproduire les memes patterns de configuration pour assurer la coherence entre workflows
- Ne jamais deviner la configuration d'un node si un workflow existant montre deja comment le faire

### Documentation et Context7
- **TOUJOURS** utiliser le MCP Context7 (`resolve-library-id` puis `query-docs`) pour consulter la documentation officielle n8n avant d'utiliser un node ou une integration
- Cela concerne : les nodes, les triggers, les expressions, les credentials, les options de configuration
- Ne jamais se fier uniquement aux connaissances internes — la doc n8n evolue frequemment

### Exploitation des MCP existants
- Utiliser le **MCP Airtable** quand un workflow implique des bases Airtable (recuperer schemas, verifier champs, lister records pour contexte)
- Utiliser le **MCP Notion** quand un workflow implique Notion
- Utiliser le **MCP Linear** si le contexte l'implique
- Utiliser les **CLI** en fallback si un MCP rencontre des problemes
- Utiliser **dev-browser** pour tester des interfaces, verifier des webhooks, ou debugger visuellement

### Gestion des credentials
1. Lire `env.local` a la racine du projet pour verifier les credentials disponibles
2. Verifier les credentials existantes dans n8n (via MCP ou API)
3. Reutiliser les credentials n8n existantes autant que possible
4. Si une credential manque : **demander a l'utilisateur**, puis l'ajouter a `env.local` pour les prochaines sessions
5. Ne jamais inventer ou deviner des credentials

## Processus de migration d'un workflow

### Phase 1 : Analyse
1. Lire le `make-blueprint.json` du workflow cible
2. Identifier : declencheur, modules, connexions, APIs externes, transformations de donnees
3. Utiliser les MCP pertinents pour recuperer du contexte (ex: schema Airtable, structure Notion)
4. Presenter a l'utilisateur un resume du role de l'automatisation

### Phase 2 : Cadrage avec l'utilisateur
Poser ces questions :
- Migration simple (reproduction fidele) ou migration avec ameliorations ?
- Y a-t-il des problemes connus avec le workflow Make actuel ?
- Y a-t-il des fonctionnalites a ajouter, modifier ou supprimer ?
- Quel declencheur : schedule, webhook, manuel ?
- Frequence d'execution si schedule ?

### Phase 3 : Planification
1. Creer `implementation_plan.md` dans le dossier du workflow (utiliser le template `templates/implementation_plan.md`)
2. Lister chaque node n8n necessaire avec : type, operation, parametres cles
3. Verifier chaque node via Context7
4. Identifier les credentials necessaires et leur disponibilite
5. Faire valider le plan par l'utilisateur avant implementation

### Phase 4 : Implementation
1. Creer le workflow dans n8n via MCP
2. Configurer chaque node sequentiellement
3. Connecter les nodes entre eux
4. Associer les credentials existantes

### Phase 5 : Verification et tests
1. Creer `verification_checklist.md` (utiliser le template `templates/verification_checklist.md`)
2. Executer le workflow en mode test
3. Verifier chaque node individuellement (entree/sortie)
4. Tester les cas limites (donnees vides, erreurs API, timeouts)
5. Verifier les error handling et retries
6. Iterer jusqu'a ce que le workflow fonctionne completement

### Phase 6 : Finalisation
1. Activer le workflow si schedule/webhook
2. Mettre a jour `docs/migration-tracker.md`
3. Documenter les differences avec le workflow Make original

## Systemes externes connectes

> **Note** : Les IDs et domaines specifiques sont dans `env.local`.

| Systeme | Usage |
|---------|-------|
| Airtable | Stockage donnees |
| Google Sheets | Source donnees |
| Notion | Gestion sprints/projets |
| Freshdesk | Support ticketing |
| Google Calendar | Evenements / calendrier |

## Fichiers importants

- `env.local` — Credentials et cles API (NE JAMAIS COMMITTER)
- `.env.example` — Template des credentials a remplir
- `docs/migration-tracker.md` — Etat d'avancement de toutes les migrations
- `docs/credentials-registry.md` — Registre de toutes les credentials
- `docs/n8n-node-patterns.md` — **Reference obligatoire** : configurations exactes des nodes n8n
- `docs/n8n-instance.md` — Informations techniques sur l'instance n8n
- `templates/` — Templates pour plans et checklists
