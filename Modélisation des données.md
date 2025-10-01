#  Modélisation des données (ER + dictionnaire PK/FK/Index)

## 1) Diagramme ER (synthèse)

<img width="2712" height="557" alt="image" src="https://github.com/user-attachments/assets/48297354-c0e8-40d8-8582-81c9b9104ac2" />


## 2) Dictionnaire des tables (PK, FK, Index)

### USER
- **PK:** `user_id`
- **Champs:** `email UNIQUE`, `name`, `photo_url`, `preferred_lang`, `roles`, `id_verified`, `reliability_score`, `reputation`, `created_at`
- **Index:** `email UNIQUE`

### SKILL
- **PK:** `skill_id`
- **FK:** `user_id → USER.user_id`
- **Champs:** `label`, `type {TEACH|LEARN}`, `level {BEGINNER|INTERMEDIATE|ADVANCED}`, `langs`
- **Index:** `(user_id)`, `label (TEXT)`, `(type, level)`, `(langs)`

### SESSION
- **PK:** `session_id`
- **FK:** `user_a_id → USER`, `user_b_id → USER`, `created_by → USER`
- **Champs:** `start_at`, `end_at`, `mode {remote|onsite}`, `place_point` **ou** `video_url`, `status {PROPOSED|CONFIRMED|DONE|CANCELLED|NO_SHOW}`, `translated_chat (bool)`
- **Index:** `(user_a_id, user_b_id, start_at)`, `(status, start_at)`, `place_point (2D/2dsphere si géo)`

### MESSAGE
- **PK:** `message_id`
- **FK:** `session_id → SESSION`, `sender_id → USER`
- **Champs:** `text`, `lang`, `audio_url?`, `file_url?`, `translated_text?`, `created_at`
- **Index:** `(session_id, created_at)`, `(sender_id, created_at)`

### REVIEW
- **PK:** `review_id`
- **FK:** `session_id → SESSION`, `reviewer_id → USER`, `reviewee_id → USER`
- **Champs:** `stars (1..5)`, `punctuality`, `pedagogy`, `engagement`, `communication`, `comment`, `created_at`
- **Contraintes:** `UNIQUE(session_id, reviewer_id)`
- **Index:** `(reviewee_id, created_at DESC)`, `(session_id)`

### GROUP
- **PK:** `group_id`
- **FK:** `admin_id → USER`
- **Champs:** `title`, `description`, `visibility`, `created_at`
- **Index:** `(admin_id)`, `(created_at DESC)`

### GROUP_MEMBERSHIP
- **PK:** `membership_id`
- **FK:** `group_id → GROUP`, `user_id → USER`
- **Champs:** `role {ADMIN|COADMIN|MEMBER}`, `joined_at`
- **Contraintes:** `UNIQUE(group_id, user_id)`
- **Index:** `(group_id, user_id)`, `(joined_at)`

### TASK
- **PK:** `task_id`
- **FK:** `group_id → GROUP`
- **Champs:** `title`, `description`, `status {TODO|IN_PROGRESS|DONE}`, `deadline`, `attachment_url?`, `created_at`
- **Index:** `(group_id, status, deadline)`

### BADGE
- **PK:** `badge_id`
- **Champs:** `code UNIQUE`, `name`, `description`, `icon`
- **Index:** `code UNIQUE`

### USER_BADGE
- **PK:** `user_badge_id`
- **FK:** `user_id → USER`, `badge_id → BADGE`
- **Champs:** `awarded_at`
- **Contraintes:** `UNIQUE(user_id, badge_id)`
- **Index:** `(user_id, awarded_at DESC)`

### USER_STATS
- **PK:** `user_id` (1–1 avec USER)
- **FK:** `user_id → USER`
- **Champs:** `counters JSON/colonnes`, `updated_at`

### IDENTITY_LINK
- **PK:** `identity_link_id`
- **FK:** `user_id → USER`
- **Champs:** `provider`, `provider_id`
- **Contraintes:** `UNIQUE(provider, provider_id)`

### PHONE_VERIF
- **PK:** `phone_verif_id`
- **FK:** `user_id → USER`
- **Champs:** `phone`
- **Contraintes:** `UNIQUE(phone)`

### MEDIA
- **PK:** `media_id`
- **FK:** `user_id → USER`
- **Champs:** `type {INTRO_VIDEO|IMAGE|DOC|AUDIO}`, `url`, `duration_sec?`, `mime?`, `size?`, `created_at`
- **Index:** `(user_id, created_at DESC)`, `(type, created_at DESC)`

