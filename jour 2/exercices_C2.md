# Exercices pratiques C2 — MyToDo

---

## Exercice 1 — Enrichir l'architecture : ajout de Redis

### Container ajouté : Redis (Service de cache)

**Justification :**
Redis est un store clé-valeur en mémoire utilisé comme couche de cache entre l'API Back-end et la base de données PostgreSQL. Il permet de réduire la charge sur la base de données pour les requêtes fréquentes (ex : liste des tâches d'un utilisateur, données de session) et d'améliorer les temps de réponse de l'application.

**Interactions :**
- L'API Back-end interroge Redis **avant** PostgreSQL. Si la donnée est en cache (cache hit), elle est retournée directement sans requête SQL.
- Si la donnée est absente (cache miss), l'API interroge PostgreSQL, puis stocke le résultat dans Redis pour les prochaines requêtes.
- Lors d'une modification ou suppression de tâche, l'API invalide les entrées Redis correspondantes pour garantir la cohérence des données.

### Diagramme C2 enrichi

```mermaid
graph LR
    U["👤 Utilisateur"]
    subgraph "MyToDo [Système]"
        FE["🌐 Front-end Web<br/>[React]<br/>Interface utilisateur"]
        API["⚙️ API Back-end<br/>[Node.js/Express]<br/>Logique métier"]
        CACHE["⚡ Cache<br/>[Redis]<br/>Cache & Sessions"]
        DB[("🗄️ Base de données<br/>[PostgreSQL]<br/>Stockage")]
    end
    E["📧 Service Email<br/>[SendGrid]"]
    U -->|"HTTPS"| FE
    FE -->|"HTTP/JSON"| API
    API -->|"Redis Protocol/TCP"| CACHE
    API -->|"SQL/TCP"| DB
    API -->|"API REST"| E
    CACHE -.->|"Cache miss → fallback"| DB
```

---

## Exercice 2 — Diagramme C2 personnalisé (Stack Python)

Application : **MyBlog** — plateforme de publication d'articles avec système de commentaires.

```mermaid
graph LR
    U["👤 Utilisateur"]
    A["🔧 Administrateur"]
    subgraph "MyBlog [Système]"
        FE["🌐 Front-end Web<br/>[Vue.js]<br/>Interface de lecture et rédaction"]
        API["⚙️ API Back-end<br/>[FastAPI]<br/>Logique métier & REST"]
        DB[("🗄️ Base de données<br/>[PostgreSQL]<br/>Articles, users, commentaires")]
        STORE["🗂️ Stockage fichiers<br/>[S3-compatible]<br/>Images et médias"]
    end
    MAIL["📧 Service Email<br/>[SendGrid]"]
    U -->|"HTTPS"| FE
    A -->|"HTTPS"| FE
    FE -->|"HTTP/JSON — REST"| API
    API -->|"SQL/TCP"| DB
    API -->|"HTTPS — S3 API"| STORE
    API -->|"HTTPS — API REST"| MAIL
```

**Annotations des connexions :**

| Connexion | Protocole | Usage |
|---|---|---|
| Utilisateur → Front-end | HTTPS | Navigation, lecture, soumission de formulaires |
| Front-end → API | HTTP/JSON (REST) | Appels CRUD vers les endpoints FastAPI |
| API → PostgreSQL | SQL/TCP | Lecture et écriture des données métier |
| API → Stockage S3 | HTTPS / S3 API | Upload et récupération d'images |
| API → SendGrid | HTTPS / API REST | Envoi d'emails (confirmation, notification) |

---

## Exercice 3 — Diagrammes de séquence

### Scénario A : Modification d'une tâche

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant FE as Front-end
    participant API as API Back-end
    participant CACHE as Redis
    participant DB as Base de données

    U->>FE: Modifie le titre et la date d'une tâche
    FE->>FE: Validation côté client
    FE->>API: PUT /api/tasks/123 {title, dueDate}
    API->>API: Validation + Vérification JWT
    API->>DB: UPDATE tasks SET ... WHERE id = 123
    DB-->>API: OK (rows affected: 1)
    API->>CACHE: DEL task:123
    CACHE-->>API: OK (cache invalidé)
    API-->>FE: 200 OK {task updated}
    FE-->>U: Affiche la tâche mise à jour
