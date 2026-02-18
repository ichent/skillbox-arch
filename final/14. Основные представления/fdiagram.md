graph TD
    user((Пользователь)) --> mobile[Мобильное Приложение<br/>React Native]

    subgraph "Backend System"
        gateway[API Gateway<br/>Nginx]

        subgraph "Core Business Functions"
            auth[Identity Service<br/>Auth & Profile]
            training[Training Service<br/>Telemetry Processing]
            inventory[Inventory Service<br/>Equipment Lifecycle]
        end

        subgraph "Engagement Functions"
            gamify[Gamification Service<br/>Leaderboards & XP]
            social[Social Service<br/>Search & Connections]
        end
    end

    mobile -->|REST/HTTPS| gateway
    gateway --> auth
    gateway --> training
    gateway --> inventory
    gateway --> gamify
    gateway --> social