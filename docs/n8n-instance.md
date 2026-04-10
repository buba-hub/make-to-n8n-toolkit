# Instance n8n — Informations techniques

## Hebergement
- **Type** : Auto-hebergee (self-hosted)
- **Hebergeur** : [Votre hebergeur VPS]
- **URL** : voir `env.local` → `N8N_BASE_URL`
- **Version n8n** : A verifier via API (`GET /api/v1/workflows` header ou `/healthz`)

## Acces API
- **Base URL** : `${N8N_BASE_URL}/api/v1`
- **Authentification** : JWT token dans header `X-N8N-API-KEY`
- **Token** : voir `env.local` (cle `N8N_API_KEY`)

## Endpoints principaux

| Endpoint | Methode | Usage |
|----------|---------|-------|
| `/workflows` | GET | Lister tous les workflows |
| `/workflows/{id}` | GET | Details d'un workflow |
| `/workflows` | POST | Creer un workflow |
| `/workflows/{id}` | PUT | Mettre a jour un workflow |
| `/workflows/{id}/activate` | POST | Activer un workflow |
| `/workflows/{id}/deactivate` | POST | Desactiver un workflow |
| `/executions` | GET | Lister les executions |
| `/credentials` | GET | Lister les credentials |

## Workflows existants

> Remplir au fur et a mesure des migrations.

| ID | Nom | Statut | Origine |
|----|-----|--------|---------|
| | | | |

## Contraintes
- Pas de community nodes disponibles — utiliser uniquement les nodes natifs
- Verifier la version avant d'utiliser des features recentes
- Les mises a jour n8n doivent etre faites manuellement sur le VPS
