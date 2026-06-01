---
description: Migrer un workflow Make.com vers n8n. Usage: /project:migrate <nom-dossier-workflow>
---

Tu es un expert en migration d'automatisations Make.com vers n8n. Tu vas migrer le workflow situe dans `workflows/$ARGUMENTS/`.

## Etape 0 : Pre-verification

Avant toute chose, verifie si ce workflow a deja ete migre :
1. Lis `docs/migration-tracker.md` et verifie le statut du workflow
2. Verifie la presence de `workflows/$ARGUMENTS/implementation_plan.md` et `workflows/$ARGUMENTS/n8n-workflow.json`
3. Si le workflow est deja migre (statut MIGRE ou n8n-workflow.json present) :
   - Analyse le n8n-workflow.json pour detecter d'eventuels problemes (nodes desactives, credentials manquantes, etc.)
   - Presente un resume de l'etat actuel avec les problemes trouves
   - Demande a l'utilisateur s'il veut : (a) corriger les problemes existants, (b) relancer la migration de zero, ou (c) lancer une verification avec `/check-workflow $ARGUMENTS`
   - **ATTENDS la reponse avant de continuer**
4. Si le workflow n'est pas migre, continue a l'etape 1

## Etape 1 : Analyse du blueprint Make

1. Lis le fichier `workflows/$ARGUMENTS/make-blueprint.json`

   **1.a — Blueprint absent du dossier** : si `workflows/$ARGUMENTS/make-blueprint.json` n'existe pas :
   - **Demande a l'utilisateur le nom exact de l'automatisation** (ou son ID de scenario Make).
   - Recupere le blueprint via le **MCP Make** (`mcp.make.com`) et sauvegarde-le sous `workflows/$ARGUMENTS/make-blueprint.json`, OU analyse directement le scenario dans Make via le MCP si l'export n'est pas disponible.
   - N'invente jamais la structure : sans blueprint local ni acces MCP, demande l'export a l'utilisateur. **ATTENDS sa reponse.**

   **1.b — Enrichir la comprehension de la source** : si le blueprint contient des elements non triviaux (expressions IML, routing/filtres, Iterator/Aggregator, modules custom ou *Make an API Call*), appuie-toi sur les skills `make-scenario-building` et `make-module-configuring` pour interpreter precisement la semantique avant de concevoir l'equivalent n8n. Objectif : comprendre la source, pas influencer le choix des nodes n8n.

2. Identifie :
   - Le declencheur (webhook, schedule, instant, etc.)
   - Chaque module Make et son equivalent n8n (consulte `docs/make-to-n8n-mapping.md` pour les equivalences modules + IML)
   - Les APIs externes appelees
   - Les transformations de donnees (filtres, routeurs, aggregateurs)
   - Les credentials necessaires
3. Si le workflow implique Airtable, utilise le MCP Airtable pour recuperer le schema des tables concernees
4. Si le workflow implique Notion, utilise le MCP Notion pour recuperer la structure
5. Utilise le MCP Context7 (`resolve-library-id` puis `query-docs`) pour verifier la documentation n8n de chaque node que tu envisages d'utiliser

## Etape 2 : Briefing utilisateur

Presente un resume clair :
- **Nom du workflow** : ...
- **Role** : description en 2-3 phrases de ce que fait cette automatisation
- **Declencheur** : type et frequence
- **Services connectes** : liste des APIs/services
- **Nombre de modules Make** : X modules
- **Complexite estimee** : faible / moyenne / elevee

Puis pose ces questions :
1. "Souhaites-tu une migration fidele (reproduction exacte) ou avec des ameliorations ?"
2. "Y a-t-il des problemes connus avec ce workflow Make ?"
3. "Y a-t-il des fonctionnalites a ajouter, modifier ou supprimer ?"
4. "Des preferences pour le declencheur (schedule, webhook, manuel) ?"

**ATTENDS la reponse de l'utilisateur avant de continuer.**

## Etape 3 : Planification

1. Lis `env.local` pour verifier les credentials disponibles
2. Verifie les credentials existantes dans n8n via MCP
3. Pour chaque node n8n prevu, consulte la doc via Context7 pour confirmer :
   - Le node existe en natif (PAS de community node)
   - Les parametres sont compatibles avec la derniere version
   - Les credentials requises
4. Cree le fichier `workflows/$ARGUMENTS/implementation_plan.md` en suivant le template `templates/implementation_plan.md`
5. Presente le plan a l'utilisateur et attends sa validation

## Etape 4 : Implementation

1. Cree le workflow dans n8n via le MCP n8n (ou API REST si MCP indisponible)
2. Configure chaque node un par un dans l'ordre du flux
3. Utilise UNIQUEMENT des nodes natifs n8n
4. Si un node natif n'existe pas pour un service, utilise le node HTTP Request
5. Associe les credentials n8n existantes — si une credential manque, demande-la a l'utilisateur et ajoute-la a `env.local`
6. Connecte les nodes entre eux selon le flux defini

## Etape 5 : Verification

1. Cree `workflows/$ARGUMENTS/verification_checklist.md` en suivant le template `templates/verification_checklist.md`
2. Execute le workflow en mode test via MCP
3. Verifie chaque node : entrees, sorties, erreurs
4. Teste les cas limites
5. Si erreur : corrige et re-teste
6. **Parite golden run** : recupere une execution Make connue (memes entrees) via le MCP Make ou un echantillon fourni, rejoue le workflow n8n sur les memes entrees, et compare la sortie (champs, ecritures Airtable/Notion/Freshdesk/HubSpot, emails). Valide sur 2+ iterations distinctes. Documente les ecarts voulus vs bugs.
7. Itere jusqu'a ce que TOUS les tests passent ET que la parite soit verifiee
8. Utilise dev-browser si besoin de verifier visuellement des resultats

## Etape 6 : Finalisation

1. Active le workflow dans n8n si c'est un schedule ou webhook
2. Mets a jour `docs/migration-tracker.md` avec le statut
3. Informe l'utilisateur que la migration est terminee avec un resume des differences vs Make

## Regles imperatives

- JAMAIS de community nodes
- JAMAIS generer un JSON a importer — toujours creer via MCP/API
- TOUJOURS verifier la doc Context7 avant d'utiliser un node
- TOUJOURS demander les credentials manquantes
- TOUJOURS faire valider le plan avant d'implementer
- Instance n8n AUTO-HEBERGEE (pas cloud)
