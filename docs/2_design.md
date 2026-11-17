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

### 2.4.2. Висновок

Спроєктована архітектура розділяє клієнтську частину, серверні сервіси та інфраструктуру обміну повідомленнями. Pet
Service виділено як компонент із найбільш складною доменною логікою через використання Event Sourcing та Transactional
Outbox.

Трасування пов'язує вимоги SRS із контейнерами та внутрішніми компонентами системи, а ADR-001 фіксує обґрунтування
вибору брокера повідомлень. Документація підтримується у Markdown безпосередньо в Git-репозиторії відповідно до принципу
Docs-as-Code.