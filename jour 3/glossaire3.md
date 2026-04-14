# Définitions clés
Component : Groupement logique de code avec une responsabilité cohérente (pas une seule classe).

Controller : Reçoit requêtes HTTP, valide entrées, délègue au Service.

Service : Logique métier, orchestre opérations.

Repository : Gère accès données, encapsule requêtes SQL.

Styles : MVC (simple), N-Tiers (couches), Hexagonale (domaine centre), Clean (dépendances vers centre).

# Liste complète des composants
AuthController, TaskController, StatsController, AuthService, TaskService, StatisticsService, UserRepository, TaskRepository, CategoryRepository, NotificationService, AuthMiddleware, LoggingMiddleware, ErrorHandler.

# Tableau des responsabilités

| Composant           | Responsabilité (fait)                            | Ne doit PAS faire                     |
| ------------------- | ------------------------------------------------ | ------------------------------------- |
| AuthController      | Reçoit requêtes auth, valide format              | Vérifier mots de passe, accéder BDD   |
| TaskController      | Reçoit requêtes CRUD tâches                      | Appliquer règles métier, requêtes SQL |
| StatsController     | Reçoit requêtes stats                            | Calculer stats, accéder BDD           |
| AuthService         | Vérifie credentials, génère tokens JWT           | Gérer HTTP, stocker en BDD            |
| TaskService         | Applique logique métier tâches (ex: priorités)   | Connaître détails BDD                 |
| StatisticsService   | Calcule stats d'utilisation                      | Gérer affichage stats                 |
| NotificationService | Envoie notifications (email/push)                | Gérer logique métier tâches           |
| UserRepository      | CRUD utilisateurs en BDD                         | Logique métier, validation            |
| TaskRepository      | CRUD tâches en BDD                               | Règles métier, calculs stats          |
| CategoryRepository  | CRUD catégories en BDD                           | Logique métier tâches                 |
| AuthMiddleware      | Vérifie tokens JWT sur routes                    | Générer tokens                        |
| LoggingMiddleware   | Logue requêtes/réponses                          | Traiter logique métier                |
| ErrorHandler        | Gère erreurs globales, renvoie réponses standard | Décider logique métier                |

# Diagramme mermaid

```mermaid
graph TD
    subgraph "API Back-end Container"
        subgraph "Controllers"
            ACAuthController["AuthController"]
            TCTaskController["TaskController"]
            SCStatsController["StatsController"]
        end
        subgraph "Services"
            ASAuthService["AuthService"]
            TSTaskService["TaskService"]
            SSStatisticsService["StatisticsService"]
            NSNotificationService["NotificationService"]
        end
        subgraph "Repositories"
            URUserRepository["UserRepository"]
            TRTaskRepository["TaskRepository"]
            CRCategoryRepository["CategoryRepository"]
        end
        subgraph "Middlewares"
            AuthMW["AuthMiddleware"]
            LogMW["LoggingMiddleware"]
            ErrH["ErrorHandler"]
        end
    end
    ACAuthController --> ASAuthService
    TCTaskController --> TSTaskService
    SCStatsController --> SSStatisticsService
    TSTaskService --> NSNotificationService
    ASAuthService --> URUserRepository
    TSTaskService --> TRTaskRepository
    SSStatisticsService --> TRTaskRepository
    NSNotificationService --> TRTaskRepository
    URUserRepository --> DB["Database"]
    TRTaskRepository --> DB
    CRCategoryRepository --> DB
    AuthMW -.->|Applique sur routes| ACAuthController
    AuthMW -.->|Applique sur routes| TCTaskController
    LogMW -.->|Logue tout| ACAuthController
    ErrH -.->|Gère erreurs| TCTaskController
   
 ```

# Choix d'architecture : N-Tiers (justification)
### Choix : Architecture N-Tiers (couches : Présentation/Controllers, Métier/Services, Données/Repositories).
1. Séparation claire des responsabilités (chaque couche un rôle).

2. Facile à tester (mock par couche).

3. Adapté à MyToDo (projet moyen, pas critique).

4. Simple à implémenter pour débutant.

5. Bonne scalabilité sans complexité excessive.

Risques : Rigidité (difficile changer ordre couches), dépendances descendantes (haut dépend bas).

### Comparaison styles
| Style      | Complexité | Testabilité | Flexibilité | Idéal pour        |
| ---------- | ---------- | ----------- | ----------- | ----------------- |
| MVC        | Faible     | Moyenne     | Faible      | Petits projets    |
| N-Tiers    | Moyenne    | Bonne       | Moyenne     | Projets moyens    |
| Hexagonale | Haute      | Excellente  | Haute       | Gros projets      |
| Clean      | Très haute | Excellente  | Très haute  | Projets critiques |

# Mapping Use Cases - Components
Use Cases typiques de MyToDo mappés aux composants.

| Use Case                    | Composants impliqués                                             |
| --------------------------- | ---------------------------------------------------------------- |
| UC1: Se connecter           | AuthController, AuthMiddleware, AuthService, UserRepository      |
| UC2: Créer tâche            | TaskController, AuthMiddleware, TaskService, TaskRepository      |
| UC3: Lister tâches          | TaskController, TaskService, TaskRepository                      |
| UC4: Modifier tâche         | TaskController, TaskService, TaskRepository, NotificationService |
| UC5: Supprimer tâche        | TaskController, TaskService, TaskRepository                      |
| UC6: Ajouter catégorie      | TaskController, TaskService, CategoryRepository                  |
| UC7: Voir stats utilisateur | StatsController, StatisticsService, TaskRepository               |