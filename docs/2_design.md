# 2. Проєктування архітектури системи

## 2.1. Загальна архітектура та контекст системи (C4 Level 1)

На контекстному рівні Pet Care System розглядається як єдина програмна система, що взаємодіє з власниками домашніх
тварин, співвласниками, ветеринарами та зовнішніми сервісами.

Основним інтерфейсом користувача є Android-застосунок. Серверна частина забезпечує керування профілями тварин, медичною
історією, нагадуваннями та геолокаційними даними.

### Діаграма 1. System Context

```mermaid
%%{init: {'themeVariables': {'fontSize': '16px'}}}%%
flowchart TD
    classDef person fill: #08427b, stroke: #052e56, color: #ffffff, rx: 12, ry: 12
    classDef system fill: #1168bd, stroke: #0b4884, color: #ffffff, rx: 6, ry: 6
    classDef extSystem fill: #6c757d, stroke: #495057, color: #ffffff, rx: 6, ry: 6
    subgraph Actors [" Actors "]
        direction LR
        owner["<b>Власник</b><br/><i>[Person]</i><br/><br/>Керує профілями тварин,<br/>геозонами і доглядом"]:::person
        coowner["<b>Співвласник</b><br/><i>[Person]</i><br/><br/>Має делегований доступ<br/>до догляду за тваринами"]:::person
        vet["<b>Ветеринар</b><br/><i>[Person]</i><br/><br/>Працює з медичними<br/>записами тварин"]:::person
        admin["<b>Системний адміністратор</b><br/><i>[Person]</i><br/><br/>Керує обліковими<br/>записами"]:::person
    end

%% Цільова система
    petcare["<b>Pet Care System</b><br/><i>[Software System]</i><br/><br/>Android-застосунок і backend-сервіси<br/>для догляду за тваринами, медичної історії,<br/>нагадувань та геозон"]:::system
%% Зовнішні сервіси
    subgraph External [" External "]
        direction LR
        fcm["<b>Firebase Cloud Messaging</b><br/><i>[Software System]</i><br/><br/>Сервіс доставки push-повідомлень"]:::extSystem
        maps["<b>Google Maps Platform</b><br/><i>[Software System]</i><br/><br/>Картографічні дані та геокодування"]:::extSystem
    end

    owner -->|" REST / HTTPS "| petcare
    coowner -->|" REST / HTTPS "| petcare
    vet -->|" REST / HTTPS "| petcare
    admin -->|" REST / HTTPS "| petcare
    petcare -->|" FCM API / HTTPS "| fcm
    petcare -->|" Maps SDK / HTTPS "| maps
    style Actors fill: none, stroke: none
    style External fill: none, stroke: none
```

### Опис взаємодій

- Власник створює профілі тварин, додає медичні записи та налаштовує нагадування.
- Співвласник отримує доступ до спільної інформації відповідно до наданих прав.
- Ветеринар переглядає інформацію, до якої йому надано доступ.
- Адміністратор виконує адміністративні операції.
- Firebase Cloud Messaging використовується для доставки push-сповіщень.
- Google Maps SDK використовується для відображення географічних координат.
- GPS Simulator виступає джерелом тестових координат на початковому етапі розроблення.

## 2.2. Контейнери системи (C4 Level 2)

На контейнерному рівні система поділяється на Android-клієнт, три серверні мікросервіси, брокер повідомлень та сховища
даних.

### Діаграма 2. Container Diagram