### REPORT (signalement)
- **PK:** `report_id`
- **FK:** `reporter_id → USER`, `reported_user_id → USER`
- **Champs:** `target_type`, `target_id`, `reason`, `status {OPEN|IN_REVIEW|CLOSED}`, `created_at`, `updated_at`
- **Index:** `(status, created_at DESC)`, `(reported_user_id)`

### RANKING (si vous gardez la table)
- **PK:** `ranking_id`
- **FK:** `user_id → USER`
- **Champs:** `timeframe {DAILY|WEEKLY|MONTHLY|YEARLY|ALL_TIME}`, `role {TEACHER|LEARNER}`, `score`
- **Contraintes:** `UNIQUE(user_id, timeframe, role)`
- **Index:** `(timeframe, role, score DESC)`

### NOTIFICATION_LOG (utile pour traçabilité)
- **PK:** `notification_id`
- **FK:** `user_id → USER`
- **Champs:** `type {push|email}`, `status {SENT|RETRY|FAILED}`, `sent_at`
- **Index:** `(user_id, sent_at DESC)`, `TTL sur sent_at (si supporté)`

---

## 3) Normalisation (règles appliquées)

- **1NF :** attributs atomiques, listes externalisées en tables dédiées (ex. membres de groupe → `GROUP_MEMBERSHIP`).
- **2NF :** pas de dépendances partielles d’une PK composite (on évite les PK composites, ou on met une clé technique).
- **3NF :** aucun attribut non-clé ne dépend d’un autre attribut non-clé (ex. `Badge` séparé de `UserBadge`).

**Contraintes d’unicité :**
- `USER.email`, `REVIEW(session_id, reviewer_id)`,  
  `GROUP_MEMBERSHIP(group_id, user_id)`,  
  `BADGE.code`, `USER_BADGE(user_id, badge_id)`,  
  `IDENTITY_LINK(provider, provider_id)`, `PHONE_VERIF.phone`.

**Données lourdes :** fichiers/vidéos hors base (stockage objet), `MEDIA` = métadonnées.

**Index métier :**
- recherche prof/compétence (`SKILL.label` TEXT, `(type,level)`, `langs`),  
- proximité (`SESSION.place_point` géo),  
- chronologie (`MESSAGE(session_id, created_at)`),  
- réputation (`REVIEW(reviewee_id, created_at DESC)`).

#  SwapSkill — **Choix de la Technique de Réplication**


---

## 2) Choix de la Technique de Réplication

### 2.1 Exploration des techniques (performance ⚡ vs fiabilité 🔒)

| Technique | Principe | Avantages | Inconvénients / Risques | Cas d’usage |
|---|---|---|---|---|
| **Synchrone** | Le commit est confirmé après écriture sur ≥1 réplique | Cohérence forte, **RPO ≈ 0** | Latence ↑, dépend réseau, risque de blocage | Paiements, commandes |
| **Asynchrone** | Le leader confirme localement puis réplique en différé | Débit ↑, latence ↓, absorbe les pics | **RPO > 0** (perte récente possible), lectures stales | Chat, flux sociaux |
| **Semi‑synchrone (quorum)** | Commit après ack de *N* répliques | Bon compromis cohérence/latence | Complexité, latence > asynchrone | Données critiques non financières |
| **Multi‑leader** | Plusieurs leaders acceptent des écritures | Haute dispo géo, écriture locale | Conflits à résoudre, complexité | Multi‑régions à faible conflit |
| **Leader–Follower** | Un leader écrit, followers lisent | Simple, scale lecture | Écritures limitées au leader | Par défaut (Mongo RS, Postgres) |

> **RPO** = perte de données max tolérée ; **RTO** = temps de reprise max. Plus c’est bas, mieux c’est.

---

### 2.2 Décision pour SwapSkill (par domaine)

#### A) Domaine social — MongoDB (profils, skills, sessions, messages, reviews)
- **Modèle** : *Leader–Followers* (Replica Sets), évolutif en **sharding**.
- **Réplication** : **Asynchrone** par défaut.
- **Garanties ciblées** :
  - `messages` → *writeConcern* `w:1` (latence prioritaire), *readConcern* `local`.
  - `sessions`, `reviews` → `w:"majority", j:true`, *readConcern* `majority` (intégrité métier).
- **Bénéfice** : UX temps réel, volume élevé, tout en sécurisant les objets critiques.

