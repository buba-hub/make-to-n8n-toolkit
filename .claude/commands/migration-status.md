---
description: Afficher le statut de toutes les migrations Make vers n8n
---

Affiche un tableau de bord complet de l'avancement des migrations.

## Instructions

1. Lis `docs/migration-tracker.md` pour l'etat enregistre
2. Liste tous les dossiers dans `workflows/` pour detecter d'eventuels nouveaux workflows
3. Pour chaque workflow, verifie :
   - Presence de `make-blueprint.json` (source Make)
   - Presence de `implementation_plan.md` (planification faite)
   - Presence de `verification_checklist.md` (tests realises)
   - Statut dans le tracker

4. Affiche le tableau de bord au format :

```
| Workflow                      | Blueprint | Plan | Impl. | Tests | Statut      |
|-------------------------------|-----------|------|-------|-------|-------------|
| import-lots                   | OK        | OK   | OK    | OK    | MIGRE       |
| sprint-creation               | OK        | -    | -     | -     | A MIGRER    |
| ...                           |           |      |       |       |             |
```

5. Affiche des statistiques :
   - Total workflows : X
   - Migres : X
   - En cours : X
   - A migrer : X
   - Progres global : X%

6. Verifie activement les incoherences entre le tracker et les fichiers reels :
   - Un workflow marque MIGRE sans `verification_checklist.md` → signaler que les tests n'ont pas ete documentes
   - Un workflow avec `n8n-workflow.json` mais pas dans le tracker → signaler l'oubli
   - Un workflow dans le tracker mais sans dossier correspondant → signaler le dossier manquant
   - Des fichiers inattendus dans un dossier → mentionner

7. Si des incoherences sont trouvees, propose les corrections specifiques avec un diff avant/apres du tracker. Demande a l'utilisateur s'il veut appliquer les corrections.
