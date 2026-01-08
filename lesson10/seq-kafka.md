```mermaid
sequenceDiagram
    autonumber
    participant Kafka as Message Broker
    participant Warehouse as Склад
    participant DB as Склад DB
    participant DLQ as Dead Letter Queue

    Kafka->>Warehouse: Event: OrderCreated

    loop Pattern: Retry with Backoff
        Warehouse->>DB: SQL: UPDATE stock SET reserved...
        DB--xWarehouse: Deadlock / Connection Error
        Note over Warehouse: Ошибка! Ждем 2 сек...
    end

    Note over Warehouse: Исчерпаны все попытки (Max Retries exceeded)

    Warehouse->>DLQ: Publish: OrderCreated (с причиной ошибки)
    Note right of DLQ: Алерт админу: "Заказ застрял!"

    Warehouse-->>Kafka: Commit Offset (Сообщение "обработано")
    Note over Warehouse: Готов к приему следующих заказов