#### B) Paiements/abonnements — PostgreSQL
- **Modèle** : **Primaire** + **Standby synchrone** (quorum = 1) + **Standby DR asynchrone** (multi‑région).
- **Réplication** : **Synchrone** (primaire ↔ standby AZ) + **Asynchrone** (site de secours).
- **Bénéfice** : **RPO ≈ 0** localement et continuité d’activité en cas d’incident régional.

---

### 2.3 Schéma de principe

<img width="658" height="992" alt="image" src="https://github.com/user-attachments/assets/6cac0d8e-550e-4943-a45d-f9a873f632f2" />


### 2.4 Impacts concrets attendus

- **Latence** : Chat P95 ↓ ; écritures `sessions/reviews` ↑ modérément (majority + journalisation).
- **Fiabilité** : Paiements tiennent la panne d’AZ sans perte ; Social rattrape via réplication différée.
- **Coût & Complexité** : Contrôlée (quorum limité côté PG ; write concerns ciblés côté Mongo).

---

### 2.5 Paramètres/bonnes pratiques (extraits)

**MongoDB (par collection)**  
- `messages` : `writeConcern: { w: 1 }`, `readPreference: primary` pour le flux actif.  
- `sessions`, `reviews` : `writeConcern: { w: "majority", j: true }`, `readConcern: "majority"`.

**PostgreSQL**  
- `synchronous_commit=on` ; `synchronous_standby_names='1 (pgsync1,pgsync2)'`.  
- WAL archiving + base backups pour **PITR** ; **pgbouncer/HAProxy** pour la bascule.

---

### 2.6 Synthèse (opinion)

- **Choix** : **Asynchrone contrôlé** pour le social (MongoDB) + **Synchrone** pour le financier (PostgreSQL).  
- **Raison** : maximise l’**expérience temps réel** tout en préservant la **cohérence critique**.  
- **Message clé** : **cohérence différenciée** selon la sensibilité de la donnée = meilleur compromis perf/fiabilité.

---


#  SwapSkill — **Configuration du Cluster — Choix et justification**

### 3.1 Choix par domaine
- **Domaine social (MongoDB : profils, skills, sessions, messages, reviews)**  
  **Mode : Actif–Actif (sharding + replica sets)**  
  **Pourquoi :** charge d’écriture/lecture élevée et distribuée, latence faible (chat), scalabilité horizontale, tolérance aux pannes par shard.

- **Paiements/abonnements (PostgreSQL)**  
  **Mode : Actif–Passif (primaire + standby synchrone + DR asynchrone)**  
  **Pourquoi :** cohérence stricte (RPO≈0) pour les transactions, bascule contrôlée et rapide.

---

### 3.2 Topologies de référence

#### A) MongoDB — Actif–Actif (sharding + replica sets)
<img width="1298" height="1048" alt="image" src="https://github.com/user-attachments/assets/f0834774-7039-4f16-a650-6b8bd5575f7e" />

* `==>|sync|` = réplication synchrone, `-.->|async|` = réplication asynchrone.

####  MongoDB — Actif–Actif (sharding + replica sets)
- **U → LB → M1/M2** : les clients passent par l’API/LB puis par deux routeurs *mongos*.
- **CS1–CS3** : serveurs de config (métadonnées du cluster). Les routeurs les consultent pour savoir où envoyer les requêtes.
- **Shard A / Shard B** : chaque shard est un **replica set** (**Primary** + **Secondaries**).
- **Écritures** : toujours sur le **Primary** du shard concerné (clé de sharding = répartition).
- **Lectures** : temps réel sur *primary* ; possibilité d’utiliser des *secondaries* pour du reporting.
- **Panne** : si un Primary tombe, une **élection** promeut un Secondary. Seul le shard touché est impacté.

  
#### B) PostgreSQL — Actif–Passif
<img width="2037" height="970" alt="image" src="https://github.com/user-attachments/assets/030ed23e-7d24-4c1b-87b7-e64bda10fd23" />


---
* `==>|sync|` = réplication synchrone, `-.->|async|` = réplication asynchrone.

####  PostgreSQL — Actif–Passif (primaire + sync standby + DR)
- **APP → pgbouncer/HAProxy → PG1** : l’application se connecte via un proxy au **Primary**.
- **PG2 (sync standby)** : chaque commit est validé après ACK de PG2 ⇒ **RPO≈0** en local.
- **PGDR (async)** : réplique distante pour **DR** (tolère un léger retard).
- **WAL Archive / S3** : archivage des journaux pour **PITR** (restaurations fines).
- **Panne** : le proxy bascule sur le standby promu ; l’ancien primaire est réintégré ensuite.
### 3.3 Pourquoi ces modes sont adaptés