```mermaid
%%{init: {'themeVariables': {'fontSize': '16px'}}}%%
flowchart TD
%% Палітра C4 Model
    classDef person fill: #08427b, stroke: #052e56, color: #ffffff, rx: 12, ry: 12
    classDef container fill: #438dd5, stroke: #2e6295, color: #ffffff, rx: 6, ry: 6
    classDef db fill: #438dd5, stroke: #2e6295, color: #ffffff, rx: 18, ry: 18
    classDef extSystem fill: #6c757d, stroke: #495057, color: #ffffff, rx: 6, ry: 6
%% Актори
subgraph Actors [" Actors "]
direction LR
owner["<b>Власник</b><br/><i>[Person]</i>"]:::person
coowner["<b>Співвласник</b><br/><i>[Person]</i>"]:::person
vet["<b>Ветеринар</b><br/><i>[Person]</i>"]:::person
admin["<b>Адміністратор</b><br/><i>[Person]</i>"]:::person
end

%% Межа системи
subgraph System ["Pet Care System [Software System]"]
direction TB

%% Шар 1: Клієнти та вхідна точка
subgraph Layer1 [" Layer1 "]
direction LR
android["<b>Android App</b><br/><i>[Container: Kotlin / Compose / Room]</i><br/>Мобільний клієнт з офлайн-кешем"]:::container
simulator["<b>GPS Simulator</b><br/><i>[Container: Python]</i><br/>Генерація тестових GPS-координат"]:::container
end

%% Шар 2: Мікросервіси
subgraph Layer2 [" Layer2 "]
direction LR
pet["<b>Pet Service</b><br/><i>[Container: FastAPI]</i><br/>Профілі, медісторія, Event Sourcing"]:::container
reminder["<b>Reminder Service</b><br/><i>[Container: FastAPI]</i><br/>Планування та надсилання нагадувань"]:::container
geo["<b>Geo Service</b><br/><i>[Container: FastAPI]</i><br/>Геозони та обробка координат"]:::container
end

%% Шар 3: Дані та брокер
subgraph Layer3 [" Layer3 "]
direction LR
postgres[("<b>PostgreSQL</b><br/><i>[Container: PostgreSQL]</i><br/>Event Store, Outbox та дані сервісів")]:::db
kafka[["<b>Apache Kafka</b><br/><i>[Container: Kafka]</i><br/>Асинхронний обмін подіями"]]:::container
redis[("<b>Redis</b><br/><i>[Container: Redis]</i><br/>Кеш останньої позиції")]:::db
end
end

%% Зовнішні системи
subgraph External [" Layer4 "]
direction LR
maps["<b>Google Maps Platform</b><br/><i>[Software System]</i><br/>Картографічні дані"]:::extSystem
fcm["<b>Firebase Cloud Messaging</b><br/><i>[Software System]</i><br/>Push-повідомлення"]:::extSystem
end

owner & coowner & vet & admin -->|" Android UI / HTTPS "|android

%% Вхідні запити
android -->|" REST API / HTTPS, JWT "|pet
android -->|"REST API / HTTPS, JWT "|reminder
android -->|" REST API / HTTPS, JWT "| geo
simulator -->|" REST API / HTTP"|geo

pet & reminder & geo -->|" SQL / TCP "|postgres
geo <-->|" Redis RESP / TCP "|redis

pet -->|" Kafka Protocol / TCP "|kafka
kafka -->|" Kafka Protocol / TCP "|reminder
geo <-->|" Kafka Protocol / TCP "|kafka

%% Зовнішні інтеграції
android -->|" Maps SDK / HTTPS "|maps
reminder -->|" FCM HTTP v1 API / HTTPS "|fcm

%% Стилізація невидимих підгруп та межі системи
style Actors fill: none, stroke: none
style Layer1 fill: none, stroke: none
style Layer2 fill: none, stroke: none
style Layer3 fill: none, stroke: none
style External fill: none, stroke: none
style System fill: none, stroke: #94a3b8, stroke-width: 2px, stroke-dasharray: 5 5, color: #e2e8f0
```

### Опис контейнерів

**Android Application**

Мобільний клієнт, розроблений на Kotlin із використанням Jetpack Compose. Забезпечує взаємодію користувача із системою
та надсилає запити до серверних API.

**Pet Service**

Відповідає за профілі домашніх тварин та їхню медичну історію. Використовує Event Sourcing: зміни предметної області
зберігаються як послідовність доменних подій. Для надійної публікації подій використовується Transactional Outbox.

**Reminder Service**

Відповідає за створення, редагування, виконання та планування нагадувань. Отримує релевантні події через Kafka та
використовує PostgreSQL для зберігання власних даних.

**Geo Service**

Відповідає за отримання, збереження та надання географічних координат. На початковому етапі використовує GPS Simulator
як джерело тестових координат.

**Apache Kafka**

Забезпечує асинхронний обмін подіями між серверними сервісами. Дає змогу незалежно підключати споживачів та повторно
читати збережені повідомлення в межах політики зберігання.

**PostgreSQL**

Використовується як основне реляційне сховище. Кожен сервіс володіє власними даними. У Pet Service зберігаються Event
Store та Outbox.

**Redis**

Допоміжне сховище для кешування даних. Не є основним джерелом достовірного стану системи.

## 2.3. Компонентна архітектура Pet Service (C4 Level 3)

Для деталізації обрано Pet Service, оскільки саме він реалізує Event Sourcing та механізм Transactional Outbox. Його
внутрішня структура демонструє розділення бізнес-логіки, прикладних сценаріїв і деталей інфраструктури.

### Діаграма 3. Component Diagram — Pet Service

```mermaid
%%{init: {'themeVariables': {'fontSize': '15px'}}}%%
flowchart TD
%% Палітра C4 Component
    classDef component fill:#85bbf0,stroke:#5d82a8,color:#000000,rx:6,ry:6
    classDef extContainer fill:#6c757d,stroke:#495057,color:#ffffff,rx:6,ry:6
    classDef extDb fill:#6c757d,stroke:#495057,color:#ffffff,rx:18,ry:18

%% Зовнішній клієнт
    android["<b>Android App</b><br/><i>[Container: Kotlin]</i><br/>Мобільний клієнт"]:::extContainer

%% Межа сервісу
subgraph petService ["Pet Service — FastAPI [Container]"]
direction TB
api["<b>Pet Controller</b><br/><i>[Component: FastAPI]</i><br/>Обробляє HTTP-запити"]:::component
business["<b>Pet Manager</b><br/><i>[Component: Python]</i><br/>Обробляє профілі та медичні записи"]:::component

subgraph DataAccess [" Data Access"]
direction LR
database["<b>Database Manager</b><br/><i>[Component: SQLAlchemy]</i><br/>Читає та записує дані"]:::component
events["<b>Event Handler</b><br/><i>[Component: Python]</i><br/>Формує та надсилає події"]:::component
end
end

%% Зовнішні сховища
subgraph External [" External "]
direction LR
postgres[("<b>PostgreSQL</b><br/><i>[Container: Database]</i><br/>Зберігання даних")]:::extDb
kafka[["<b>Apache Kafka</b><br/><i>[Container: Message Broker]</i><br/>Обмін подіями"]]:::extContainer
end

%% Взаємодії
android -->|"HTTP"| api
api -->|"Викликає методи"| business
api -->|"Прямий доступ до БД"| database
business -->|"Зберігає дані"| database
business -->|"Створює події"| events
database -->|"SQL / TCP"| postgres
events -->|"Kafka / TCP"| kafka

style petService fill:none,stroke:#94a3b8,stroke-width:2px,stroke-dasharray: 5 5,color:#e2e8f0
style DataAccess fill:none,stroke:none
style External fill:none,stroke:none
```

