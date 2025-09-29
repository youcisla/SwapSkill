# SwapSkill

---

## 1. Imaginer une Application

- **Nom de l'Application :** SkillSwap  
- **Type :** Application mobile sociale et collaborative d’échange de compétences (apprendre/enseigner) avec gamification, groupes, IA et classe virtuelle.  

---

## 2. Décrire l'Application

### Objectif
Faciliter l’apprentissage accessible, communautaire et motivant en mettant en relation des personnes qui souhaitent **enseigner** ou **apprendre** des compétences.  
Le tout avec un système **d’échange de savoirs**, de **classements**, de **groupes collaboratifs**, et d’**outils d’IA** pour accompagner enseignants et apprenants.

### Problèmes résolus
- Coût élevé et rigidité des cours traditionnels.  
- Manque de structure pour des apprentissages collaboratifs.  
- Manque de motivation sur le long terme.  
- Difficulté à valider la crédibilité (scams, faux profils).  
- Besoin d’une plateforme unifiée et moderne pour l’apprentissage.

### Fonctionnalités principales
- **Profils & Compétences :**
  - Création de profil (bio, photo, localisation approximative).
  - Déclaration “J’enseigne / J’apprends” (compétence, niveau, disponibilité).
  - **Upload de certificats & diplômes** (vérification → attribution de points et badges “validé”).
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
  - **Visioconférences intégrées**.
  - Partage d’écran et tableau blanc collaboratif.
  - Possibilité de créer des sous-salles (breakout rooms).
- **Partage & organisation :**
  - Assignation de tâches et de devoirs (avec deadline).
  - Upload/partage de tout type de fichier.
  - Création de sondages et votes collaboratifs.
- **Ranking & Gamification :**
  - Classements enseignants et apprenants (quotidien, hebdomadaire, mensuel, annuel, “All time”).
  - Attribution de points pour chaque action (enseigner, participer, publier, aider, certifier un diplôme).
  - **Badges** pour rôles et comportements (Mentor du mois, Apprenant persévérant, etc.).
  - Système de **niveaux** avec progression visible.
- **Statistiques en temps réel :**
  - Graphiques sur la **diffusion des connaissances** (par jour, semaine, mois, année, global).
  - Indicateurs : sessions créées, tâches terminées, points cumulés, certificats validés.
- **Personnalisation & Communauté :**
  - Suggestions de groupes ou de pairs selon l’historique.
  - Section **“Proposer une nouvelle fonctionnalité”** votable par la communauté.
  - Notifications personnalisées (push/email).
- **Éducation formelle :**
  - **Mode “Écoles/Universités”** : intégration des groupes classes, professeurs, cours et devoirs.
  - Tableau de bord administrateur pour établissements.
  - Personnalisation de l’espace (logo, couleurs, privilèges).
- **Abonnements intelligents :**
  - **Gratuit (80% des fonctionnalités)** → pour tous ceux qui veulent apprendre/enseigner librement.
  - **Avancé (Premium)** → statistiques détaillées, stockage fichiers illimité, replay visioconférences.
  - **VIP/Établissements** → mode université, intégration API, gestion multi-groupes.
- **IA Agents intégrés :**
  - **IA Mentor** : suggère parcours d’apprentissage personnalisés.
  - **IA Assistant Professeur** : aide les enseignants à préparer cours, tâches, quiz.
  - **IA Copilote Étudiant** : soutien en temps réel (explications, exercices, traduction).
  - **IA Modération** : détection de contenu inapproprié, prévention scams.
- **Sécurité & Modération :**
  - Auth JWT + OAuth (Google/GitHub).
  - Vérification email/téléphone, validation certificats.
  - Signalement/blocage d’utilisateur.
  - Paramètres de confidentialité (visibilité profil, localisation approximative).