| Domaine | Mode | Besoin clé | Justification |
|---|---|---|---|
| Social (MongoDB) | Actif–Actif | **Latence & scalabilité** | Partitionnement par usage, parallélisme, absorption des pics chat/recherche |
| Paiements (PostgreSQL) | Actif–Passif | **Cohérence & conformité** | Zéro perte locale (sync), bascule simple et testable, DR multi-région |

---

### 3.4 Comportement en fonctionnement & panne

**MongoDB (Actif–Actif)**  
- Normal : `mongos` répartit lectures/écritures par shard.  
- Panne primaire : élection < 15 s, reroutage automatique.  
- Panne shard : dégradation locale, le reste du système continue.

**PostgreSQL (Actif–Passif)**  
- Normal : commit après ack du standby synchrone (RPO≈0).  
- Panne primaire : promotion du standby via proxy ; réintégration ensuite.  
- Panne région : bascule pilotée sur DR (RPO = lag async).

---

### 3.5 Paramètres & bonnes pratiques

**MongoDB**  
- 3 nœuds data-bearing par replica set.  
- `writeConcern`: `w:1` (messages), `w:"majority", j:true` (sessions/reviews).  
- `readConcern`: `local` (temps réel), `majority` (critique).

**PostgreSQL**  
- `synchronous_commit=on` ; `synchronous_standby_names='1 (pgsync1,pgsync2)'`.  
- WAL archiving + backups (PITR).  
- pgbouncer/HAProxy + Patroni/pg_auto_failover.

---

### 3.6 Checklist

- Multi-AZ pour les nœuds critiques.  
- Quorum garanti sur chaque replica set Mongo.  
- ≥ 2 mongos et 3 config servers.  
- PG primaire + standby sync (AZ distinctes) + DR.  
- Runbooks : failover Mongo/PG, restauration PITR.  
- Monitoring : latence, replication lag, élections, erreurs WAL.

---

**Conclusion**  
- **Actif–Actif (MongoDB)** pour la partie sociale temps réel et scalable.  
- **Actif–Passif (PostgreSQL)** pour sécuriser les transactions avec **RPO≈0** et bascule maîtrisée.

  # ☁️ Haute Disponibilité — SwapSkill (Section 4)

#  SwapSkill — **Planification de la Haute Disponibilité**

### 4.1 Objectifs & principes
- **RPO/RTO** cibles : MongoDB ≤ 5 min / 10 min ; PostgreSQL ≈ 0 / 1–2 min.
- Multi‑AZ, réplication, bascule automatique, backups + **PITR**, observabilité, tests réguliers.

---

### 4.2 Gestion des failles — Schémas simples (Mermaid)

#### A) MongoDB — Perte d’un Primary (élection automatique)
<img width="1262" height="421" alt="image" src="https://github.com/user-attachments/assets/0a7d6a83-2454-4fa8-a44d-e2b219cc8c21" />

 le routeur envoie au Primary ; en cas de panne, un Secondary est élu **Nouveau Primary** et le trafic est rerouté.

---

#### B) MongoDB — Perte d’un shard (dégradation locale)
<img width="1272" height="477" alt="image" src="https://github.com/user-attachments/assets/579408d6-9d93-4100-b4f1-e9070f12701c" />

 si **Shard A** tombe, seules les données qu’il héberge sont impactées ; le reste du système fonctionne via **Shard B**.

---

#### C) PostgreSQL — Perte du Primary (promotion standby)
<img width="1260" height="412" alt="image" src="https://github.com/user-attachments/assets/abb9119a-4e7f-4f87-87e8-c6f6fd604369" />

 le Proxy envoie vers le Primary ; la réplication vers PG2 est synchrone (zéro perte locale), et vers PGDR asynchrone (site de secours).
---

### 4.3 Mesures clés (résumé)
- **MongoDB** : replica sets (3 nœuds), `writeConcern` adapté (w:1 vs w:"majority"), `readConcern` (local/majority), rolling upgrades.
- **PostgreSQL** : `synchronous_commit=on`, standby synchrone + DR asynchrone, WAL archiving, tests de failover et restauration.

### 4.4 Checklist
- Multi‑AZ ; quorum assuré ; ≥ 2 routeurs ; backups quotidiens + **PITR** ; runbooks à jour ; monitoring lag/latence/élections/erreurs WAL.