### 2.3.1. Призначення компонентів

| Компонент                 | Відповідальність                                                       |
|---------------------------|------------------------------------------------------------------------|
| Pet REST Controller       | Приймає HTTP-запити та передає їх у прикладний рівень                  |
| Request / Response Mapper | Перетворює DTO запитів і відповідей                                    |
| Pet Use Cases             | Координує прикладні сценарії роботи з тваринами                        |
| Command Handler           | Обробляє команди на зміну стану                                        |
| Query Handler             | Забезпечує сценарії читання даних                                      |
| Pet Aggregate             | Інкапсулює стан та бізнес-правила агрегату тварини                     |
| Domain Events             | Представляє факти змін у предметній області                            |
| Domain Rules              | Містить правила та інваріанти предметної області                       |
| Event Store Port          | Абстракція доступу до сховища подій                                    |
| Outbox Write Port         | Абстракція запису повідомлень до Outbox                                |
| Unit of Work              | Координує транзакційне збереження змін                                 |
| Event Store Adapter       | Реалізує доступ до Event Store                                         |
| Outbox Writer Adapter     | Реалізує запис повідомлень до Outbox                                   |
| Event Store Repository    | Виконує операції з таблицями Event Store                               |
| Outbox Repository         | Виконує операції з таблицею Outbox                                     |
| Outbox Publisher Worker   | Фоново знаходить непубліковані повідомлення та ініціює їхню публікацію |
| Kafka Producer Adapter    | Відповідає за технічну взаємодію з Apache Kafka                        |

### 2.3.2. Сценарій запису доменної події

1. Користувач надсилає запит через Android Application.
2. Pet REST Controller приймає запит та передає його до відповідного Use Case.
3. Command Handler завантажує поточний стан агрегату шляхом відтворення його подій.
4. Pet Aggregate перевіряє бізнес-правила та формує нову доменну подію.
5. Unit of Work координує збереження нової події в Event Store та відповідного запису в Outbox.
6. Event Store Adapter і Outbox Writer Adapter виконують операції з PostgreSQL у межах однієї транзакції.
7. Після успішного commit Outbox Publisher Worker знаходить запис, який очікує публікації.
8. Kafka Producer Adapter надсилає повідомлення до відповідного Kafka topic.
9. Після підтвердження публікації запис Outbox позначається як оброблений.

Таким чином, зміна в Event Store та створення запису Outbox є атомарними в межах однієї транзакції PostgreSQL.
Публікація в Kafka виконується окремим фоновим процесом.

### 2.3.3. Надійність обміну подіями

У системі використовується модель доставки at-least-once. Це означає, що повідомлення може бути доставлене повторно.
Тому споживачі повинні перевіряти ідентифікатори подій та забезпечувати ідемпотентну обробку.

Для подій, пов'язаних з однією твариною, як Kafka message key використовується `pet_id`. Це дозволяє зберігати порядок
повідомлень у межах однієї partition. Глобальний порядок між усіма повідомленнями системи не гарантується.

## 2.4. Трасування вимог в архітектуру

Трасування встановлює зв'язок між вимогами з [SRS](srs.md), контейнерами та компонентами C4, а також архітектурним
рішенням [ADR-001](adr/0001-choice-of-technology.md).

| Вимоги SRS              | Архітектурний елемент                                               | Реалізація / обґрунтування                                                           |
|-------------------------|---------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| SRS-FUN-01 – SRS-FUN-05 | Android Application, Pet Service, Reminder Service, Geo Service     | Керування користувачами та доступом                                                  |
| SRS-FUN-06 – SRS-FUN-14 | Pet Service, Pet Aggregate, Pet REST Controller                     | Керування профілями тварин                                                           |
| SRS-FUN-15 – SRS-FUN-23 | Pet Service, Event Store, Pet Aggregate, Domain Events              | Медична історія та збереження історії змін                                           |
| SRS-FUN-24 – SRS-FUN-32 | Reminder Service, Apache Kafka, Firebase Cloud Messaging            | Створення нагадувань та сповіщення                                                   |
| SRS-FUN-33 – SRS-FUN-37 | Geo Service, GPS Simulator, PostgreSQL, Google Maps SDK             | Отримання та відображення координат                                                  |
| SRS-FUN-38 – SRS-FUN-43 | Pet Service, Outbox Publisher Worker, Apache Kafka, споживачі подій | Публікація та оброблення доменних подій; [ADR-001](adr/0001-choice-of-technology.md) |
| SRS-NFR-01 – SRS-NFR-03 | Серверні сервіси, REST API, Kafka                                   | Продуктивність та незалежна обробка запитів і подій                                  |
| SRS-NFR-04 – SRS-NFR-07 | PostgreSQL, Event Store, Transactional Outbox                       | Цілісність та надійність збереження і передавання подій                              |
| SRS-NFR-08 – SRS-NFR-11 | REST API та механізми автентифікації й авторизації                  | Захист даних і контроль доступу                                                      |
| SRS-NFR-12 – SRS-NFR-15 | Розділення сервісів, порти та адаптери Pet Service                  | Супроводжуваність та розширюваність                                                  |
| SRS-NFR-16 – SRS-NFR-18 | Android Application, Google Maps SDK                                | Платформна сумісність та зручність використання                                      |

