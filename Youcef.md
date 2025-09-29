# SwapSkill

---

## 1. Imaginer une Application

- **Nom de l'Application :** SkillSwap  
- **Type :** Application mobile sociale et collaborative d’échange de compétences (apprendre/enseigner) avec gamification, groupes et classe virtuelle.  

---

## 2. Décrire l'Application

### Objectif
Faciliter l’apprentissage accessible, communautaire et motivant en mettant en relation des personnes qui souhaitent **enseigner** ou **apprendre** des compétences.  
Le tout avec un système **d’échange de savoirs**, de **classements**, et de **groupes collaboratifs** pour encourager la régularité et l’engagement.

### Problèmes résolus
- Coût élevé et rigidité des cours traditionnels.  
- Manque de structure pour des apprentissages collaboratifs.  
- Manque de motivation sur le long terme.  
- Besoin de reconnaissance et de progression visible.

### Fonctionnalités principales
- **Profils & Compétences :**
  - Création de profil (bio, photo, localisation approximative).
  - Déclaration “J’enseigne / J’apprends” (compétence, niveau, disponibilité).
- **Matching intelligent :**
  - Suggestions basées sur les compétences communes, la distance, la disponibilité et la réputation.
  - Filtres (rayon, niveau, langue, présentiel/distanciel).
- **Messagerie & Sessions :**
  - Chat en temps réel (1–1 ou groupe).
  - Proposition/confirmation de sessions (date, lieu/visio).
  - Rappels automatiques.
- **Groupes collaboratifs :**
  - Création de groupes par enseignants ou apprenants.
  - Rôles (admin, co-admin, membres).
  - Forums et fils de discussion.
- **Classe virtuelle :**
  - **Visioconférences intégrées** (WebRTC / Zoom SDK).
  - Partage d’écran et tableau blanc collaboratif.
  - Possibilité de créer des sous-salles (breakout rooms).
- **Partage & organisation :**
  - Assignation de tâches et de devoirs (avec deadline).
  - Upload/partage de tout type de fichier.
  - Création de sondages et votes collaboratifs.
- **Ranking & Gamification :**
  - Classements enseignants et apprenants (quotidien, hebdomadaire, mensuel, annuel, “All time”).
  - Attribution de points pour chaque action (enseigner, participer, aider, publier du contenu).
  - **Badges** pour récompenser rôles et comportements.
  - Système de **niveaux** (gamifié).
- **Personnalisation & Communauté :**
  - Suggestions de groupes ou de pairs selon l’historique d’apprentissage.
  - Section **“Proposer une nouvelle fonctionnalité”** votable par la communauté.
  - Notifications personnalisées (push/email).
- **Sécurité & Modération :**
  - Auth JWT, vérification email/téléphone.
  - Signalement/blocage d’utilisateur.
  - Paramètres de confidentialité (visibilité profil, localisation approximative).

### Utilité
- **Pour les enseignants :** visibilité, reconnaissance via le classement, gestion de groupes & sessions.  
- **Pour les apprenants :** apprentissage gratuit et flexible, progression visible, appartenance à une communauté motivante.  
- **Pour tous :** réseau social du savoir, où l’entraide est valorisée.

---

## 3. Schéma ou Diagramme

