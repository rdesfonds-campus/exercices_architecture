# Glossaire C2 + Exercice 1 MyToDo

## Définition "Container"
**Container (C4 Model)** : Unité d'exécution ou de stockage **séparément déployable** (app web, service API, DB). **Attention** : ≠ Docker.

## Tableau Containers MyToDo (enrichi Redis)
| Container       | Responsabilité                                   | Technologie       | Protocole            |
|-----------------|--------------------------------------------------|-------------------|----------------------|
| Front-end Web  | Interface utilisateur, interactions              | React             | HTTP/JSON            |
| API Back-end   | Logique métier, auth, validation                 | Node.js/Express   | REST API             |
| Base de données| Stockage tâches/users/catégories                 | PostgreSQL        | SQL/TCP              |
| Cache          | Cache req. freq. (dashboards), sessions          | Redis             | Redis Protocol/TCP   |

## Diagramme de séquence personnalisée C2 PlantUML 
```plantuml
@startuml
title Réinitialisation du mot de passe - MyToDo

actor Utilisateur
participant "Front-end - Vue.js" as FE
participant "API Back-end - Spring Boot" as Auth
participant "Service email - PSQL" as Mail

Utilisateur -> FE : Clique sur "Mot de passe oublié"

FE -> Auth : POST /reset-password {email}
Auth -> Mail : Génère un token de reset\nEnvoie email avec lien de reset
Mail --> Utilisateur : Email reçu avec lien de reset

Utilisateur -> FE : Clique sur le lien de reset
FE -> Auth : POST /reset-password/confirm {token, newPassword}
Auth -> Auth : Met à jour le mot de passe
Auth --> Utilisateur : 200 OK\n"Mot de passe mis à jour, redirige vers login"\nEmail confirmation de réinitialisation


alt Format email invalide
  FE --> Utilisateur : "Invalid email format"
else Format email valide
  FE -> Auth : POST /reset-password {email}
  Auth -> Mail : Génère un token de reset\nEnvoie email avec lien de reset
  Mail --> Utilisateur : Email reçu avec lien de reset
end

Utilisateur -> FE : Clique sur le lien de reset
FE -> Auth : POST /reset-password/confirm {token, newPassword}

alt Token valide
  Auth -> Auth : Met à jour le mot de passe
  Auth --> Utilisateur : 200 OK\n"Mot de passe mis à jour, redirige vers login"\nEmail confirmation de réinitialisation
else Token invalide ou expiré
  Auth --> Utilisateur : 400 Bad Request\n"Lien invalide ou expiré"
end

@enduml
```
## 3 Diagrammes de séquences

```mermaid
graph LR
  U((Utilisateur)) -. HTTPS .-> FE[Front-end Web<br/>React]
  subgraph MYTODO["MyToDo Système"]
    FE -. "HTTP/JSON" .-> API[API Back-end<br/>Node.js/Express]
    API -. "Redis Protocol/TCP" .-> CACHE[Cache<br/>Redis]
    API -. "SQL/TCP" .-> DB[(PostgreSQL)]
  end
  API -. "REST/HTTPS" .-> EMAIL[Service Email<br/>SendGrid]
  CACHE -.->|cache miss| DB
```

# Tableau Responsabilités (Ex4)

| Container | ✅ Doit faire | ❌ Ne doit pas faire |
|-----------|---------------|---------------------|
| Front-end | 1. Affichage UI<br>2. Validation client<br>3. Appels API | 1. Logique métier<br>2. Accès DB direct<br>3. Stockage persistant |
| API Back-end | 1. Auth Bearer<br>2. Validation serveur<br>3. Orchestrer Cache/DB | 1. Générer HTML<br>2. UI state<br>3. Cache persistant |
| Base données | 1. Stockage ACID<br>2. Indexation<br>3. Backup | 1. Logique métier<br>2. Cache temp<br>3. Auth |
| Cache Redis | 1. Cache sessions<br>2. Listes tâches freq<br>3. TTL expire | 1. Données critiques<br>2. Persistance seule<br>3. Logique métier |
