```mermaid

graph LR
    subgraph "Public Zone (Unstrusted)"
        User[Internet User]
    end

    subgraph "DMZ (Demilitarized Zone)"
        LB[Load Balancer / WAF]
        Auth[Keycloak Public Endpoint]
    end

    subgraph "Private Zone (Trusted)"
        Services[Microservices List]
        DBs[Databases & Kafka]
    end

    User --> LB
    LB --> Auth
    LB --> Services
    Services -.-> DBs

    style User fill:#ffcdd2,stroke:#b71c1c
    style DMZ fill:#fff9c4,stroke:#fbc02d
    style Private Zone fill:#c8e6c9,stroke:#2e7d32