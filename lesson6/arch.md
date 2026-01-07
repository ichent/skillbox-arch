```mermaid
C4Container
    title C4 Architecture: Monolith + Microservice Extraction

    Person(user, "Клиент", "Пользователь приложения/сайта")

    Boundary(ecommerce_system, "E-commerce System") {

        Container(ext_gw, "External Gateway", "API Gateway (Kong/Nginx)", "Маршрутизация внешнего трафика, SSL, Auth")
        Container(int_gw, "Internal Gateway", "Service Mesh / K8s Service", "Внутренняя маршрутизация и балансировка")

        Container(ms_catalog, "Catalog Microservice", "Go/Java + DB", "Хранение товаров, категорий, цен")

        Boundary(monolith_boundary, "Backend Monolith", "Legacy Application") {
            Component(mod_cs, "Кабинет клиента", "Customer Service", "Профили, история, настройки")
            Component(mod_order, "Заказ", "Order Domain", "Корзина, чекаут, оплата")
            Component(mod_wh, "Склад", "Warehouse Domain", "Учет остатков, резервирование")
            Component(mod_log, "Логистика", "Logistics Domain", "Расчет доставки, трекинг")
        }
    }

    %% Внешний трафик
    Rel(user, ext_gw, "HTTPS", "Запросы от UI/Mobile")

    %% Маршрутизация External Gateway
    Rel(ext_gw, int_gw, "HTTPS/REST", "Просмотр товаров / Поиск")
    Rel(ext_gw, mod_cs, "HTTPS/REST", "Логин, профиль")
    Rel(ext_gw, mod_order, "HTTPS/REST", "Оформление заказа")

    %% Внутреннее взаимодействие через Internal Gateway
    %% Монолиту (Заказу) нужны данные о товарах из микросервиса
    Rel(mod_order, int_gw, "gRPC/REST", "Запрос инфо о товаре (Sync)")
    Rel(int_gw, ms_catalog, "gRPC/REST", "Proxy request")

    %% Связи внутри монолита (In-process)
    Rel(mod_order, mod_wh, "Method Call", "Резерв товара")
    Rel(mod_order, mod_log, "Method Call", "Создание накладной")
    Rel(mod_cs, mod_order, "Method Call", "История заказов")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")