---
name: needs-digest-push
description: Production et publication du digest besoins clients Batch vers Notion (étapes 5–7 du needs-prioritizer)
---

## Objectif

Tu exécutes la **phase de publication** du workflow needs-prioritizer de Batch (étapes 5 à 7).

## Étape 0 — Lire le skill

Commence par lire le fichier d'instructions complet :
`/Users/mickael/Documents/Claude/.claude/skills/needs-prioritizer/SKILL.md`

Suis scrupuleusement toutes les instructions qu'il contient pour les étapes suivantes.

## Étape 1 — Récupérer les données de staging

Cherche dans Notion la page de staging la plus récente dont le titre commence par `🔄 STAGING — Besoins Collectés` dans la base "Digest Client Needs" (ID : `32956eb4-9a84-8162-bfc7-dd1a313124d5`).

Utilise `notion-search` puis `notion-fetch` pour lire son contenu complet.

Si aucune page de staging n'est trouvée : arrête-toi et notifie que la tâche `needs-digest-collect` doit être exécutée d'abord.

## Ce que tu dois exécuter

En suivant les instructions du SKILL.md et à partir des données de staging :

1. **Étape 5** — Produis le digest au format exact défini dans le skill.
2. **Étape 6** — Pousse le digest vers Notion (base "Digest Client Needs", titre `🤖 Digest Besoins Clients — [DATE]`).
3. **Étape 7** — Mets à jour le registre d'insights Batch dans Notion (accumulation, statuts, signal récurrent si 3 runs consécutifs).

## Nettoyage

Une fois terminé, renomme la page de staging en ajoutant `✅ TRAITÉ` au début de son titre.

## Résultat attendu

Retourne :
1. Le lien Notion vers le digest publié
2. Le lien Notion vers le registre d'insights mis à jour
3. Le Top 3 en 3 lignes max