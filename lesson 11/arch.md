```mermaid
C4Container
    title C4 Architecture: Final + Feature Flags (A/B Testing)

    Person(user, "Клиент", "Пользователь приложения")
    Person(analyst, "Аналитик", "BI Staff", "Строит отчеты по продажам")
    Person(marketer, "Маркетолог", "Business Owner", "Настраивает гипотезы и A/B тесты")

    Boundary(ecommerce_system, "E-commerce System") {

        Container(ext_gw, "External Gateway", "Kong/Nginx", "HTTPS, Auth, Public API + UI Composition")

        %% Internal Gateway
        Container(int_gw, "Internal Gateway", "K8s Service / Envoy", "Internal LB & Discovery")

        %% --- НОВЫЙ КОМПОНЕНТ ---
        Container(ff_service, "Feature Flag Service", "Unleash / LaunchDarkly", "Управление конфигурацией и A/B тестами")

        %% Шина данных
        Container(kafka, "Message Broker", "Kafka/RabbitMQ", "Async Event Bus")

        %% Сервис аналитики
        Container(analytics, "Сервис аналитики", "ClickHouse / DWH", "Сбор событий для оценки экспериментов")

        Container(ms_catalog, "Каталог товаров", "Go + DB", "Товары, цены, рекомендации")

        Boundary(monolith_boundary, "Backend Monolith", "Legacy Core") {
            Component(mod_order, "Домен Заказ", "Order Logic", "Orchestrator")
            Component(mod_wh, "Домен Склад", "Inventory Logic", "Stock Management")
            Component(mod_log, "Домен Доставка", "Delivery Logic", "Shipping")
        }
    }

    %% --- УПРАВЛЕНИЕ ЭКСПЕРИМЕНТАМИ ---
    Rel(marketer, ff_service, "UI/HTTPS", "Настройка сегментов")

    %% Сервисы опрашивают флаги (ИСПРАВЛЕНО: стрелки идут от конкретных контейнеров)
    Rel(ext_gw, ff_service, "SDK/HTTP", "Check: UI Flags")
    Rel(ms_catalog, ff_service, "SDK/HTTP", "Check: Algo Flags")
    Rel(mod_order, ff_service, "SDK/HTTP", "Check: Logic Flags") 

    %% --- 1. СИНХРОННЫЕ ПОТОКИ ---
    Rel(user, ext_gw, "HTTPS", "UI Requests")
    Rel(ext_gw, int_gw, "REST", "GET /products")
    Rel(int_gw, ms_catalog, "REST", "Query Products")
    Rel(mod_order, int_gw, "gRPC", "Sync: GetPrice()") 
    Rel(int_gw, ms_catalog, "gRPC", "Reply Price")

    %% --- 2. АСИНХРОННЫЕ ПОТОКИ (EDA) ---
    Rel(ext_gw, mod_order, "REST", "POST /order")
    Rel(mod_order, kafka, "Pub: OrderCreated", "Start Process (+Ctx)")

    Rel(kafka, mod_wh, "Sub: OrderCreated", "Trigger Reserve")
    Rel(mod_wh, kafka, "Pub: StockReserved", "Reserve Result")

    Rel(kafka, mod_order, "Sub: StockReserved", "Check Status")
    Rel(mod_order, kafka, "Pub: OrderConfirmed", "Payment Success")

    Rel(kafka, mod_log, "Sub: OrderConfirmed", "Trigger Shipping")
    Rel(mod_log, kafka, "Pub: DeliveryScheduled", "Delivery Info")

    %% --- 3. АНАЛИТИКА ---
    Rel(kafka, analytics, "Sub: All Events", "ETL (с меткой A/B)")
    Rel(analyst, analytics, "SQL", "Сравнение метрик")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")