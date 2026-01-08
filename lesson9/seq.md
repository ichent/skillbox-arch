```mermaid
flowchart TD
    User((Пользователь)) -->|Cmd: CreateOrder| OrderAgg[Агрегат: Заказ]

    %% Happy Path
    OrderAgg -->|Evt: OrderCreated| Broker[Message Broker]
    Broker -->|Evt: OrderCreated| InventoryAgg[Агрегат: Склад]

    InventoryAgg -->|Evt: StockReserved| Broker
    Broker -->|Evt: StockReserved| OrderAgg

    OrderAgg -->|Evt: OrderConfirmed| Broker
    Broker -->|Evt: OrderConfirmed| DeliveryAgg[Агрегат: Доставка]
    Broker -->|Evt: OrderConfirmed| Analytics[Сервис Отчетности]

    DeliveryAgg -->|Evt: DeliveryScheduled| Broker

    %% Unhappy Path (Исправлено)
    InventoryAgg -->|Evt: StockReservationFailed| Broker
    Broker -->|Evt: StockReservationFailed| OrderAgg

    %% Кавычки нужны, чтобы скобки внутри текста не ломали парсер
    OrderAgg -->|"Evt: OrderCancelled <br/>(Компенсация)"| Broker
    Broker -->|Evt: OrderCancelled| Analytics

    %% Styling
    classDef broker fill:#f9f,stroke:#333;
    class Broker broker;