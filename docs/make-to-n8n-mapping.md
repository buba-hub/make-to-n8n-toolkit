# Equivalences Make.com → n8n

Reference de traduction pour la migration. Deux sections :
1. **Modules / primitives** Make → node n8n equivalent
2. **Cheatsheet IML** → expression n8n (JS / Luxon)

> **Portee** : ce doc dit *par quoi remplacer* un module/fonction Make. Pour la **config exacte** du node n8n cible (typeVersion, format des parametres, credential keys), se referer a `docs/n8n-node-patterns.md` — ne pas dupliquer ces details ici.
>
> **Methode** : pour interpreter precisement un module Make ambigu avant de choisir l'equivalent, s'appuyer sur les skills `make-scenario-building` et `make-module-configuring` (cf. Phase 1.b de `CLAUDE.md`).

---

## 1. Modules / primitives Make → n8n

### 1.1 Primitives de flux (le coeur des contresens)

| Primitive Make | Comportement Make | Equivalent n8n | Notes de migration |
|---|---|---|---|
| **Iterator** (array → bundles) | Eclate un array en N bundles, traites un par un en aval | **Split Out** (`splitOut`) ou *Loop Over Items* (`splitInBatches`) | En n8n, 1 bundle Make ↔ 1 item. Souvent l'Iterator disparait : les nodes n8n traitent deja chaque item. Utiliser `splitInBatches` seulement si une vraie boucle sequentielle est requise. |
| **Aggregator** (`builtin:BasicAggregator`, Array/Numeric/Text) | Recombine N bundles en 1 seul (array, somme, concat) | **Aggregate** (`aggregate`) ou **Summarize** (`summarize`) | Array Aggregator → `aggregate`. Numeric → `summarize`. Text Aggregator (concat avec separateur) → `aggregate` puis `Code`/`Set` pour join. |
| **Router** (`builtin:BasicRouter`) | Duplique le flux sur plusieurs routes, chacune avec son filtre | **Switch** (multi-branches) ou **IF** (2 branches) | 1 route Make = 1 sortie de Switch. Le **filtre de route** Make devient la **condition** du Switch/IF. Voir `n8n-node-patterns.md`. |
| **Filter** (sur un lien entre 2 modules) | Laisse passer le bundle si condition vraie, sinon stoppe la branche | **IF** (branche true uniquement) ou **Filter** (`filter`) | Le node `Filter` supprime les items non conformes sans brancher — equivalent le plus fidele a un filtre de lien Make. |
| **Router + fallback route** | Derniere route sans filtre = "sinon" | Sortie **fallback** du Switch (`Fallback Output`) | Activer l'option fallback du Switch plutot qu'une condition negative manuelle. |

### 1.2 Variables, donnees, format

| Module Make | Equivalent n8n | Notes |
|---|---|---|
| `util:SetVariable2` / `util:SetVariables` | **Set / Edit Fields** (`set`) | Les variables Make sont scopees au flux ; en n8n elles deviennent des champs d'item. Si une valeur doit survivre a une boucle, utiliser le *static data* du workflow via un node `Code`. |
| `json:TransformToJSON` (Create JSON) | Node **Code** (`JSON.stringify`) ou expression `{{ JSON.stringify($json.x) }}` | |
| `json:ParseJSON` | Node **Code** (`JSON.parse`) ou *Set* avec expression | n8n parse souvent deja le JSON des reponses HTTP automatiquement. |
| Tools > **Text aggregator / Numeric** | `aggregate` / `summarize` | Voir 1.1. |
| **Data store** Make | n8n n'a pas d'equivalent natif simple. Options : *static data* (via `Code`), une table Airtable/Sheets dediee, ou n8n **Data Tables** si l'instance le supporte | Choisir selon la duree de vie et le volume. Documenter le choix. |

### 1.3 Gestion d'erreur (directives Make)