### 2.4.1. Зв'язок архітектури з ADR-001

Вибір Apache Kafka для асинхронної взаємодії між сервісами обґрунтовано
в [ADR-001: Вибір брокера повідомлень](adr/0001-choice-of-technology.md).

Kafka використовується для передавання доменних подій від Pet Service до інших сервісів. Її модель збереження потоку
подій, незалежні consumer groups та можливість повторного читання повідомлень відповідають обраному підходу Event
Sourcing.

Transactional Outbox у Pet Service забезпечує узгодженість між збереженням доменної події в PostgreSQL та подальшою
публікацією в Kafka.


### 2.4.2.1. ER-діаграма схеми `pet`

Схема `pet` містить основні сутності системи та компоненти реалізації Event Sourcing. Діаграма відображає зв'язки між користувачами, домашніми тваринами, правами доступу, запрошеннями, журналом подій, проєкціями та таблицею Outbox.

```mermaid
erDiagram
    PET_USERS {
        uuid id PK
        varchar email UK
        varchar password_hash
        varchar display_name
        timestamptz created_at
        timestamptz updated_at
        timestamptz deleted_at
    }

    PET_REFRESH_TOKENS {
        uuid id PK
        uuid user_id FK
        varchar token_hash UK
        uuid family_id
        timestamptz expires_at
        timestamptz revoked_at
        timestamptz created_at
    }

    PET_PETS {
        uuid id PK
        varchar name
        varchar species
        varchar breed
        date birth_date
        varchar sex
        varchar microchip_id
        timestamptz created_at
        timestamptz updated_at
    }

    PET_ACCESS {
        uuid id PK
        uuid pet_id FK
        uuid user_id FK
        varchar role
        uuid granted_by FK
        timestamptz granted_at
        timestamptz revoked_at
    }

    PET_INVITATIONS {
        uuid id PK
        uuid pet_id FK
        uuid created_by FK
        varchar invitee_email
        varchar token_hash UK
        varchar status
        timestamptz expires_at
        timestamptz accepted_at
        timestamptz created_at
    }

    PET_EVENT_STORE {
        uuid event_id PK
        uuid aggregate_id
        varchar aggregate_type
        bigint version
        varchar event_type
        jsonb payload
        uuid author_id FK
        timestamptz occurred_at
        timestamptz recorded_at
    }

    PET_OUTBOX {
        uuid id PK
        uuid event_id FK
        varchar destination
        jsonb payload
        varchar status
        integer attempts
        timestamptz next_attempt_at
        timestamptz published_at
        timestamptz created_at
    }

    PET_PROFILES {
        uuid pet_id PK
        varchar name
        varchar species
        varchar breed
        date birth_date
        varchar sex
        varchar microchip_id
        bigint version
        timestamptz updated_at
    }

    PET_PROCEDURES {
        uuid id PK
        uuid pet_id
        varchar procedure_type
        varchar description
        date performed_at
        uuid performed_by
        uuid source_event_id
        timestamptz created_at
    }

    PET_PROJECTION_CHECKPOINTS {
        varchar projection_name PK
        bigint last_position
        timestamptz updated_at
    }

    PET_USERS ||--o{ PET_REFRESH_TOKENS : owns
    PET_USERS ||--o{ PET_ACCESS : receives
    PET_USERS ||--o{ PET_ACCESS : grants
    PET_PETS ||--o{ PET_ACCESS : has
    PET_USERS ||--o{ PET_INVITATIONS : creates
    PET_PETS ||--o{ PET_INVITATIONS : has
    PET_USERS ||--o{ PET_EVENT_STORE : authors
    PET_PETS ||--|| PET_PROFILES : projected_as
    PET_EVENT_STORE ||--o{ PET_OUTBOX : publishes
```

### 2.4.2.2. ER-діаграма схеми `geo`

Схема `geo` призначена для зберігання геолокаційних даних і визначення стану перебування тварин у заданих геозонах. Основними сутностями є геозони, стани геозон, вимірювання координат і записи Outbox.

```mermaid
erDiagram
    GEO_GEOZONES {
        uuid id PK
        uuid pet_id
        varchar name
        varchar shape_type
        numeric center_lat
        numeric center_lon
        numeric radius_m
        jsonb polygon
        boolean enabled
        integer version
        timestamptz created_at
        timestamptz updated_at
    }

    GEO_ZONE_STATES {
        uuid id PK
        uuid geozone_id FK
        uuid pet_id
        varchar current_state
        varchar candidate_state
        integer consecutive_confirmations
        timestamptz last_measurement_at
        timestamptz changed_at
    }

    GEO_MEASUREMENTS {
        uuid id PK
        uuid pet_id
        numeric latitude
        numeric longitude
        numeric accuracy_m
        numeric altitude_m
        numeric speed_mps
        varchar source
        timestamptz recorded_at
        timestamptz received_at
    }

    GEO_OUTBOX {
        uuid id PK
        uuid measurement_id
        varchar event_type
        jsonb payload
        varchar status
        integer attempts
        timestamptz created_at
        timestamptz published_at
    }

    GEO_GEOZONES ||--o{ GEO_ZONE_STATES : tracks
```

