# Plan de migration : [NOM DU WORKFLOW]

## Origine
- **Blueprint Make** : `workflows/[dossier]/make-blueprint.json`
- **Scenario Make ID** : [ID si disponible]
- **Date d'analyse** : [DATE]

## Resume du workflow Make
[Description en 3-5 phrases de ce que fait l'automatisation, son declencheur, sa frequence, et son objectif metier]

## Decision utilisateur
- [ ] Migration fidele (reproduction exacte)
- [ ] Migration avec ameliorations

### Ameliorations demandees (si applicable)
- [Liste des modifications/ajouts/suppressions]

## Architecture n8n proposee

### Declencheur
| Propriete | Valeur |
|-----------|--------|
| Type de node | [Schedule Trigger / Webhook / Manual Trigger] |
| Configuration | [Cron expression / URL / etc.] |

### Nodes

| # | Node n8n | Operation | Service/API | Credential requise | Notes |
|---|----------|-----------|-------------|-------------------|-------|
| 1 | [Type] | [Operation] | [Service] | [Nom credential] | [Details] |
| 2 | ... | ... | ... | ... | ... |

### Flux de connexions
```
[Node 1] -> [Node 2] -> [Node 3]
                      -> [Node 4] (branche conditionnelle)
```

## Credentials

### Disponibles dans n8n
| Credential | Type | Statut |
|------------|------|--------|
| [Nom] | [Type] | Existante |

### A creer
| Credential | Type | Information requise |
|------------|------|-------------------|
| [Nom] | [Type] | [Ce qu'il faut demander] |

## Gestion d'erreurs
- [Strategy de retry]
- [Notifications en cas d'echec]
- [Fallback si applicable]

## Differences avec le workflow Make
| Aspect | Make | n8n | Raison |
|--------|------|-----|--------|
| [Aspect] | [Comportement Make] | [Comportement n8n] | [Pourquoi ce changement] |

## Validation utilisateur
- [ ] Plan valide par l'utilisateur
- Date de validation : [DATE]