### Architecture
```mermaid
flowchart LR
  App[App Mobile]
  API[API REST\nAuth JWT, validation]
  WS[Temps réel]
  DB[(MongoDB Atlas)]
  Push[Notifications Push]
  Maps[Localisation & GPS]
  Video[Serveur WebRTC/Zoom API]
  Files[Stockage fichiers]

  App -- HTTPS/JSON --> API
  API --- DB
  App <-- WebSocket --> WS
  App -. Push .-> Push
  App -. GPS/Map .-> Maps
  App <-- Stream --> Video
  App <-- Upload/Download --> Files
````

### Modèle de données simplifié

```mermaid
erDiagram
  USER ||--o{ SKILL : possede
  USER ||--o{ SESSION : participe
  USER ||--o{ REVIEW : evalue
  SESSION ||--o{ MESSAGE : contient
  USER ||--o{ GROUP : membre
  GROUP ||--o{ TASK : contient
  USER ||--o{ BADGE : gagne
  USER ||--o{ RANKING : classe

  USER {
    string _id
    string name
    string email
    string photoUrl
    number reputation
    number points
    string[] roles
  }

  SKILL {
    string _id
    string userId
    string label
    enum type "TEACH|LEARN"
    enum level "BEGINNER|INTERMEDIATE|ADVANCED"
  }

  SESSION {
    string _id
    string userAId
    string userBId
    datetime startAt
    datetime endAt
    string placeOrLink
    enum status "PROPOSED|CONFIRMED|DONE|CANCELLED"
  }

  GROUP {
    string _id
    string name
    string adminId
    string[] members
    string description
  }

  TASK {
    string _id
    string groupId
    string title
    string description
    datetime deadline
    enum status "TODO|IN_PROGRESS|DONE"
  }

  BADGE {
    string _id
    string name
    string description
    string icon
  }

  RANKING {
    string _id
    string userId
    enum timeframe "DAILY|WEEKLY|MONTHLY|YEARLY|ALL_TIME"
    number score
    string role "TEACHER|LEARNER"
  }

  MESSAGE {
    string _id
    string sessionId
    string senderId
    string text
    datetime createdAt
  }

  REVIEW {
    string _id
    string sessionId
    string reviewerId
    string revieweeId
    number rating
    string comment
    datetime createdAt
  }
```

---

## 4. Processus de Création

### Conception

* Analyse des besoins & personas (étudiant, pro, retraité mentor, autodidacte).
* Wireframes et prototypage (Figma).
* Parcours : Onboarding → Choix compétences → Matching → Groupe/Session → Classe virtuelle → Progression/Classement.
* UX gamifiée : points, badges, classements visibles → motivation continue.

### Développement

* **Frontend :** React Native (Expo), TypeScript.
* **Backend :** Node.js, Express (REST), Socket.io (chat & notifications temps réel).
* **DB :** MongoDB (Mongoose).
* **Vidéo & Fichiers :** WebRTC/Zoom API, stockage S3/Google Drive.
* **Auth :** JWT + OAuth optionnel (Google/GitHub).
* **Gamification :** microservices pour points & classements.

### Test

* Unitaires (services : ranking, scoring, chat).
* Intégration (API sécurisée JWT, fichiers, visioconf).
* E2E mobile avec Detox (flux complet).
* Tests de charge (sessions vidéo, chat de groupe).
* Tests de sécurité (injections, XSS, auth).

### Déploiement

* Backend (Docker, Railway/Render/Fly.io).
* Base (MongoDB Atlas).
* Mobile (Expo EAS Build).
* Stockage (S3/Cloud).
* Observabilité (logs + monitoring).

### Maintenance

* CI/CD GitHub Actions (lint/test/build).
* Roadmap ouverte (Issues + votes).
* Mise à jour régulière des features proposées par la communauté.
* Évolution de l’algo de ranking.

---

## 5. Acteurs et Parties prenantes

* **Acteurs :**

  * Développeurs mobile (React Native).
  * Développeurs backend (Node/Express).
  * DevOps (déploiement, CI/CD).
  * QA/testeurs.
  * Utilisateurs finaux (apprenants, enseignants).
  * Modérateurs & administrateurs de groupes.

* **Parties prenantes :**

  * Communautés locales, associations, écoles.
  * Stores mobiles (Google Play, App Store).
  * Partenaires éventuels (plateformes de visioconférence).

---

## 6. Outils et Protocoles

* **Méthodologie & Gestion :** Agile/Scrum, GitHub Projects.
* **Design :** Figma, Diagrammes (Mermaid).
* **Stack :** React Native (Expo), Node.js, Express, MongoDB, Socket.io, WebRTC.
* **Qualité :** ESLint, Prettier, Jest, RTL, Supertest, Detox.
* **Sécurité :** HTTPS, JWT, Helmet, Rate-limit, RGPD.
* **CI/CD :** GitHub Actions, Docker, Sentry.

---

## Gamification & Ranking

### Classements dynamiques

Les utilisateurs sont classés selon leur rôle **Enseignant** ou **Apprenant** sur plusieurs périodes :

* Quotidien (Daily)
* Hebdomadaire (Weekly)
* Mensuel (Monthly)
* Annuel (Yearly)
* Historique (All Time)

Chaque tableau de classement affiche le **Top 10** avec : nom, photo, points, progression (+/-).

---

### Attribution des points

Chaque action dans l’application rapporte des points. Exemples :

| Action                             | Rôle            | Points |
| ---------------------------------- | --------------- | ------ |
| Créer un profil complet            | Tous            | +50    |
| Déclarer une compétence            | Tous            | +20    |
| Assister à une session             | Apprenant       | +30    |
| Donner une session                 | Enseignant      | +50    |
| Terminer une tâche de groupe       | Tous            | +40    |
| Publier un fichier/ressource       | Tous            | +25    |
| Participer à une visioconférence   | Tous            | +35    |
| Recevoir une bonne évaluation (5⭐) | Enseignant      | +100   |
| Laisser un avis constructif        | Apprenant       | +20    |
| Créer un sondage                   | Admin de groupe | +30    |

---

### Badges & Récompenses

Des **badges** visuels récompensent la progression et les comportements positifs :

* **Mentor du mois** : enseignant le mieux classé sur 30 jours.
* **Apprenant persévérant** : participation à 10 sessions consécutives.
* **Réponse utile** : recevoir 20 votes positifs sur ses conseils.
* **Super organisateur** : créer 5 groupes actifs.
* **Toujours présent** : taux de présence ≥ 90% sur les sessions confirmées.
* **Polyglotte** : enseigner/apprendre dans 3 langues différentes.

---

### Groupes et Classes virtuelles

* Les enseignants peuvent créer des **groupes** avec plusieurs apprenants.
* Outils de **classe virtuelle** : visioconférence, partage de fichiers, tableau blanc, sondages.
* Attribution de **devoirs/tâches** aux membres avec suivi de progression.
* Les groupes peuvent aussi concourir dans des **classements collectifs** (classe la plus active de la semaine).

---

### Engagement communautaire

* Section **“Proposer une fonctionnalité”** : les utilisateurs peuvent soumettre une idée et voter.
* Les propositions populaires sont priorisées dans la roadmap.
* Système de **points bonus communautaires** pour ceux qui contribuent à l’évolution de la plateforme.

---

## Organisation du travail

* Décider en groupe des fonctionnalités principales (MVP : profils, matching, chat, sessions).
* Étendre avec ranking, groupes, gamification.
* Mise en commun → arbitrage, priorisation.
* Démo + présentation orale (montrer l’aspect fun et motivant).

---
