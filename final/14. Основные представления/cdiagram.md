```mermaid

sequenceDiagram
    actor U as User
    participant App as Mobile App
    participant TS as Training Service
    participant K as Kafka Broker
    participant GS as Gamification Svc
    participant IS as Inventory Svc

    U->>App: Завершить тренировку
    App->>TS: POST /api/training/finish
    activate TS
    TS->>TS: Сохранить в DB (статус: processing)
    TS->>K: Publish Event: TrainingFinished
    TS-->>App: 200 OK (Тренировка принята)
    deactivate TS

    par Parallel Processing
        K->>GS: Consume Event
        activate GS
        GS->>GS: Расчет очков (XP)
        GS->>GS: Обновление Leaderboard (Redis)
        deactivate GS
    and
        K->>IS: Consume Event
        activate IS
        IS->>IS: Расчет пробега
        IS->>IS: Проверка износа (Alert?)
        deactivate IS
    end