Для таблиці `geo_measurements` передбачається секціонування за полем `recorded_at`. З цієї причини первинний ключ у фізичній базі даних формується як складений ключ `(id, recorded_at)`. На ER-діаграмі тип ідентифікатора показано окремо для зручності читання, тому повне фізичне визначення ключа необхідно враховувати під час реалізації SQL-схеми.

Поле `pet_id` у таблицях `geozones`, `zone_states` і `geo_measurements` є логічним посиланням на тварину, зареєстровану в сервісі `pet`. Воно не реалізується як фізичний зовнішній ключ між схемами.

### 2.4.2.3. ER-діаграма схеми `reminder`

Схема `reminder` забезпечує створення, зберігання та доставку нагадувань користувачам. Вона містить інформацію про пристрої, підписки, заплановані нагадування та результати їх доставки.

```mermaid
erDiagram
    REMINDER_DEVICE_TOKENS {
        uuid id PK
        uuid user_id
        varchar token UK
        varchar platform
        boolean active
        timestamptz created_at
        timestamptz updated_at
        timestamptz last_used_at
    }

    REMINDER_SUBSCRIBERS {
        uuid id PK
        uuid user_id
        uuid pet_id
        boolean active
        jsonb preferences
        timestamptz created_at
        timestamptz updated_at
    }

    REMINDER_REMINDERS {
        uuid id PK
        uuid pet_id
        uuid user_id
        varchar title
        text description
        varchar reminder_type
        timestamptz scheduled_at
        varchar status
        uuid source_event_id
        timestamptz created_at
        timestamptz updated_at
    }

    REMINDER_DELIVERIES {
        uuid id PK
        uuid reminder_id FK
        uuid device_token_id FK
        integer attempt_no
        varchar status
        varchar provider_message_id
        text error_message
        timestamptz attempted_at
        timestamptz delivered_at
    }

    REMINDER_PROCESSED_EVENTS {
        varchar consumer_name PK
        uuid event_id PK
        timestamptz processed_at
    }

    REMINDER_REMINDERS ||--o{ REMINDER_DELIVERIES : generates
    REMINDER_DEVICE_TOKENS ||--o{ REMINDER_DELIVERIES : receives
```

Поля `user_id`, `pet_id` і `source_event_id` у схемі `reminder` використовуються як логічні посилання на дані інших сервісів. Це дає змогу сервісу нагадувань працювати незалежно від фізичної структури таблиць сервісу керування тваринами.

### 2.4.3. Висновок

Спроєктована архітектура розділяє клієнтську частину, серверні сервіси та інфраструктуру обміну повідомленнями. Pet
Service виділено як компонент із найбільш складною доменною логікою через використання Event Sourcing та Transactional
Outbox.

Трасування пов'язує вимоги SRS із контейнерами та внутрішніми компонентами системи, а ADR-001 фіксує обґрунтування
вибору брокера повідомлень. Документація підтримується у Markdown безпосередньо в Git-репозиторії відповідно до принципу
Docs-as-Code.

## 2.5. Специфікація міжсервісних контрактів (API Contracts)

## 2.5.1. Вибір технології та межі контракту

Для специфікації REST-інтерфейсу системи використовується стандарт **OpenAPI 3.0.3** у форматі YAML.

Контракт описує взаємодію між контейнерами **Frontend SPA** та **Backend API (API Gateway)**. Він визначає структуру HTTP-запитів і відповідей, правила валідації вхідних даних, механізм авторизації та формати повідомлень про помилки.

Для демонстрації контракту обрано сценарій керування домашніми тваринами. Він охоплює три основні операції:

- отримання списку домашніх тварин авторизованого користувача;
- створення нової домашньої тварини;
- отримання інформації про конкретну тварину за її ідентифікатором.

Авторизація здійснюється за допомогою JWT Bearer Token. Ідентифікатор користувача визначається сервером на основі токена, а не передається клієнтом у тілі запиту. Це запобігає можливості створення записів від імені іншого користувача шляхом підміни `user_id`.

Контракт описує зовнішній REST-інтерфейс Backend API. Внутрішня взаємодія Backend API з Pet Service не є частиною цього контракту та може реалізовуватися через окремі внутрішні інтерфейси.

## 2.5.2. Опис API-операцій

У межах контракту визначено три операції для роботи з ресурсом `Pet`.

| HTTP-метод | Endpoint        | Призначення                                                   |
|------------|-----------------|---------------------------------------------------------------|
| GET        | `/pets`         | Отримання списку тварин, доступних авторизованому користувачу |
| POST       | `/pets`         | Створення нової тварини                                       |
| GET        | `/pets/{petId}` | Отримання інформації про конкретну тварину                    |

### GET /pets — отримання списку тварин

