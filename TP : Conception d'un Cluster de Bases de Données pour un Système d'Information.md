<!-- prettier-ignore-start -->
<p align="center">
  <h1 align="center">SwapSkill — Modélisation & Réplication & HA</h1>
  <p align="center">
    <em>ER + dictionnaire PK/FK/Index • Réplication • Configuration du cluster • Haute dispo</em>
 
</p>
<!-- prettier-ignore-end -->

---

##  Table des matières
- [1) Modélisation des données](#-1-modélisation-des-données-er--dictionnaire-pkfkindex)
- [2) Choix de la technique de réplication](#-2-choix-de-la-technique-de-réplication)
- [3) Configuration du cluster](#-3-configuration-du-cluster--choix-et-justification)
- [4) Haute disponibilité](#-4-haute-disponibilité--swapskill-section-4)

---

##  1) Modélisation des données (ER + dictionnaire PK/FK/Index)

### 1.1 Diagramme ER (synthèse)
<p align="center">
  <img width="2712" height="557" alt="ER Diagram" src="https://github.com/user-attachments/assets/48297354-c0e8-40d8-8582-81c9b9104ac2" />
</p>

### 1.2 Dictionnaire des tables (PK, FK, Index)

> **Note** : FK réelles en SQL, références logiques en MongoDB.

#### USER
- **PK:** `user_id`  
- **Champs:** `email UNIQUE`, `name`, `photo_url`, `preferred_lang`, `roles`, `id_verified`, `reliability_score`, `reputation`, `created_at`  
- **Index:** `email UNIQUE`

#### SKILL
- **PK:** `skill_id` — **FK:** `user_id → USER.user_id`  
- **Champs:** `label`, `type {TEACH|LEARN}`, `level {BEGINNER|INTERMEDIATE|ADVANCED}`, `langs`  
- **Index:** `(user_id)`, `label (TEXT)`, `(type, level)`, `(langs)`

#### SESSION
- **PK:** `session_id` — **FK:** `user_a_id → USER`, `user_b_id → USER`, `created_by → USER`  
- **Champs:** `start_at`, `end_at`, `mode {remote|onsite}`, `place_point` **ou** `video_url`, `status {PROPOSED|CONFIRMED|DONE|CANCELLED|NO_SHOW}`, `translated_chat (bool)`  
- **Index:** `(user_a_id, user_b_id, start_at)`, `(status, start_at)`, `place_point (2D/2dsphere si géo)`

#### MESSAGE
- **PK:** `message_id` — **FK:** `session_id → SESSION`, `sender_id → USER`  
- **Champs:** `text`, `lang`, `audio_url?`, `file_url?`, `translated_text?`, `created_at`  
- **Index:** `(session_id, created_at)`, `(sender_id, created_at)`

#### REVIEW
- **PK:** `review_id` — **FK:** `session_id → SESSION`, `reviewer_id → USER`, `reviewee_id → USER`  
- **Champs:** `stars (1..5)`, `punctuality`, `pedagogy`, `engagement`, `communication`, `comment`, `created_at`  
- **Contraintes:** `UNIQUE(session_id, reviewer_id)`  
- **Index:** `(reviewee_id, created_at DESC)`, `(session_id)`

#### GROUP
- **PK:** `group_id` — **FK:** `admin_id → USER`  
- **Champs:** `title`, `description`, `visibility`, `created_at`  
- **Index:** `(admin_id)`, `(created_at DESC)`

#### GROUP_MEMBERSHIP
- **PK:** `membership_id` — **FK:** `group_id → GROUP`, `user_id → USER`  
- **Champs:** `role {ADMIN|COADMIN|MEMBER}`, `joined_at`  
- **Contraintes:** `UNIQUE(group_id, user_id)`  
- **Index:** `(group_id, user_id)`, `(joined_at)`

#### TASK
- **PK:** `task_id` — **FK:** `group_id → GROUP`  
- **Champs:** `title`, `description`, `status {TODO|IN_PROGRESS|DONE}`, `deadline`, `attachment_url?`, `created_at`  
- **Index:** `(group_id, status, deadline)`

#### BADGE
- **PK:** `badge_id`  
- **Champs:** `code UNIQUE`, `name`, `description`, `icon`  
- **Index:** `code UNIQUE`

#### USER_BADGE
- **PK:** `user_badge_id` — **FK:** `user_id → USER`, `badge_id → BADGE`  
- **Champs:** `awarded_at`  
- **Contraintes:** `UNIQUE(user_id, badge_id)`  
- **Index:** `(user_id, awarded_at DESC)`

#### USER_STATS
- **PK:** `user_id` (1–1 avec USER) — **FK:** `user_id → USER`  
- **Champs:** `counters JSON/colonnes`, `updated_at`

#### IDENTITY_LINK
- **PK:** `identity_link_id` — **FK:** `user_id → USER`  
- **Champs:** `provider`, `provider_id`  
- **Contraintes:** `UNIQUE(provider, provider_id)`

#### PHONE_VERIF
- **PK:** `phone_verif_id` — **FK:** `user_id → USER`  
- **Champs:** `phone`  
- **Contraintes:** `UNIQUE(phone)`

#### MEDIA
- **PK:** `media_id` — **FK:** `user_id → USER`  
- **Champs:** `type {INTRO_VIDEO|IMAGE|DOC|AUDIO}`, `url`, `duration_sec?`, `mime?`, `size?`, `created_at`  
- **Index:** `(user_id, created_at DESC)`, `(type, created_at DESC)`

#### REPORT (signalement)
- **PK:** `report_id` — **FK:** `reporter_id → USER`, `reported_user_id → USER`  
- **Champs:** `target_type`, `target_id`, `reason`, `status {OPEN|IN_REVIEW|CLOSED}`, `created_at`, `updated_at`  
- **Index:** `(status, created_at DESC)`, `(reported_user_id)`

#### RANKING (si utilisé)
- **PK:** `ranking_id` — **FK:** `user_id → USER`  
- **Champs:** `timeframe {DAILY|WEEKLY|MONTHLY|YEARLY|ALL_TIME}`, `role {TEACHER|LEARNER}`, `score`  
- **Contraintes:** `UNIQUE(user_id, timeframe, role)`  
- **Index:** `(timeframe, role, score DESC)`

#### NOTIFICATION_LOG (traçabilité)
- **PK:** `notification_id` — **FK:** `user_id → USER`  
- **Champs:** `type {push|email}`, `status {SENT|RETRY|FAILED}`, `sent_at`  
- **Index:** `(user_id, sent_at DESC)`, `TTL sur sent_at (si supporté)`

---

##  2) Choix de la Technique de Réplication

### 2.1 Panorama (perf ⚡ vs fiabilité 🔒)
| Technique | Principe | Avantages | Limites | Cas d’usage |
|---|---|---|---|---|
| **Synchrone** | Commit confirmé après écriture sur ≥1 réplique | Cohérence forte, **RPO≈0** | Latence ↑, dépend réseau | Paiements |
| **Asynchrone** | Commit local puis réplication différée | Débit ↑, latence ↓ | **RPO>0**, lectures stales | Chat/flux |
| **Semi‑synchrone** | Quorum d’ACK | Compromis cohérence/latence | Complexité | Données critiques |
| **Multi‑leader** | Plusieurs leaders | Haute dispo géo | Conflits | Multi‑régions |
| **Leader–Follower** | 1 leader écrit | Simple, scale lecture | Écritures centralisées | Par défaut |

### 2.2 Décision (par domaine)
- **Social (MongoDB)** : **Asynchrone** + write concerns différenciés (`w:1` pour `messages`, `w:"majority", j:true` pour `sessions/reviews`).  
- **Paiements (PostgreSQL)** : **Synchrone** local (primary↔standby) + **Asynchrone** DR multi‑région.

### 2.3 Schéma de principe
<p align="center">
  <img width="658" height="992" alt="Replication Diagram" src="https://github.com/user-attachments/assets/6cac0d8e-550e-4943-a45d-f9a873f632f2" />
</p>

---

##  3) Configuration du Cluster — Choix et justification

### 3.1 Choix par domaine
- **MongoDB (social)** : **Actif–Actif** (sharding + replica sets) — latence faible, échelle horizontale, pannes isolées par shard.  
- **PostgreSQL (paiements)** : **Actif–Passif** (primary + sync standby + DR async) — **RPO≈0**, bascule maîtrisée.

### 3.2 Topologies (illustrations)
<p align="center">
  <img width="1298" height="1048" alt="Mongo Cluster" src="https://github.com/user-attachments/assets/f0834774-7039-4f16-a650-6b8bd5575f7e" />
</p>

<p align="center">
  <img width="2037" height="970" alt="Postgres Cluster" src="https://github.com/user-attachments/assets/030ed23e-7d24-4c1b-87b7-e64bda10fd23" />
</p>

### 3.3 Pourquoi c’est adapté
| Domaine | Mode | Besoin clé | Justification |
|---|---|---|---|
| MongoDB | Actif–Actif | Latence & scalabilité | Partitionnement, parallélisme, pics chat/recherche |
| PostgreSQL | Actif–Passif | Cohérence & conformité | Zéro perte locale (sync), DR testable |

---

##  4) Haute Disponibilité — SwapSkill (Section 4)

### 4.1 Objectifs & principes
- **RPO/RTO** : MongoDB ≤ 5 min / 10 min ; PostgreSQL ≈ 0 / 1–2 min.  
- Multi‑AZ, réplication, bascule auto, backups + **PITR**, observabilité, tests réguliers.

### 4.2 Gestion des failles — schémas & explications
**MongoDB — Perte d’un Primary** : le routeur envoie au Primary ; si panne, un Secondary est élu **Nouveau Primary** et le trafic est rerouté.  
<p align="center">
  <img width="1262" height="421" alt="Mongo Primary Failover" src="https://github.com/user-attachments/assets/0a7d6a83-2454-4fa8-a44d-e2b219cc8c21" />
</p>

**MongoDB — Perte d’un shard** : si **Shard A** tombe, seules ses données sont impactées ; le reste continue via **Shard B**.  
<p align="center">
  <img width="1272" height="477" alt="Mongo Shard Loss" src="https://github.com/user-attachments/assets/579408d6-9d93-4100-b4f1-e9070f12701c" />
</p>

**PostgreSQL — Perte du Primary** : le proxy envoie vers le **Standby synchrone** promu **Primary** ; DR reste disponible en async.  
<p align="center">
  <img width="1260" height="412" alt="Postgres Failover" src="https://github.com/user-attachments/assets/abb9119a-4e7f-4f87-87e8-c6f6fd604369" />
</p>

### 4.3 Mesures clés
- **MongoDB** : replica sets (3 nœuds), `writeConcern` (w:1 vs w:"majority"), `readConcern` (local/majority), rolling upgrades.  
- **PostgreSQL** : `synchronous_commit=on`, standby synchrone + DR asynchrone, WAL archiving, tests de failover & restauration.

### 4.4 Checklist
- Multi‑AZ ; quorum assuré ; ≥ 2 routeurs ; backups quotidiens + **PITR** ; runbooks à jour ; monitoring lag/latence/élections/erreurs WAL.
