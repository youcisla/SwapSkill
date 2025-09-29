# SwapSkill

---

## 1. Imaginer une Application

- **Nom de l'Application :** SkillSwap  
- **Type :** Application mobile sociale d’échange de compétences (apprendre/enseigner) sans transaction financière.  

---

## 2. Décrire l'Application

### Objectif
Faciliter l’apprentissage accessible et local en mettant en relation des personnes qui souhaitent **enseigner** une compétence X et **apprendre** une compétence Y, via un système d’échange (barter) simple et sécurisé.

### Problèmes résolus
- Coût élevé et rigidité des cours traditionnels.  
- Difficulté à trouver des pairs proches, disponibles et de confiance.  
- Manque de structure pour planifier et valider des sessions d’échange.

### Fonctionnalités principales
- **Profils & Compétences :**
  - Création de profil (bio, photo, localisation approximative).
  - Déclaration “J’enseigne / J’apprends” (compétence, niveau, disponibilité).
- **Matching intelligent :**
  - Suggestions basées sur les compétences communes, la distance, la disponibilité et la réputation.
  - Filtres (rayon, niveau, langue, mode présentiel/distanciel).
- **Messagerie & Sessions :**
  - Chat en temps réel (Socket.io).
  - Proposition/confirmation de sessions (date, lieu/visio), rappels.
- **Avis & Réputation :**
  - Notes post-session, commentaires, badges (fiabilité, pédagogie).
- **Sécurité & Modération :**
  - Authentification JWT, vérification email, signalement/blocage d’utilisateur.
  - Paramètres de confidentialité (partage de position approximative).

### Utilité
Encourage le partage de savoirs, l’entraide locale et la montée en compétences continue, avec une barrière d’entrée minimale (pas d’échange d’argent).

---

## 3. Schéma ou Diagramme

### Architecture
```mermaid
flowchart LR
  App[App Mobile (React Native/Expo)]
  API[API REST (Node.js/Express)\nAuth JWT, validation]
  WS[Temps réel (Socket.io)]
  DB[(MongoDB Atlas)]
  Push[Notifications (Expo)]
  Maps[Localisation (Expo Location / Map)]

  App -- HTTPS/JSON --> API
  API --- DB
  App <-- WebSocket --> WS
  App -. Push .-> Push
  App -. GPS/Map .-> Maps
````

### Modèle de données simplifié

```mermaid
erDiagram
  USER ||--o{ SKILL : possede
  USER ||--o{ SESSION : participe
  USER ||--o{ REVIEW : evalue
  SESSION ||--o{ MESSAGE : contient

  USER {
    string _id
    string name
    string email
    string photoUrl
    number reputation
    string[] availability
    Point location
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

* Analyse des besoins & définition des personas.
* Parcours utilisateur : Onboarding → Déclarer compétences → Voir matches → Discuter → Session → Avis.
* UX/UI :

  * Écrans en cartes (photo, compétences, distance, score).
  * Filtres/moteur de recherche.
  * États vides + feedbacks (toasts, erreurs).
  * Accessibilité (contraste, labels).
* Prototypage avec **Figma**.

### Développement

* **Frontend :** React Native (Expo), TypeScript, React Navigation, Redux Toolkit/Zustand, Expo Location, Expo Notifications.
* **Backend :** Node.js, Express (API REST), Socket.io (chat temps réel), MongoDB (Mongoose), JWT, bcrypt, validation avec Zod/Joi.
* **Qualité :** ESLint, Prettier, Husky, conventions de commits.

### Test

* **Unitaires :** services (matching, calcul de score), contrôleurs API (Supertest).
* **Intégration :** endpoints REST sécurisés (JWT), WebSocket (chat).
* **Front :** tests de composants (React Testing Library), snapshots.
* **E2E mobile :** Detox (flux complet onboarding → match → chat → session → avis).
* **Sécurité :** tests d’injection, CORS strict, validation d’entrée.

### Déploiement

* **Backend :** Docker, déploiement sur Railway/Render/Fly.io.
* **Base :** MongoDB Atlas (sauvegardes, accès sécurisé).
* **Mobile :** Expo EAS Build (staging/prod), EAS Update (OTA).
* **Monitoring :** logs (winston/pino), Sentry, UptimeRobot.

### Maintenance

* **CI/CD :** GitHub Actions (lint, test, build, déploiement auto).
* **Versioning :** semver, changelog, releases GitHub.
* **Sécurité :** rotation secrets, audit dépendances (Dependabot).
* **Support & Modération :** outils admin (ban, suppression contenu), tutoriels in-app, FAQ.
* **Roadmap :** GitHub Projects/Milestones, priorisation (MoSCoW).

---

## 5. Acteurs et Parties prenantes

* **Acteurs :**

  * Développeurs frontend (React Native).
  * Développeurs backend (Node/Express).
  * DevOps (CI/CD, infra).
  * QA/testeurs.
  * Utilisateurs finaux (enseignants/apprenants).
  * Modérateurs/Support.

* **Parties prenantes :**

  * Associations, fablabs, universités (promotion et communauté).
  * Stores mobiles (Google Play, App Store).

---

## 6. Outils et Protocoles

* **Méthodologie & Gestion :** Agile (Kanban, sprints courts), Git/GitHub (Issues, PR, Projects).
* **Design :** Figma (maquettes, UI kit), Diagrammes (Mermaid/Draw.io).
* **Stack technique :** React Native (Expo), Node.js, Express, Socket.io, MongoDB (Mongoose), TypeScript.
* **Qualité :** ESLint, Prettier, Jest, React Testing Library, Supertest, Detox.
* **Sécurité :** HTTPS, JWT Bearer, Helmet, rate-limit, validation Zod/Joi.
* **CI/CD :** GitHub Actions, Docker, Sentry, MongoDB Atlas (sauvegardes).
* **Conformité :** RGPD (mentions légales, consentement, droit à l’effacement).

---

## Organisation du travail

* Décision en groupe des fonctionnalités principales (MVP : profils, matching, chat, sessions).
* Temps individuel pour détailler processus, acteurs, outils.
* Mise en commun → arbitrage, priorisation (MoSCoW).
* Rédaction finale & préparation de la présentation orale.

---
