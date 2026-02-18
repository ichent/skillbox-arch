```mermaid

graph TB
    subgraph "External Network (Internet)"
        Client[Mobile Devices]
    end

    subgraph "Kubernetes Cluster"
        ingress[Ingress Controller / Load Balancer]

        subgraph "Stateless Services Layer"
            id_pod[Pod: Identity]
            train_pod[Pod: Training X 3]
            game_pod[Pod: Gamification]
        end

        subgraph "Messaging Backbone"
            kafka{{Apache Kafka Cluster}}
        end
    end

    subgraph "Persistence Layer (Cloud/On-Prem)"
        pg[(PostgreSQL Cluster)]
        ts[(TimescaleDB)]
        redis[(Redis Cache)]
    end

    Client -->|HTTPS| ingress
    ingress --> id_pod
    ingress --> train_pod
    ingress --> game_pod

    train_pod -.-> kafka

    id_pod --> pg
    train_pod --> ts
    game_pod --> redis