### Utilité
- **Pour les enseignants :** visibilité, reconnaissance via le classement, gestion de groupes & sessions, reconnaissance officielle via diplômes.  
- **Pour les apprenants :** apprentissage gratuit et flexible, progression visible, validation de diplômes, appartenance à une communauté motivante.  
- **Pour les institutions :** adoption comme **plateforme e-learning personnalisée**.  
- **Pour tous :** un **hub central** de l’apprentissage moderne, motivant et assisté par IA.

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
  AI[IA Agents intégrés]

  App -- HTTPS/JSON --> API
  API --- DB
  App <-- WebSocket --> WS
  App -. Push .-> Push
  App -. GPS/Map .-> Maps
  App <-- Stream --> Video
  App <-- Upload/Download --> Files
  App <-- AI Assistance --> AI
````

---

## 4. Processus de Création

### Conception

* Analyse besoins (individuels & institutions).
* Personas : étudiants, profs, mentors bénévoles, écoles.
* Wireframes (Figma).
* UX : gamifiée, motivante, intuitive, adaptée mobile.
* Flow : Onboarding → Matching → Groupe/Session → Classe virtuelle → Stats → Progression.

### Développement

* **Frontend :** React Native (Expo), TypeScript.
* **Backend :** Node.js, Express, Socket.io, MongoDB.
* **Vidéo :** WebRTC, Zoom SDK.
* **IA :** intégration API LLM (OpenAI, HuggingFace).
* **Auth :** JWT + OAuth.
* **Gamification :** microservice ranking/scoring.

### Test

* Unitaires : ranking, IA assistants, sessions.
* Intégration : API JWT, visioconf, certificats.
* E2E mobile (Detox).
* Tests charge (sessions vidéo).
* Sécurité : anti-injection, validation certificats.

### Déploiement

* Backend : Docker + Railway/Render.
* DB : MongoDB Atlas.
* Mobile : Expo EAS Build.
* Fichiers : S3/Cloud.
* Observabilité : logs, Sentry, Prometheus.

### Maintenance

* CI/CD GitHub Actions.
* Roadmap ouverte (Issues & votes).
* Mises à jour régulières (feedback utilisateurs + votes).
* IA évolutive (nouvelles fonctionnalités).

---

## 5. Acteurs et Parties prenantes

* **Acteurs :**

  * Développeurs (front RN, back Node, IA).
  * DevOps (infra, CI/CD).
  * QA/testeurs.
  * Utilisateurs finaux (apprenants/enseignants).
  * Admins & modérateurs.
  * Écoles/Universités.

* **Parties prenantes :**

  * Associations, communautés locales.
  * Stores mobiles.
  * Partenaires IA et visioconférence.

---

## 6. Outils et Protocoles

* **Gestion :** Agile/Scrum, GitHub Projects.
* **Design :** Figma, Mermaid.
* **Stack :** React Native, Node.js, Express, MongoDB, Socket.io, WebRTC, IA APIs.
* **Qualité :** ESLint, Prettier, Jest, RTL, Supertest, Detox.
* **Sécurité :** HTTPS, JWT, Helmet, RGPD, validation certificats.
* **CI/CD :** GitHub Actions, Docker, Sentry, Prometheus.

---

## Gamification & Ranking

### Classements

* Classements enseignants & apprenants : Daily, Weekly, Monthly, Yearly, All Time.
* Top 10 visible avec progression.

### Points

Chaque action → points : enseignement, participation, certification, partage fichier, avis, sondages.

### Badges

* Mentor du mois
* Apprenant persévérant
* Réponse utile
* Super organisateur
* Toujours présent
* Polyglotte
* Certifié (diplômes validés)

### Statistiques

* Graphiques sur sessions, tâches, diplômes validés, diffusion du savoir.
* Vue : Jour, Semaine, Mois, Année, Global.

### Communauté

* Proposer une fonctionnalité → votée par la communauté.
* Points bonus pour contributeurs.

---

## Abonnements & Monétisation

* **Gratuit (80% des fonctionnalités)** : profils, matching, chat, sessions, certificats basiques.
* **Avancé (Premium)** : stats détaillées, stockage illimité, replay vidéos, AI assistant.
* **VIP / Établissements** : intégration université, multi-groupes, analytics avancées.

---

## Vision finale

Faire de **SwapSkill** le **hub central du savoir**, alliant **communauté**, **éducation formelle**, **gamification**, et **IA** pour créer un écosystème unique de partage de connaissances.

---