Операція повертає список домашніх тварин, доступних авторизованому користувачу відповідно до його прав доступу.

Для керування обсягом відповіді передбачено два query-параметри:

- `limit` — максимальна кількість записів у відповіді. Допустимий діапазон — від 1 до 100, значення за замовчуванням — 20.
- `offset` — кількість записів, які потрібно пропустити. Значення за замовчуванням — 0.

Успішна відповідь має статус `200 OK` і містить масив тварин та метадані пагінації.

### POST /pets — створення тварини

Операція створює новий запис про домашню тварину.

Клієнт передає дані у форматі JSON відповідно до схеми `CreatePetRequest`. Ідентифікатор власника не передається в тілі запиту: Backend API визначає його на основі перевіреного JWT.

У разі успішного створення сервер повертає статус `201 Created`, URI створеного ресурсу в заголовку `Location` та представлення створеної тварини у форматі `PetResponse`.

### GET /pets/{petId} — отримання тварини за ідентифікатором

Операція повертає інформацію про конкретну тварину за її UUID.

Перед поверненням даних Backend API перевіряє наявність ресурсу та права доступу автентифікованого користувача. Якщо користувач не має необхідних прав, повертається `403 Forbidden`. Якщо ресурс не знайдено — `404 Not Found`.

Деталізовану специфікацію ендпоінтів у форматі OpenAPI 3.0 зафіксовано у файлі
[openapi.yaml](./api/openapi.yaml)

## 2.6. Моделювання логіки критичних вузлів системи

У цьому підрозділі наведено алгоритми трьох критичних вузлів системи Pet Care: обробки команд із використанням Event Sourcing, визначення перетину геозон та обробки подій для надсилання push-сповіщень. Алгоритми описують послідовність операцій, умови переходів, обробку помилок і граничних випадків.

Умовні позначення:

- `:=` — оператор присвоювання;
- `=` — оператор порівняння на рівність;
- `AND`, `OR`, `NOT` — логічні оператори;
- `Events[i]`, `Geofences[i]`, `Reminders[i]` — елементи відповідних колекцій;
- `MAX_RETRIES` — максимальна кількість спроб;
- `TIMEOUT` — граничний час очікування відповіді зовнішнього сервісу.

---

## 2.6.1. Алгоритм обробки команди в Pet Service з використанням Event Sourcing та Outbox

**Призначення.** Алгоритм забезпечує обробку команди зміни даних домашньої тварини. Поточний стан агрегату відновлюється на основі послідовності незмінних подій. Після перевірки команди нові події записуються до Event Store, а відповідні повідомлення — до Outbox у межах однієї транзакції PostgreSQL. Окремий процес публікує повідомлення Outbox у Kafka.

**Вхідні дані:**

- `Command` — команда зміни даних тварини;
- `aggregateId` — ідентифікатор агрегату Pet;
- `commandId` — унікальний ідентифікатор команди;
- `expectedVersion` — очікувана версія агрегату.

**Вихідні дані:**

- результат виконання команди;
- нові події `NewEvents[i]`;
- записи Outbox для подальшої публікації в Kafka.

**Основні змінні:**

- `Events[i]` — збережені події агрегату;
- `State` — відновлений стан агрегату;
- `CurrentVersion` — поточна версія агрегату;
- `NewEvents[i]` — події, сформовані в результаті команди;
- `OutboxRecord[i]` — записи для публікації в Kafka;
- `RetryCount` — кількість спроб публікації.

### Блок-схема алгоритму

```mermaid
flowchart TD
    A([Початок]) --> B[/Отримати Command, commandId, aggregateId, expectedVersion/]
    B --> C{"Command валідна AND aggregateId != null?"}
    C -- Ні --> X1[Повернути помилку валідації]
    C -- Так --> D[Перевірити commandId]
    D --> E{"commandId вже оброблено?"}
    E -- Так --> Y[/Повернути збережений результат/]
    E -- Ні --> F[Завантажити Events_i з Event Store]
    F --> G[State := Fold Events_i CurrentVersion := остання версія або -1]
    G --> H{"expectedVersion = CurrentVersion?"}
    H -- Ні --> X2[Повернути конфлікт версії]
    H -- Так --> I{"Бізнес-правила виконано?"}
    I -- Ні --> X3[Повернути помилку бізнес-валідації]
    I -- Так --> J[NewEvents_i := Decide Command, State]
    J --> K{"NewEvents порожній?"}
    K -- Так --> Y
    K -- Ні --> L[BEGIN TRANSACTION]
    L --> M[Додати NewEvents_i до Event Store]
    M --> N[OutboxRecord_i := сформувати записи для NewEvents_i]
    N --> O[Додати OutboxRecord_i до Outbox]
    O --> P{"Запис і COMMIT успішні?"}
    P -- Ні --> Q[ROLLBACK]
    Q --> X4[Повернути помилку збереження]
    P -- Так --> R[/Повернути успішний результат/]
    R --> S([Кінець])
    
    X1 --> S
    X2 --> S
    X3 --> S
    X4 --> S
    Y --> S
```

### Блок-схема публікації Outbox

