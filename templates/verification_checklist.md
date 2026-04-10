# Checklist de verification : [NOM DU WORKFLOW]

## Informations
- **Workflow n8n ID** : [ID]
- **Date de test** : [DATE]
- **Iteration** : #[N]

## Tests structurels

| # | Verification | Statut | Notes |
|---|-------------|--------|-------|
| 1 | Tous les nodes presents selon le plan | [ ] | |
| 2 | Connexions entre nodes correctes | [ ] | |
| 3 | Credentials associees a chaque node | [ ] | |
| 4 | Aucun community node utilise | [ ] | |
| 5 | Expressions et mappings corrects | [ ] | |
| 6 | Error handling configure | [ ] | |

## Tests d'execution

| # | Node | Entree OK | Sortie OK | Erreurs | Notes |
|---|------|-----------|-----------|---------|-------|
| 1 | [Node 1] | [ ] | [ ] | | |
| 2 | [Node 2] | [ ] | [ ] | | |

## Tests cas limites

| # | Scenario | Resultat attendu | Resultat obtenu | Statut |
|---|----------|-------------------|-----------------|--------|
| 1 | Donnees vides | [Comportement] | [Resultat] | [ ] |
| 2 | Erreur API externe | [Comportement] | [Resultat] | [ ] |
| 3 | Timeout | [Comportement] | [Resultat] | [ ] |
| 4 | Donnees malformees | [Comportement] | [Resultat] | [ ] |

## Comparaison Make vs n8n

| Critere | Make (attendu) | n8n (observe) | Conforme |
|---------|---------------|---------------|----------|
| Donnees traitees | [Description] | [Description] | [ ] |
| Sorties produites | [Description] | [Description] | [ ] |
| Gestion erreurs | [Description] | [Description] | [ ] |
| Performance | [Description] | [Description] | [ ] |

## Erreurs et corrections

| # | Erreur | Cause | Correction | Re-test OK |
|---|--------|-------|------------|------------|

## Statut final

- [ ] **VALIDE** — Le workflow fonctionne correctement
- [ ] **A CORRIGER** — Des problemes subsistent (voir erreurs ci-dessus)

### Notes de finalisation
[Observations, recommandations, points d'attention pour la maintenance]