| Directive Make | Equivalent n8n |
|---|---|
| `builtin:Ignore` (error handler) | Sur le node source : **Settings → On Error → Continue (using error output)** ou *Continue on Fail* |
| `builtin:Resume` | Error output branch (gerer l'erreur puis continuer le flux) |
| `builtin:Break` | Pas d'equivalent direct ; gerer via *Stop And Error* ou condition d'arret de boucle |
| `Rollback` / `Commit` | Pas de transaction native n8n. Reproduire la logique manuellement (compensation) si critique. |
| Retry / "number of consecutive errors" | Node Settings → **Retry On Fail** (max tries, wait between tries) |

### 1.4 Triggers

| Trigger Make | Equivalent n8n |
|---|---|
| `gateway:CustomWebHook` | **Webhook** (`webhook`) — voir `n8n-node-patterns.md` |
| Schedule / Clock | **Schedule Trigger** (`scheduleTrigger`) |
| Instant / app trigger (polling) | Trigger natif de l'app si dispo, sinon **Schedule Trigger** + node de lecture + dedup |
| Execution manuelle | **Manual Trigger** (`manualTrigger`) |

### 1.5 Modules d'app courants

| Module Make | Node n8n | Notes |
|---|---|---|
| `airtable:ActionGetRecord` / `ActionSearchRecords` | **Airtable** (Get / Search) | Base/table en format `__rl: true`, credential `airtableTokenApi`. Voir `n8n-node-patterns.md`. |
| `airtable:ActionCreateRecord` / `ActionUpdateRecord` | **Airtable** (Create / Update) | |
| `google-email:sendAnEmail` | **Gmail** (`gmail`, sendEmail) ou **Send Email** (SMTP) | Verifier la credential dispo dans n8n avant de choisir. |
| `google-sheets:*` | **Google Sheets** | |
| `google-calendar:*` | **Google Calendar** | OAuth2. |
| `notion:searchObjects` / `updateADatabaseItem` | **Notion** | |
| `freshdesk:getATicket` / `createANote` | **Freshdesk** (si node natif dispo) sinon **HTTP Request** | Reproduire le pattern d'un workflow Freshdesk existant. |
| `hubspotcrm:*` | **HubSpot** ou **HTTP Request** | Normaliser enums/multi-checkbox avant POST (voir `n8n-node-patterns.md`). |
| **Make an API Call** (generique app) | **HTTP Request** | Reprendre baseURL + auth de la connexion Make. |

> **Regle** : si aucun node natif n8n ne correspond, utiliser **HTTP Request** (jamais de community node). Toujours verifier la dispo du node natif via Context7 + MCP n8n avant de trancher.

---

## 2. Cheatsheet IML → expression n8n

Make = IML dans `{{ ... }}`. n8n = JavaScript dans `{{ ... }}`, dates via **Luxon** (`$now`, `DateTime`). Reference des items : `{{ $json.champ }}`, node amont : `{{ $('NomDuNode').item.json.champ }}`.

### 2.1 Reference de donnees

| IML Make | n8n |
|---|---|
| `{{1.field}}` (module 1) | `{{ $('Nom Node 1').item.json.field }}` |
| `{{2.array[].id}}` | `{{ $('Node 2').item.json.array.map(x => x.id) }}` |
| `{{1.field}}` du module precedent immediat | `{{ $json.field }}` |

### 2.2 Texte

| IML Make | n8n |
|---|---|
| `toString(x)` | `{{ String(x) }}` ou `{{ $json.x.toString() }}` |
| `length(s)` | `{{ s.length }}` |
| `upper(s)` / `lower(s)` | `{{ s.toUpperCase() }}` / `{{ s.toLowerCase() }}` |
| `substring(s; start; end)` | `{{ s.substring(start, end) }}` |
| `replace(s; old; new)` | `{{ s.replace(old, new) }}` (ou `.replaceAll`) |
| `trim(s)` | `{{ s.trim() }}` |
| `split(s; sep)` | `{{ s.split(sep) }}` |
| `join(arr; sep)` | `{{ arr.join(sep) }}` |
| `remove(s; sub)` | `{{ s.replace(sub, '') }}` |
| `contains(s; sub)` | `{{ s.includes(sub) }}` |

### 2.3 Nombres / math

| IML Make | n8n |
|---|---|
| `add(a; b)` | `{{ a + b }}` |
| `round(x)` / `floor` / `ceil` | `{{ Math.round(x) }}` / `Math.floor` / `Math.ceil` |
| `formatNumber(x; dec; sep)` | `{{ x.toFixed(dec) }}` (separateur via `toLocaleString`) |
| `parseNumber(s)` | `{{ Number(s) }}` ou `parseFloat(s)` |

### 2.4 Dates (Luxon en n8n)

| IML Make | n8n (Luxon) |
|---|---|
| `now` | `{{ $now }}` |
| `formatDate(d; "YYYY-MM-DD")` | `{{ $now.toFormat('yyyy-MM-dd') }}` ou `{{ DateTime.fromISO(d).toFormat('yyyy-MM-dd') }}` |
| `addDays(d; n)` | `{{ DateTime.fromISO(d).plus({ days: n }).toISO() }}` |
| `addHours/addMinutes` | `.plus({ hours: n })` / `.plus({ minutes: n })` |
| `parseDate(s; fmt)` | `{{ DateTime.fromFormat(s, fmt) }}` |
| Difference de dates | `{{ DateTime.fromISO(a).diff(DateTime.fromISO(b), 'days').days }}` |

> Attention aux tokens de format : Make `YYYY/DD` ≠ Luxon `yyyy/dd`. Luxon : `yyyy` (annee), `MM` (mois), `dd` (jour), `HH:mm:ss`.

### 2.5 Logique / conditions

| IML Make | n8n |
|---|---|
| `if(cond; a; b)` | `{{ cond ? a : b }}` |
| `ifempty(x; fallback)` | `{{ x || fallback }}` (ou `?? ` si seul `null`/`undefined`) |
| `isempty(x)` | `{{ !x }}` ou `{{ x == null \|\| x === '' }}` |
| `coalesce(a; b; c)` | `{{ a ?? b ?? c }}` |
| `not(x)` | `{{ !x }}` |
| `and` / `or` | `{{ a && b }}` / `{{ a \|\| b }}` |

### 2.6 Arrays / collections

| IML Make | n8n |
|---|---|
| `map(arr; field)` | `{{ arr.map(x => x.field) }}` |
| `length(arr)` | `{{ arr.length }}` |
| `first(arr)` / `last(arr)` | `{{ arr[0] }}` / `{{ arr[arr.length - 1] }}` |
| `merge(a; b)` | `{{ [...a, ...b] }}` |
| `get(obj; path)` | `{{ obj.path }}` (acces direct) |
| `keys(obj)` / `toArray(obj)` | `{{ Object.keys(obj) }}` / `{{ Object.values(obj) }}` |

### 2.7 Pieges courants

- **`null` vs `erase`** dans un mapper Make : un champ mappe a `erase` doit etre **omis** du body n8n (ne pas envoyer `null`), sinon l'API peut ecraser la valeur. Filtrer les champs vides avant POST.
- **Bundling implicite** : un module Make qui sort N bundles fait tourner l'aval N fois. En n8n c'est N items — ne pas rajouter de boucle si le node aval itere deja par item.
- **Boucles + paired item** : voir `docs/n8n-node-patterns.md` (expressions dans les boucles SplitInBatches). Bug classique : toutes les iterations modifient le meme enregistrement → verifier que l'output est distinct par iteration.
- **Index 1-based** : IML est parfois 1-based, JS est 0-based (`arr[0]`).

---

## Maintenance

Completer ce doc au fil des migrations : a chaque module Make rencontre sans equivalent evident, ajouter une ligne ici (et le pattern de config exact dans `n8n-node-patterns.md`).
