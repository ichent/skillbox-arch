```mermaid
C4Container
    title C4 Architecture: IoT Gateway as Unified Data Hub

    %% === ACTORS & EXTERNAL ===
    Person(user, "Клиент", "Browser / Mobile")
    Person(attacker, "Злоумышленник", "Пытается украсть данные")
    Person(distributor, "Дистрибьютор", "Partner", "Поставляет товары")
    System_Ext(sensors, "IoT Sensors", "Physical Devices", "GPS, Temp, Vibration")

    %% Staff
    Person(marketer, "Маркетолог", "Business Owner", "Управляет Feature Flags")
    Person(analyst, "Аналитик", "BI Staff", "Работает с качеством")

    %% === 1. DMZ ZONE ===
    Boundary(dmz_zone, "DMZ (Public Zone)", "Internet Access") {
        Container(waf, "WAF", "Cloud Armor", "Защита от атак")
        Container(ext_gw, "External Gateway", "Kong / Nginx", "TLS, Ingress, Rate Limiting")
    }

    %% === 2. APP ZONE ===
    Boundary(app_zone, "Application Zone", "No Direct Internet Access") {

        %% Infra
        Container(auth_service, "Auth Service", "Keycloak", "SSO, 2FA/OTP")
        Container(int_gw, "Internal Gateway", "Envoy", "VPN GW / Service Mesh")
        Container(ff_service, "Feature Flag Service", "Unleash", "Config Management")

        %% IoT & Quality Chain
        Container(iot_gw, "IoT Gateway", "Go/Java", "Aggregator & Data API") 
        Container(ms_iot, "IoT Service", "Python", "Writer / Persistence Logic")
        Container(ms_quality, "Quality Service", "Node/Go", "UI Backend / Dashboards")

        %% Business Services
        Container(ms_catalog, "Catalog Service", "Go", "Товары")
        Container(ms_logistics, "Logistics Service", "Go", "Доставка")

        %% Monolith
        Boundary(monolith_boundary, "Backend Monolith", "Legacy Core") {
            Component(mod_order, "Order Domain", "Java", "Orders")
            Component(mod_wh, "Warehouse Domain", "Java", "Inventory")
            Component(mod_clients, "Clients Domain", "Java", "CRM / Profiles")
            Component(mod_distributors, "Distributors Domain", "Java", "Partner Mgmt")
            Component(mod_suppliers, "Suppliers Domain", "Java", "Supply Chain")
        }
    }

    %% === 3. DATA ZONE ===
    Boundary(data_zone, "Data Zone", "Restricted Access") {
        ContainerDb(db_catalog, "Catalog DB", "PostgreSQL", "Encrypted")
        ContainerDb(db_core, "Core DB", "PostgreSQL", "Encrypted & Masked")
        Container(kafka, "Message Broker", "Kafka", "Events backbone")
        Container(analytics, "Analytics Service", "ClickHouse", "IoT Data Lake")
    }

    %% === СВЯЗИ ===

    %% 1. WRITE FLOW (Ingestion)
    Rel(sensors, waf, "MQTT/HTTPS", "Raw Telemetry")
    Rel(waf, ext_gw, "Proxies", "Traffic")
    Rel(ext_gw, iot_gw, "Stream", "Send Raw Data")

    %% Gateway агрегирует и отдает "писателю"
    Rel(iot_gw, ms_iot, "gRPC", "Push Aggregated Data")
    Rel(ms_iot, kafka, "Pub: SensorData", "Persist")
    Rel(kafka, analytics, "Sub: SensorData", "Save to ClickHouse")

    %% 2. READ FLOW (Quality Analysis) - UPDATED
    %% Аналитик заходит в UI
    Rel(analyst, int_gw, "HTTPS", "VPN Access")
    Rel(int_gw, ms_quality, "HTTP", "View Dashboard")

    %% Quality идет в IoT Gateway за данными
    Rel(ms_quality, iot_gw, "REST/gRPC", "Request Reports")

    %% IoT Gateway делает запрос в базу
    Rel(iot_gw, analytics, "SQL", "Fetch Analytics")

    %% 3. Public & Auth
    Rel(user, waf, "HTTPS", "Site Access")
    Rel(distributor, waf, "HTTPS", "Partner Portal")
    Rel(attacker, waf, "Attacks", "Blocked")
    Rel(ext_gw, auth_service, "Redirect/Verify", "OIDC Flow")
    Rel(auth_service, user, "Returns Token", "JWT")

    %% 4. Other Staff Access
    Rel(marketer, int_gw, "HTTPS", "VPN Access")
    Rel(int_gw, ff_service, "HTTP", "UI Access")

    %% 5. Business Routing
    Rel(ext_gw, int_gw, "REST", "Internal Route")
    Rel(int_gw, ms_catalog, "REST", "Get Products")
    Rel(int_gw, ms_logistics, "REST", "Track Order")

    %% 6. Monolith Logic
    Rel(ext_gw, mod_order, "REST", "Checkout")
    Rel(mod_order, int_gw, "gRPC", "Sync Price Check")
    Rel(int_gw, ms_catalog, "gRPC", "Price Reply")
    Rel(ext_gw, ff_service, "SDK", "Flags")

    %% 7. EDA (Standard Events)
    Rel(mod_order, kafka, "Pub: OrderCreated", "Start")
    Rel(kafka, mod_wh, "Sub: OrderCreated", "Reserve")
    Rel(mod_wh, kafka, "Pub: StockReserved", "Result")
    Rel(kafka, mod_order, "Sub: StockReserved", "Update")
    Rel(mod_order, kafka, "Pub: OrderConfirmed", "Pay OK")
    Rel(kafka, ms_logistics, "Sub: OrderConfirmed", "Ship")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")