```mermaid
flowchart TD
    A([Початок]) --> B[Отримати записи Outbox зі статусом PENDING]
    B --> C{"Існує OutboxRecord_i?"}
    C -- Ні --> Z([Кінець])
    C -- Так --> D[RetryCount := 0]
    D --> E[Опублікувати OutboxRecord_i у Kafka]
    E --> F{"Kafka підтвердила публікацію?"}
    F -- Так --> G[Status := PUBLISHED]
    G --> H[Перейти до наступного OutboxRecord_i]
    F -- Ні --> I[RetryCount := RetryCount + 1]
    I --> J{"RetryCount < MAX_RETRIES?"}
    J -- Так --> K[Застосувати затримку повторної спроби]
    K --> E
    J -- Ні --> L[Status := FAILED зафіксувати помилку]
    L --> H
    H --> C
```

**Примітка щодо надійності.** Запис події та Outbox-запис виконуються атомарно в одній транзакції PostgreSQL. Публікація в Kafka відбувається окремо, тому можлива повторна доставка повідомлення у разі збою між публікацією та оновленням статусу Outbox. Споживачі подій мають враховувати можливість повторної доставки та забезпечувати ідемпотентну обробку за ідентифікатором події.

## 2.6.2. Алгоритм визначення входу та виходу з геозони в Geo Service

**Призначення.** Алгоритм визначає перетин геозони домашньою твариною на основі GPS-координат. Для зменшення хибних перемикань стану використовується гістерезис: умови входу та виходу мають різні порогові значення відстані.

**Вхідні дані:**

- `GPS[j]` — чергова GPS-позиція;
- `Geofences[i]` — множина налаштованих геозон;
- `R[i]` — радіус геозони;
- `H[i]` — ширина гістерезису;
- `A_MAX` — максимально допустима похибка GPS;
- `LastTimestamp` — час останньої прийнятої позиції.

**Вихідні дані:**

- оновлений стан перебування у геозоні;
- подія `GEOFENCE_ENTER` або `GEOFENCE_EXIT` у Kafka, якщо відбувся перехід.

**Основні змінні:**

- `Distance[i]` — відстань від GPS-позиції до центру геозони;
- `State[i]` — поточний стан геозони: `INSIDE` або `OUTSIDE`;
- `NewState[i]` — обчислений стан;
- `Event[i]` — сформована подія переходу.

### Блок-схема алгоритму

```mermaid
flowchart TD
    A([Початок]) --> B[/Отримати GPS_j/]
    B --> C{"Координати, deviceId і timestamp наявні?"}
    C -- Ні --> X1[Відхилити GPS_j]
    C -- Так --> D{"Координати в діапазоні AND accuracy <= A_MAX?"}
    D -- Ні --> X1
    D -- Так --> E{"GPS_j.timestamp > LastTimestamp?"}
    E -- Ні --> X2[Відхилити застарілу позицію]
    E -- Так --> F[Отримати Geofences_i]
    F --> G{"Geofences не порожній?"}
    G -- Ні --> H[Оновити останню прийняту GPS-позицію]
    H --> Z([Кінець])
    G -- Так --> I[Вибрати Geofences_i]
    I --> J[Distance_i := обчислити відстань до центру]
    J --> K[State_i := отримати стан із Redis]
    K --> L{"State_i відсутній?"}
    L -- Так --> M[State_i := початковий стан за Distance_i]
    M --> N[NewState_i := State_i]
    L -- Ні --> O{"State_i = OUTSIDE AND Distance_i <= R_i - H_i?"}
    O -- Так --> P[NewState_i := INSIDE]
    O -- Ні --> Q{"State_i = INSIDE AND Distance_i >= R_i + H_i?"}
    Q -- Так --> R[NewState_i := OUTSIDE]
    Q -- Ні --> S[NewState_i := State_i]
    P --> T
    R --> T
    S --> T
    N --> U
    T{"NewState_i != State_i?"}
    U[Перейти до наступного елемента]
    T -- Ні --> V[Зберегти актуальний стан у Redis]
    T -- Так --> W[Сформувати Event_i з унікальним eventId]
    W --> AA{"Kafka підтвердила публікацію?"}
    AA -- Ні --> AB[Залишити позицію для повторної обробки]
    AB --> AC([Кінець])
    AA -- Так --> AD[State_i := NewState_i оновити Redis]
    AD --> U
    V --> U
    U --> AE{"Існує наступна Geofences_i?"}
    AE -- Так --> I
    AE -- Ні --> AF[Оновити LastTimestamp]
    AF --> Z
    X1 --> Z
    X2 --> Z
```

**Примітка щодо реалізації.** Для обчислення `Distance[i]` може використовуватися формула гаверсинусів. Параметри `R[i]`, `H[i]` та `A_MAX` мають зберігатися в конфігурації геозони або сервісу. Публікація в Kafka та оновлення Redis не є однією атомарною транзакцією. Тому повторна обробка може породити повторну доставку події, а споживачі повинні підтримувати ідемпотентність за `eventId`.

## 2.6.3. Алгоритм обробки події та надсилання push-сповіщення в Reminder Service

**Призначення.** Алгоритм забезпечує обробку повідомлення, отриманого з Kafka, пошук відповідних активних нагадувань та надсилання push-сповіщення користувачу через Firebase Cloud Messaging (FCM).

**Вхідні дані:**

