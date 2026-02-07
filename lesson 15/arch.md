```mermaid
C4Container
    title C4 Architecture + Observability

    %% === ACTORS ===
    Person(user, "Клиент", "Browser / Mobile")
    System_Ext(sensors, "IoT Sensors", "Devices")

    %% === 1. DMZ & 2. APP ZONE (Simplified for clarity) ===
    Boundary(app_zone, "Application Zone", "Бизнес-логика") {
        Container(iot_gw, "IoT Gateway", "Go/Java", "Запись логов в stdout")
        Container(monolith, "Backend Monolith", "Legacy Java", "Запись логов в файлы")
        Container(ms_services, "Microservices", "Go/Python", "Запись логов в stdout")

        %% AGENTS
        Container(log_agent, "Log Collectors", "Fluent-bit", "Sidecar/DaemonSet: Читает логи и шлет в Kafka")
    }

    %% === 3. DATA ZONE & OBSERVABILITY ===
    Boundary(data_zone, "Data & Observability Zone", "Restricted Access") {
        Container(kafka, "Message Broker", "Kafka", "Topic: SysLogs")

        %% NEW COMPONENTS
        Boundary(obs_stack, "Observability Stack", "Logging & Metrics") {
            Container(logstash, "Log Aggregator", "Logstash", "Reads Kafka, Parses, Masks PII")
            ContainerDb(elastic, "Log Storage", "Elasticsearch", "Indexes Logs")
            Container(kibana, "Kibana / Grafana", "UI", "Dashboards & Alerts")
        }
    }

    %% === RELATIONS ===
    Rel(sensors, iot_gw, "Telemetry")

    %% Logging Flow
    Rel(iot_gw, log_agent, "Доставка логов")
    Rel(monolith, log_agent, "Чтение файлов")
    Rel(ms_services, log_agent, "Доставка логов")

    Rel(log_agent, kafka, "Отправка логов", "TCP/Async")
    Rel(kafka, logstash, "Получение логов", "Batch")
    Rel(logstash, elastic, "Index Data", "HTTP")
    Rel(elastic, kibana, "Query", "HTTP")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")