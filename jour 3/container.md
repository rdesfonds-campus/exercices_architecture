```mermaid
graph TD
    subgraph "API Back-end [Container]"
        subgraph Controllers
            AC[AuthController]
            TC[TaskController]
            SC[StatsController]
        end
        subgraph Services
            AS[AuthService]
            TS[TaskService]
            SS[StatisticsService]
        end
        subgraph Repositories
            UR[UserRepository]
            TR[TaskRepository]
        end
    end
    
    DB[(Base de données)]
    AC --> AS
    TC --> TS
    SC --> SS
    AS --> UR
    TS --> TR
    SS --> TR
    UR --> DB
    TR --> DB