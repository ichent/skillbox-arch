```mermaid

graph TD
    %% Акторы
    User(Пользователь приложения)
    Employee(Сотрудник)

    %% Границы мобильного приложения
    subgraph "Мобильное Приложение"

        %% Интеграционный слой
        IntModule[Модуль интеграции / Единый ID]

        %% Подсистемы интерфейса и бизнес-логики
        subgraph Core_Logic [Ядро Системы]
            Profile[Профили пользователей]
            Workouts[Управление тренировками]
            Inventory[Управление инвентарем]
        end

        subgraph Engagement [Вовлечение]
            Gamification[Геймификация]
            Promo[Промоакции и реклама]
        end

        subgraph Services [Сервисы]
            Notify[Уведомления]
        end

        %% Слой работы с железом и API
        subgraph Hardware_Layer [Слой оборудования]
            SensorHandler[Работа с внешними датчиками]
            PhoneHealthHandler[Работа с фитнес-функциями телефона]
        end
    end

    %% Внешние системы (УДАЛЕНО: Система оплаты)
    PhoneAPI[("Встроенные фитнес-функции")]
    ExtSensors[("Внешние датчики\n(пульс, O2)")]
    CompanyApps[("Существующие\nприложения компании")]

    %% Связи
    User --> Profile
    User --> Workouts

    %% Связь с экосистемой
    IntModule --> Profile
    IntModule <--> CompanyApps

    Employee --> Promo

    %% Внутренние связи
    Workouts --> Gamification
    Workouts --> Profile
    Gamification --> Inventory
    Promo --> Notify
    Notify --> User

    %% Внешние связи (фитнес)
    ExtSensors --> SensorHandler
    PhoneAPI --> PhoneHealthHandler

    SensorHandler --> Workouts
    PhoneHealthHandler --> Workouts