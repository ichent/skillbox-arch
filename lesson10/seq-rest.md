```mermaid
sequenceDiagram
    autonumber
    actor User as Клиент
    participant API as Заказ
    participant CB as Circuit Breaker
    participant Catalog as Каталог

    User->>API: POST /orders (Создать заказ)
    API->>CB: Выполнить запрос к Каталогу?

    rect rgb(200, 255, 200)
        note right of CB: Состояние: CLOSED (Норма)
        CB->>Catalog: GET /products/{id}
        Catalog--xCB: Ошибка соединения / 500
    end

    rect rgb(255, 255, 200)
        note over API, Catalog: Pattern: Retry (Попытка 2)
        CB->>Catalog: GET /products/{id}
        Catalog--xCB: Timeout / 500
    end

    rect rgb(255, 200, 200)
        note over CB: Превышен порог ошибок!
        note right of CB: Переход в состояние: OPEN
    end

    CB-->>API: Exception: CircuitBreakerOpen
    API-->>User: 503 Service Unavailable<br/>(Сервис временно недоступен)