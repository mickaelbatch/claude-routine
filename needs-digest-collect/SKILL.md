---
name: needs-digest-collect
description: Besoins clients hebdo — lundi 14h : collecte #ask-product, log Notion, message #team-pm
---

## Objectif

Chaque lundi, analyser les remontées de la semaine passée dans #ask-product et produire le digest besoins clients : page Notion + mise à jour du Registre + message #team-pm.

---

## Suivi d'exécution — OBLIGATOIRE

À chaque étape, note le statut dans un journal interne :
- ✅ Étape X — succès (résumé en 1 ligne)
- ⚠️ Étape X — succès partiel (ce qui manque)
- ❌ Étape X — échec (cause précise)

Ce journal doit être inclus dans la page Notion (section "Rapport d'exécution") ET dans le message Slack #team-pm, juste après le tableau des besoins. Si une étape critique échoue (1, 5, 6, 7), le signaler clairement en tête du message Slack avec 🚨.

---

## Étape 1 — Lecture de #ask-product

Lis le canal #ask-product (ID : C0410AZ345C) sur les 7 derniers jours avec `slack_read_channel`.
Calcule dynamiquement le timestamp "7 jours avant aujourd'hui" — ne jamais coder en dur une date.
Pour chaque message avec `reply_count > 0`, lis le thread avec `slack_read_thread`.

Tu cherches : demandes de features, frictions, besoins clients remontés par CSMs/SEs/Customer Care.
Ignore : bugs techniques isolés, questions config SDK, billing.

**En cas d'échec :** note ❌ Étape 1 dans le journal. C'est une étape critique — signale-le en tête du message Slack avec 🚨 et arrête le run.

---

## Étape 1b — Lecture des conversations Intercom via Metabase

Interroge Metabase en **deux requêtes** pour récupérer les conversations Intercom des 7 derniers jours ayant le tag "Topic - Product feedback".

### Requête 1 — Contenu des conversations (database_id: 11)

```sql
SELECT p.CONVERSATION_ID, p.BODY, p.CREATED_AT, p.AUTHOR_TYPE
FROM INTERCOM.STG_INTERCOM__CONVERSATION_PART_HISTORY p
JOIN INTERCOM.STG_INTERCOM__CONVERSATION_TAG_HISTORY t 
  ON p.CONVERSATION_ID = t.CONVERSATION_ID
WHERE t.TAG_ID = '3378237'
AND p.CREATED_AT >= DATEADD(day, -7, CURRENT_DATE())
ORDER BY p.CREATED_AT DESC
```

- `TAG_ID = '3378237'` correspond au tag "Topic - Product feedback" dans `INTERCOM.STG_INTERCOM__TAGS`
- La colonne `BODY` contient le texte brut des messages, dont les blocs "Note Productboard"
- `AUTHOR_TYPE` permet de distinguer les messages clients des messages agents

### Requête 2 — Noms de clients (database_id: 10)

Pour chaque `CONVERSATION_ID` distinct retourné par la requête 1, récupère le nom du client :

```sql
SELECT CONVERSATION_ID, COMPANY_NAME, COMPANY_ID
FROM BATCH_LLM_ACCESS.FACT_INTERCOM_CONVERSATIONS
WHERE CONVERSATION_ID IN (<liste_des_ids>)
```

### Traitement des résultats

Pour chaque conversation :
1. Regroupe tous les `BODY` par `CONVERSATION_ID` pour reconstruire le fil de la conversation
2. Cherche un bloc commençant par **"Note Productboard"** dans les BODY
3. Si trouvé, extrais :
   - Le contenu de la demande (ce qui suit "Note Productboard")
   - Le nom du client (via la requête 2)
   - La présence ou non du mot **"critical"** (case-insensitive) n'importe où dans la conversation
4. Si aucun bloc "Note Productboard" → ignore la conversation

Ces signaux Intercom viennent s'ajouter aux signaux Slack de l'étape 1 pour le regroupement suivant.

**En cas d'échec :** note ❌ Étape 1b dans le journal avec la cause précise. Continue le run avec les signaux Slack uniquement — ne pas bloquer. Le journal devra mentionner explicitement "Intercom non inclus ce run".

---

## Étape 2 — Regroupement par besoin

Regroupe les signaux par besoin distinct. Ce qu'on compte : le **nombre de clients distincts** demandant ce besoin (pas le nombre de messages).
- Même client mentionné plusieurs fois pour le même besoin = 1 client
- Normalise chaque besoin en phrase courte à l'infinitif

Pour chaque groupe : nombre de clients distincts, liste des noms de clients, **un verbatim par client demandeur** (tous les verbatims, pas un seul global).

**En cas d'absence totale de signaux :** note ⚠️ Étape 2 — "Aucun signal identifié cette semaine". Continue jusqu'à l'étape 7 pour le signaler dans Notion et Slack.

---

## Étape 3 — Catégorisation

Assigne une Catégorie et une Squad à chaque besoin :

| Catégorie | Squad |
|-----------|-------|
| Segmentation / Ciblage | Profile |
| Automations / Builder | Engage |
| Campaigns | Engage |
| Email | Messaging |
| Push / In-App | Messaging |
| SMS | Messaging |
| Analytics / Rapports | Engage |
| Data / Intégrations | Profile |
| Permissions / Accès | Profile |
| API / Webhooks | Messaging |
| Dashboard UX | Engage |

---

## Étape 4 — Classement

Structure le tableau en deux blocs :

