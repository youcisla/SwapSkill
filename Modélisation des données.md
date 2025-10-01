# 📚 Modélisation des données (ER + dictionnaire PK/FK/Index)

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

