```mermaid
flowchart LR
  U[Utilisateur]
  FE[Front-end Web]
  API[API Back-end]
  DB[(Base de données)]
  EMAIL[Service email (externe)]
  PUSH[Service de notifications push]

  U --> FE
  FE --> API
  API --> DB
  API --> EMAIL
  API --> PUSH
```