```

---

### Scénario B : Consultation des statistiques

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant FE as Front-end
    participant API as API Back-end
    participant CACHE as Redis
    participant DB as Base de données

    U->>FE: Accède au tableau de bord statistiques
    FE->>API: GET /api/stats
    API->>API: Vérification JWT
    API->>CACHE: GET stats:userId
    alt Cache hit
        CACHE-->>API: Données en cache
    else Cache miss
        CACHE-->>API: null
        API->>DB: SELECT COUNT(*), category, status FROM tasks GROUP BY ...
        DB-->>API: Résultats agrégés
        API->>CACHE: SET stats:userId (TTL: 5min)
    end
    API-->>FE: 200 OK {stats}
    FE-->>U: Affiche les graphiques et indicateurs
```

---

### Scénario C : Réinitialisation de mot de passe

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant FE as Front-end
    participant API as API Back-end
    participant DB as Base de données
    participant MAIL as SendGrid

    U->>FE: Saisit son email sur "Mot de passe oublié"
    FE->>API: POST /api/auth/forgot-password {email}
    API->>DB: SELECT * FROM users WHERE email = ?
    DB-->>API: User data (ou null)
    API->>API: Génère un token de réinitialisation (UUID + expiration 1h)
    API->>DB: INSERT INTO reset_tokens (userId, token, expiresAt)
    DB-->>API: OK
    API->>MAIL: Envoie email avec lien /reset-password?token=...
    MAIL-->>API: 202 Accepted
    API-->>FE: 200 OK (message générique, sans révéler si l'email existe)
    FE-->>U: "Si l'email existe, un lien vous a été envoyé."
    U->>FE: Clique sur le lien et saisit le nouveau mot de passe
    FE->>API: POST /api/auth/reset-password {token, newPassword}
    API->>DB: SELECT * FROM reset_tokens WHERE token = ? AND expiresAt > NOW()
    DB-->>API: Token valide
    API->>API: Hash du nouveau mot de passe (bcrypt)
    API->>DB: UPDATE users SET password = ? WHERE id = ?
    API->>DB: DELETE FROM reset_tokens WHERE token = ?
    DB-->>API: OK
    API-->>FE: 200 OK
    FE-->>U: Redirige vers la page de connexion
```

---

## Exercice 4 — Tableau des responsabilités (Fait / Ne fait pas)

### Front-end Web [React]

| ✅ Fait | ❌ Ne fait pas |
|---|---|
| Affiche l'interface utilisateur et les vues | Accède directement à la base de données |
| Valide les données côté client (format, champs requis) | Applique les règles métier (ex : droits d'accès) |
| Envoie les requêtes HTTP à l'API Back-end | Stocke des données sensibles (tokens en localStorage non sécurisé) |

---

### API Back-end [Node.js/Express]

| ✅ Fait | ❌ Ne fait pas |
|---|---|
| Applique la logique métier et les règles de gestion | Génère ou affiche du HTML/CSS |
| Authentifie et autorise les requêtes (JWT) | Communique directement avec des services externes sans passer par une couche d'abstraction |
| Orchestre les appels vers la base de données et les services tiers | Stocke des fichiers binaires ou des médias |

---

### Base de données [PostgreSQL]

| ✅ Fait | ❌ Ne fait pas |
|---|---|
| Stocke et persiste les données de manière fiable | Exécute de la logique métier complexe |
| Garantit l'intégrité des données (contraintes, clés étrangères) | Envoie des emails ou des notifications |
| Répond aux requêtes SQL de l'API | Gère l'authentification des utilisateurs |

---

### Cache [Redis]

| ✅ Fait | ❌ Ne fait pas |
|---|---|
| Met en cache les résultats de requêtes fréquentes | Sert de source de vérité (les données ne sont pas persistées de façon fiable) |
| Stocke les sessions utilisateur | Remplace la base de données pour les données critiques |
| Accélère les temps de réponse de l'API | Applique des règles de validation ou de sécurité |
