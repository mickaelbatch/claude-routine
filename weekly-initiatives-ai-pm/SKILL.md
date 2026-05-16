---
name: weekly-initiatives-ai-pm
description: Tous les vendredis 17h : ajoute dans la DB Notion "Initiatives AI" les nouvelles initiatives GenAI internes des PMs de la semaine.
---

Chaque vendredi, identifie et log dans Notion les nouvelles initiatives GenAI internes des PMs de la semaine.

## Contexte
Tu surveilles les 5 personnes suivantes (PMs + CPO) : Mickaël Bentz, Alaïs Pluyaud, Claire Zunda, Céline Aschenbrenner, Antoine (CPO).
Base Notion cible : https://www.notion.so/33356eb49a8480e29a74ef65cdc66fac (Initiatives AI x PM)

## Étape 1 — Calcul de la fenêtre
Calcule le lundi et le vendredi de la semaine en cours (ou semaine passée si run le vendredi après 17h).

## Étape 2 — Scan Slack
Cherche les messages des 5 personnes sur les canaux suivants pendant la fenêtre calculée :
- #team-pm (G01J0BZHWQL)
- #team-product
- #ask-genai
- #use-case-genai
- #veille-genai
- #team-genai-referents

Utilise slack_search_public_and_private avec des queries ciblées par personne + période.

## Étape 3 — Filtrage : qu'est-ce qui compte comme initiative ?

**✅ Compte comme initiative concrète (à logger) :**
- Création ou partage d'une Claude skill → tout message contenant un lien `claude.ai/customize/skills` ou `claude.ai` vers un outil = initiative concrète, MÊME SI le message contient des réserves comme "faudrait itérer", "pas encore finalisé", "work in progress". Le lien prouve que quelque chose a été créé.
- Création d'une scheduled task / routine automatisée
- Création ou configuration d'un MCP
- Création d'un artifact, document, ou template GenAI réutilisable
- Mise en place d'un workflow GenAI (prompt + outil combinés)
- Partage d'un gem / skill depuis une autre plateforme AI

**❌ Ne compte PAS comme initiative :**
- Simple question sur un outil AI
- Partage d'un article ou lien externe sans création propre
- Proposition ou idée sans lien ni livrable ("ça vaudrait pas la peine de faire une skill X ?" sans lien = exploration, pas initiative)
- Feature produit Batch shippée (ce sont des livrables métier, pas des initiatives internes AI)
- Réaction emoji uniquement

**Règle clé :** La présence d'un lien vers un outil créé (skill, artifact, doc, script) prime sur le langage hésitant. Si un lien existe → initiative.

## Étape 4 — Déduplicate
Récupère les pages existantes dans la DB Notion. Compare sur Owner + Use case + semaine pour éviter les doublons.

## Étape 5 — Création Notion
Pour chaque nouvelle initiative, crée une page dans la DB avec :
- **Use case** : titre court de l'initiative
- **Owner** : prénom du PM
- **Détails et ROI** : description de ce que ça fait + valeur estimée
- **Lien** : URL vers la skill / artifact / tool si disponible
- **Date** : date du message Slack

## Étape 6 — Récap Slack
Poste un message dans #team-pm (G01J0BZHWQL) avec la liste des initiatives loggées cette semaine.
Format : "📊 Initiatives AI PM – semaine du [lun] au [ven] : [liste]"
Si rien de nouveau → ne poste rien (silencieux).