**Bloc 1 — ⚠️ À traiter en priorité**
Tous les besoins ayant au moins un signal Intercom avec niveau = `critical` (3), quel que soit leur nb de clients. Classés entre eux par nb de clients distincts décroissant, puis par compte stratégique en cas d'égalité.

**Bloc 2 — 📋 Autres besoins**
Tous les autres besoins (niveaux nice to have, important, ou sans niveau). Classés par :
1. Nb de clients distincts décroissant
2. Niveau max décroissant (Important avant Nice to have)
3. Compte stratégique en cas d'égalité persistante

Les comptes stratégiques pour le tiebreaker : Orange, Sephora, ManoMano, Leboncoin, Le Monde, BNP, Betclic, RATP, FNAC, Decathlon, La Redoute, SNCF.

---

## Étape 5 — Log dans Notion (page digest)

Crée une nouvelle page dans Digest Client Needs (ID : 32956eb4-9a84-8162-bfc7-dd1a313124d5) avec :
- **Titre** : `Besoins clients — semaine du [DATE]`
- **Contenu** : deux blocs séparés par un titre `⚠️ À traiter en priorité` et `📋 Autres besoins`, chacun avec un tableau colonnes Besoin / Catégorie / Clients / Clients demandeurs, suivi des verbatims par besoin
- **Section finale obligatoire** : `## Rapport d'exécution` avec le journal complet des étapes (✅ / ⚠️ / ❌)

**En cas d'échec :** note ❌ Étape 5. C'est une étape critique — signale-le avec 🚨 dans le message Slack.

---

## Étape 6 — Mise à jour du Registre Besoins Clients

Base Notion ID : `f91db26a03064283af6eac6ed1a59592`
Data source ID : `6aa96d07-342b-4a15-9d5a-76b08e8de9fb`

Pour chaque besoin identifié :

1. Cherche s'il existe déjà via `notion-search` (correspondance sur le sens, pas l'orthographe exacte)
2. **S'il existe** → `notion-update-page` :
   - `Clients (total)` ← incrémenter avec les nouveaux clients non déjà comptabilisés
   - `Clients demandeurs` ← ajouter les nouveaux noms (sans doublons)
   - `Verbatim` ← remplacer par la liste complète des verbatims de cette semaine, un par client demandeur (format : "**[Client]** : [verbatim]")
   - `Date dernière remontée` ← date d'aujourd'hui (run courant)
   - `Niveau max` ← niveau le plus élevé parmi tous les signaux de ce besoin cette semaine (Critical > Important > Nice to have). Ne pas écraser si la valeur existante est plus haute que celle de cette semaine.
   - Ne pas toucher au Statut si "En roadmap" ou "Livré"
3. **S'il est nouveau** → `notion-create-pages` avec toutes les propriétés, Statut = Actif, `Date dernière remontée` = aujourd'hui, `Niveau max` = niveau le plus élevé parmi les signaux de cette semaine (ou vide si aucun niveau renseigné)

**En cas d'échec partiel :** note ⚠️ Étape 6 avec la liste des besoins non mis à jour. Continuer pour les autres.
**En cas d'échec total :** note ❌ Étape 6. Signaler avec 🚨 dans le message Slack.

---

## Étape 7 — Message dans #team-pm

Envoie sans demander confirmation dans #team-pm (ID : G01J0BZHWQL) :

```
[🚨 Échecs critiques : Étape X, Étape Y — si applicable, sinon ne pas afficher cette ligne]

📋 *Besoins clients — semaine du [DATE]*
Source : #ask-product + Intercom | [N] besoins · [M] clients distincts

⚠️ *À traiter en priorité*
| # | Besoin | Catégorie | Clients | Clients demandeurs |
|---|--------|-----------|---------|-------------------|
[besoins avec au moins un signal critical — vide si aucun]

📋 *Autres besoins*
| # | Besoin | Catégorie | Clients | Clients demandeurs |
|---|--------|-----------|---------|-------------------|
[reste des besoins]

🔗 [lien Notion vers la page digest]

---
*Rapport d'exécution :*
✅ Étape 1 — #ask-product : X messages, Y threads lus
[✅ / ⚠️ / ❌] Étape 1b — Intercom : [résumé ou cause d'échec]
✅ Étape 2 — Z besoins identifiés
✅ Étape 3-4 — Catégorisation et classement OK
[✅ / ❌] Étape 5 — Page Notion créée
[✅ / ⚠️ / ❌] Étape 6 — Registre mis à jour
```

Si le bloc "À traiter en priorité" est vide, ne pas l'afficher.

**En cas d'échec d'envoi :** c'est une étape critique — l'information doit absolument parvenir à Mickaël. Si le message Slack échoue, créer quand même la page Notion (si pas déjà fait) et noter l'échec dans la page.

---

## Règles

- On compte des **clients distincts**, pas des messages.
- Un client présent dans Slack ET dans Intercom pour le même besoin = 1 client (pas de double-comptage).
- Les besoins avec au moins un signal "critical" (niveau 3) sont épinglés dans un bloc séparé en tête, quel que soit leur volume.
- Les niveaux "important" (2) et "nice to have" (1) n'ont pas de traitement spécial sur le ranking — ils restent dans le bloc principal.
- Si le bloc "À traiter en priorité" est vide cette semaine, ne pas l'afficher dans Notion ni Slack.
- Si peu de signaux cette semaine, le dire clairement plutôt que gonfler.
- **Une étape qui échoue silencieusement est pire qu'une étape qui signale son échec.** Toujours reporter explicitement.
- Langue : français.