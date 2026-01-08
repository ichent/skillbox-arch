```mermaid
C4Container
    title C4 Architecture: Final (EDA + Analytics + Hybrid Gateway)

    Person(user, "Клиент", "Пользователь приложения")
    Person(analyst, "Аналитик", "BI / Marketing", "Строит отчеты по продажам")

    Boundary(ecommerce_system, "E-commerce System") {

        Container(ext_gw, "External Gateway", "Kong/Nginx", "HTTPS, Auth, Public API")

        %% Internal Gateway нужен для синхронных запросов (чтение, быстрые проверки)
        Container(int_gw, "Internal Gateway", "K8s Service / Envoy", "Internal LB & Discovery")

        %% Шина данных (EDA)
        Container(kafka, "Message Broker", "Kafka/RabbitMQ", "Async Event Bus")

        %% Сервис аналитики (Сбор данных)
        Container(analytics, "Сервис аналитики", "ClickHouse / DWH", "Сбор событий и построение витрин данных")

        Container(ms_catalog, "Каталог товаров", "Go + DB", "Товары, цены")

        Boundary(monolith_boundary, "Backend Monolith (Modular)", "Legacy Core") {
            Component(mod_order, "Домен Заказ", "Order Logic", "Orchestrator")
            Component(mod_wh, "Домен Склад", "Inventory Logic", "Stock Management")
            Component(mod_log, "Домен Доставка", "Delivery Logic", "Shipping")
        }
    }

    %% --- 1. СИНХРОННЫЕ ПОТОКИ (Direct calls via Gateways) ---
    Rel(user, ext_gw, "HTTPS", "UI Requests")

    %% Чтение каталога (быстро, синхронно)
    Rel(ext_gw, int_gw, "REST", "GET /products")
    Rel(int_gw, ms_catalog, "REST", "Query Products")

    %% Монолит (Заказ) может синхронно уточнить цену перед стартом процесса
    Rel(mod_order, int_gw, "gRPC", "Sync: GetPrice()") 
    Rel(int_gw, ms_catalog, "gRPC", "Reply Price")


    %% --- 2. АСИНХРОННЫЕ ПОТОКИ (Event Driven Business Process) ---

    %% Шаг А: Создание заказа
    Rel(ext_gw, mod_order, "REST", "POST /order")
    Rel(mod_order, kafka, "Pub: OrderCreated", "Start Process")

    %% Шаг Б: Резерв товара (Склад)
    Rel(kafka, mod_wh, "Sub: OrderCreated", "Trigger Reserve")
    Rel(mod_wh, kafka, "Pub: StockReserved", "Reserve Result")

    %% Шаг В: Обработка ответа склада и Подтверждение
    Rel(kafka, mod_order, "Sub: StockReserved", "Check Status")
    Rel(mod_order, kafka, "Pub: OrderConfirmed", "Payment Success & Ready")

    %% Шаг Г: Логистика (Добавлено по вашему замечанию)
    Rel(kafka, mod_log, "Sub: OrderConfirmed", "Trigger Shipping")
    Rel(mod_log, kafka, "Pub: DeliveryScheduled", "Delivery Info")

    %% --- 3. АНАЛИТИКА (Добавлено по вашему замечанию) ---
    %% Сервис слушает всё
    Rel(kafka, analytics, "Sub: All Events", "ETL Process")
    %% Аналитик смотрит данные
    Rel(analyst, analytics, "SQL / BI Tool", "SELECT * FROM orders_view")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")