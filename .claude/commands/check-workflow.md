---
description: Verifier et tester un workflow n8n existant. Usage: /project:check-workflow <nom-dossier-workflow>
---

Tu vas verifier le bon fonctionnement d'un workflow n8n migre depuis Make.com.

## Etape 0 : Pre-verification des fichiers

Verifie d'abord quels fichiers existent dans `workflows/$ARGUMENTS/` :
- Si `n8n-workflow.json` est ABSENT et qu'aucun workflow n8n n'existe : **STOP** — informe l'utilisateur que ce workflow n'a pas encore ete migre et suggere de lancer `/migrate $ARGUMENTS`
- Si `make-blueprint.json` est ABSENT : note-le et adapte l'etape 4 (pas de comparaison Make vs n8n possible, utilise le screenshot de reference ou le plan si disponible)
- Si `implementation_plan.md` est ABSENT : note-le et base la verification structurelle sur le workflow n8n seul

## Etape 1 : Chargement du contexte

1. Lis `workflows/$ARGUMENTS/implementation_plan.md` pour comprendre le workflow (si present)
2. Lis `workflows/$ARGUMENTS/make-blueprint.json` pour la reference Make originale (si present)
3. Recupere les details du workflow depuis n8n via MCP

## Etape 2 : Verification structurelle

Verifie que le workflow n8n contient :
- [ ] Tous les nodes decrits dans le plan d'implementation
- [ ] Les connexions entre nodes sont correctes
- [ ] Les credentials sont associees a chaque node qui en necessite
- [ ] Aucun community node n'est utilise
- [ ] Les expressions et mappings de donnees sont corrects

## Etape 3 : Tests d'execution

1. Execute le workflow en mode test via MCP n8n
2. Pour chaque node, verifie :
   - Le node s'execute sans erreur
   - Les donnees de sortie sont coherentes
   - Le format des donnees correspond a ce qu'attend le node suivant
3. Utilise les MCP disponibles pour verifier les resultats :
   - MCP Airtable : verifier que les records ont ete crees/modifies correctement
   - MCP Notion : verifier les pages/bases modifiees
4. Utilise dev-browser si besoin de verifier visuellement

## Etape 4 : Comparaison Make vs n8n

Compare les resultats du workflow n8n avec le comportement attendu du blueprint Make :
- Les memes donnees sont-elles traitees ?
- Les memes sorties sont-elles produites ?
- Les cas d'erreur sont-ils geres ?

## Etape 5 : Rapport

Mets a jour `workflows/$ARGUMENTS/verification_checklist.md` avec :
- Statut de chaque test (PASS / FAIL)
- Erreurs rencontrees et corrections appliquees
- Differences identifiees avec le workflow Make
- Statut final : VALIDE ou A CORRIGER

Si des corrections sont necessaires, applique-les et re-teste jusqu'a validation complete.
