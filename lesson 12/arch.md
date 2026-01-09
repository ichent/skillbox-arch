```mermaid
C4Container
    title C4 Architecture: Security + EDA + Operations (Fixed)

    Person(user, "Клиент", "Browser / Mobile")
    Person(attacker, "Злоумышленник", "Пытается украсть данные")
    Person(marketer, "Маркетолог", "Business Owner", "Управляет Feature Flags")
    Person(analyst, "Аналитик", "BI Staff", "Смотрит отчеты")

    %% === 1. DMZ (Public Zone) ===
    Boundary(dmz_zone, "DMZ (Public Zone)", "Internet Access") {
        Container(waf, "WAF", "Cloud Armor", "Защита от атак")
        Container(ext_gw, "External Gateway", "Kong / Nginx", "TLS, Rate Limiting")
    }

    %% === 2. APP ZONE (Private Network) ===
    Boundary(app_zone, "Application Zone", "No Direct Internet Access") {

        %% Инфраструктура безопасности и Ops
        Container(auth_service, "Auth Service", "Keycloak", "SSO, 2FA/OTP")
        Container(vault, "Secrets Manager", "Vault", "Хранение секретов")
        Container(int_gw, "Internal Gateway", "Envoy", "Service Mesh")
        Container(ff_service, "Feature Flag Service", "Unleash", "A/B Configs")

        %% Сервисы
        Container(ms_catalog, "Catalog Service", "Go", "Товары")

        Boundary(monolith_boundary, "Backend Monolith", "Legacy Core") {
            Component(mod_sec, "Sec Middleware", "Lib", "Security Check")
            Component(mod_order, "Order Domain", "Java", "Orders")
            Component(mod_wh, "Warehouse Domain", "Java", "Inventory")
            Component(mod_log, "Logistics Domain", "Java", "Delivery")
        }
    }

    %% === 3. DATA ZONE (Restricted) ===
    Boundary(data_zone, "Data Zone", "Restricted Access") {
        ContainerDb(db_catalog, "Catalog DB", "PostgreSQL", "Encrypted")
        ContainerDb(db_core, "Core DB", "PostgreSQL", "Encrypted & Masked")
        Container(kafka, "Message Broker", "Kafka", "Events")
        Container(analytics, "Analytics Service", "ClickHouse", "Data Lake")
    }

    %% --- СВЯЗИ ---

    %% 1. Security & Auth
    Rel(attacker, waf, "Attacks", "Blocked")
    Rel(user, waf, "HTTPS", "Requests")
    Rel(waf, ext_gw, "Proxies", "Clean Traffic")
    Rel(ext_gw, auth_service, "Verify/Redirect", "Auth Check")
    Rel(auth_service, user, "Send OTP", "2FA")

    %% Исправлено: Связь идет от компонента, а не от границы
    Rel(mod_sec, vault, "Get Secrets", "Credentials") 
    Rel(ms_catalog, vault, "Get Secrets", "Credentials")

    %% 2. Feature Flags (Все связи на месте)
    Rel(marketer, ff_service, "UI", "Manage Flags")
    Rel(ext_gw, ff_service, "SDK", "Check UI Flags")
    Rel(ms_catalog, ff_service, "SDK", "Check Algo Flags")
    Rel(mod_order, ff_service, "SDK", "Check Logic Flags")

    %% 3. Internal Gateway (Синхронные вызовы на месте)
    Rel(ext_gw, int_gw, "REST", "Route Internal")
    Rel(int_gw, ms_catalog, "REST", "Get Products")
    Rel(mod_order, int_gw, "gRPC", "Sync Price Check")
    Rel(int_gw, ms_catalog, "gRPC", "Price Reply")

    %% 4. EDA (Асинхронные вызовы на месте)
    Rel(ext_gw, mod_order, "REST", "Create Order")
    Rel(mod_order, kafka, "Pub: OrderCreated", "Start")
    Rel(kafka, mod_wh, "Sub: OrderCreated", "Reserve")
    Rel(mod_wh, kafka, "Pub: StockReserved", "Result")
    Rel(kafka, mod_order, "Sub: StockReserved", "Update")
    Rel(mod_order, kafka, "Pub: OrderConfirmed", "Pay OK")
    Rel(kafka, mod_log, "Sub: OrderConfirmed", "Ship")

    %% 5. Data & Analytics
    Rel(ms_catalog, db_catalog, "SQL", "RW")
    Rel(mod_order, db_core, "SQL", "RW")
    Rel(kafka, analytics, "Sub: All", "ETL")
    Rel(analyst, analytics, "SQL", "Audit")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")