- `Event` — повідомлення з Kafka;
- `eventId` — унікальний ідентифікатор події;
- `eventType` — тип події;
- `petId` — ідентифікатор тварини;
- `occurredAt` — час виникнення події;
- `Reminders[i]` — набір нагадувань, пов'язаних із твариною та типом події.

**Вихідні дані:**

- результат обробки події;
- статус надсилання повідомлення;
- запис про успішну або невдалу доставку.

**Основні змінні:**

- `ProcessedEvent` — ознака попередньої обробки події;
- `MatchingReminders[i]` — нагадування, що відповідають події;
- `Notification[i]` — сформоване push-сповіщення;
- `RetryCount[i]` — кількість спроб надсилання;
- `MAX_RETRIES` — максимальна кількість спроб;
- `FCMResponse` — результат відповіді FCM.

### Блок-схема алгоритму

```mermaid
flowchart TD
    A([Початок]) --> B[/Отримати Event із Kafka/]
    B --> C{"eventId, eventType і обов'язкові поля наявні?"}
    C -- Ні --> X1[Зафіксувати некоректне повідомлення]
    C -- Так --> D{"eventId вже оброблено?"}
    D -- Так --> Y[/Підтвердити повторне повідомлення Kafka/]
    D -- Ні --> E[Отримати Reminders_i з PostgreSQL]
    E --> F[MatchingReminders_i := відфільтрувати за petId та eventType]
    F --> G{"MatchingReminders не порожній?"}
    G -- Ні --> H[Зафіксувати відсутність відповідних нагадувань]
    H --> Y
    G -- Так --> I[Вибрати MatchingReminders_i]
    I --> J{"Нагадування активне AND доставка ще не зафіксована?"}
    J -- Ні --> K[Перейти до наступного нагадування]
    J -- Так --> L[Notification_i := сформувати повідомлення]
    L --> M[RetryCount_i := 0]
    M --> N[Надіслати Notification_i через FCM]
    N --> O{"FCM повернув успішну відповідь?"}
    O -- Так --> P[DeliveryStatus_i := SENT]
    O -- Ні --> Q[RetryCount_i := RetryCount_i + 1]
    Q --> R{"RetryCount_i < MAX_RETRIES?"}
    R -- Так --> S[Затримка повторної спроби]
    S --> N
    R -- Ні --> T[DeliveryStatus_i := FAILED]
    P --> U[Зберегти результат доставки в PostgreSQL]
    T --> U
    U --> V{"Результат збережено?"}
    V -- Ні --> W[Не підтверджувати повідомлення Kafka]
    W --> Z([Кінець])
    V -- Так --> K
    K --> AA{"Існує наступне MatchingReminders_i?"}
    AA -- Так --> I
    AA -- Ні --> AB[Зафіксувати eventId як оброблений]
    AB --> AC{"Результат обробки збережено?"}
    AC -- Ні --> W
    AC -- Так --> Y
    Y --> AD([Кінець])
    X1 --> AD
```

**Примітка щодо доставки.** Kafka та FCM не утворюють спільної транзакції. Тому алгоритм не гарантує абсолютну доставку рівно один раз. Він забезпечує контроль повторної обробки на рівні `eventId` і збережених статусів. У разі аварії між прийняттям повідомлення FCM та фіксацією статусу в PostgreSQL можливе повторне надсилання. Цей ризик потрібно враховувати під час реалізації механізму доставки.

## 2.7. Інтеграційний аудит та верифікація цілісності доменів

## Офіційний технічний висновок (Production Readiness Report)

За результатами проведеного архітектурного інтеграційного аудиту встановлено, що модель даних PostgreSQL 16, зовнішній REST API-контракт OpenAPI 3.0.3 та алгоритми Pet Service, Geo Service і Reminder Service можуть функціонувати в межах єдиної узгодженої архітектури системи Pet Care.

Перевірка підтвердила:

1. **Узгодженість структури даних та API.** Поля REST-контракту мають відповідники в моделі даних. Типи ідентифікаторів, правила доступу та валідації визначені без суперечностей.
2. **Узгодженість Pet Service.** Механізми Event Sourcing, контролю версій агрегатів, ідемпотентності команд та Outbox узгоджені з призначенням таблиць домену `pet`.
3. **Узгодженість Geo Service.** Модель даних підтримує збереження геозон, вимірювань, поточних станів і результатів алгоритму визначення переходів. Конфігураційні параметри та повторна публікація подій визначені на архітектурному рівні.
4. **Узгодженість Reminder Service.** Модель підтримує облік нагадувань, пристроїв, доставок та оброблених подій. Передбачено механізми ідемпотентності логічних операцій та контролю повторної обробки.
5. **Узгодженість міжсервісної взаємодії.** Відповідальність REST API, Kafka та FCM розмежована. Логічні посилання між доменами не порушують принцип незалежності сервісів.

Зауваження, виявлені під час первинного аудиту, опрацьовані шляхом уточнення ролей сутностей, обмежень цілісності, правил ідемпотентності та форматів міжсервісної взаємодії. За умови відображення зазначених рішень у фінальних специфікаціях моделі даних та алгоритмів, **архітектурних конфліктів між СУБД, REST API-контрактом і логікою алгоритмів не виявлено**.

**Підсумковий статус: архітектурний інтеграційний аудит пройдено. Проєктна документація узгоджена на рівні моделей даних, API та алгоритмів.**