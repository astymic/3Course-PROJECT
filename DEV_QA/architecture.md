# Архітектурна схема системи — LiLu E-Commerce MVP

**Автор:** Мульков Максим (Dev/QA, частково SA)
**Дата:** 06.04.2026
**Версія:** 2.0

---

## 1. Загальна архітектура системи (Three-Tier)

```mermaid
graph TB
    subgraph CLIENT["🖥️ Клієнтський рівень (Frontend)"]
        CAT["📦 Каталог товарів<br/>+ фільтри"]
        CARD["👟 Картка товару<br/>фото, розміри"]
        CART["🛒 Кошик &<br/>Замовлення"]
        POPUP["💬 Popup-чат<br/>консультація"]
        PROF["👤 Особистий<br/>кабінет"]
    end

    subgraph SERVER["⚙️ Серверний рівень (Backend API)"]
        M_CAT["Модуль каталогу"]
        M_ORD["Модуль замовлень"]
        M_AUTH["Модуль авторизації"]
        M_STOCK["Модуль складу"]
        M_PAY["Модуль оплати"]
        M_ADMIN["Адмін-панель API"]
    end

    subgraph DATA["💾 Рівень даних"]
        PG["🐘 PostgreSQL<br/>(основна БД)"]
        FILES["📁 Cloudinary<br/>(фото товарів)"]
        REDIS["⚡ Redis<br/>(кеш, сесії)"]
    end

    CLIENT -->|"HTTPS / REST API"| SERVER
    SERVER -->|"SQL / Prisma ORM"| DATA

    style CLIENT fill:#1a1a2e,stroke:#e94560,color:#fff
    style SERVER fill:#16213e,stroke:#0f3460,color:#fff
    style DATA fill:#0f3460,stroke:#533483,color:#fff
```

---

## 2. Діаграма взаємодії компонентів

```mermaid
graph LR
    BUYER["🛍️ Покупець<br/>(браузер)"] --> FRONT["🌐 Vercel<br/>Next.js Frontend"]
    MANAGER["👔 Менеджер<br/>(браузер)"] --> ADMIN["🔧 Адмін-панель<br/>Next.js"]

    FRONT -->|"REST API"| BACK["⚙️ Backend<br/>Node.js + Express"]
    ADMIN -->|"REST API"| BACK

    BACK --> PG["🐘 PostgreSQL<br/>(Render DB)"]
    BACK --> CLOUD["☁️ Cloudinary<br/>(фото)"]
    BACK --> LIQPAY["💳 LiqPay API<br/>(оплата)"]
    BACK --> NP["🚚 Нова Пошта API<br/>(доставка)"]

    FRONT --> TAWK["💬 Tawk.to<br/>(popup-чат)"]

    style BUYER fill:#e94560,stroke:#fff,color:#fff
    style MANAGER fill:#533483,stroke:#fff,color:#fff
    style FRONT fill:#0f3460,stroke:#e94560,color:#fff
    style ADMIN fill:#0f3460,stroke:#533483,color:#fff
    style BACK fill:#16213e,stroke:#0f3460,color:#fff
    style PG fill:#336791,stroke:#fff,color:#fff
    style CLOUD fill:#3448c5,stroke:#fff,color:#fff
    style LIQPAY fill:#7ab800,stroke:#fff,color:#fff
    style NP fill:#e2001a,stroke:#fff,color:#fff
    style TAWK fill:#1cc761,stroke:#fff,color:#fff
```

---

## 3. ER-діаграма бази даних

```mermaid
erDiagram
    Category ||--o{ Product : "має"
    Product ||--|{ ProductSize : "має розміри"
    Product ||--o{ OrderItem : "у замовленнях"
    User ||--o{ Order : "створює"
    Order ||--|{ OrderItem : "містить"
    Product ||--o{ ProductVariant : "варіанти (Фаза 2)"

    Category {
        int id PK
        string name
        string slug
        string season "літнє / зимове"
        int parent_id FK "підкатегорії"
    }

    Product {
        int id PK
        string name
        text description
        decimal price
        int category_id FK
        string material
        json images "масив URL фото"
        boolean is_active
        datetime created_at
    }

    ProductSize {
        int id PK
        int product_id FK
        int size "35, 36, 37..."
        int quantity "залишок на складі"
        boolean is_available "автоматично"
    }

    User {
        int id PK
        string email
        string phone
        string name
        string password_hash
        enum role "buyer / manager / admin"
    }

    Order {
        int id PK
        int user_id FK
        enum status "new / paid / shipped / done"
        enum payment_method "card / cod"
        enum delivery_method "nova_poshta / courier"
        string np_ttn "номер ТТН"
        decimal total_amount
        datetime created_at
    }

    OrderItem {
        int id PK
        int order_id FK
        int product_id FK
        int size
        int quantity
        decimal price
    }

    ProductVariant {
        int id PK
        int product_id FK
        string sole_type "ФАЗА 2"
        string material "ФАЗА 2"
        string lining "ФАЗА 2"
        string color "ФАЗА 2"
        decimal price_modifier "ФАЗА 2"
    }
```

> ⚠️ `ProductVariant` — НЕ реалізується в MVP. Закладена в схему для Фази 2 (конструктор).

---

## 4. Обраний технологічний стек

| Компонент | Технологія | Обґрунтування |
|---|---|---|
| **Frontend** | Next.js (React) | SSR для SEO каталогу, швидкий рендеринг |
| **CSS** | Tailwind CSS | Швидка розробка адаптивного UI |
| **Backend** | Node.js + Express | Єдина мова JS/TS, швидкість розробки |
| **ORM** | Prisma | Типізація, міграції, зручна робота з PostgreSQL |
| **База даних** | PostgreSQL | Надійна, безкоштовна, JSON для варіацій |
| **Кешування** | Redis (Upstash) | Швидкий доступ до залишків, сесії |
| **Фото** | Cloudinary | Зберігання та автооптимізація зображень |
| **Оплата** | LiqPay (ПриватБанк) | Найпоширеніший шлюз в Україні |
| **Доставка** | Нова Пошта API | Офіційний API, розрахунок вартості + ТТН |
| **Popup-чат** | Tawk.to | Безкоштовний, вимога клієнта |
| **Хостинг** | Vercel + Railway/Render | Free tiers для MVP |
| **Домен** | .ua (Hostiq) | Географічна прив'язка |

---

## 5. Потік замовлення (User Journey)

```mermaid
sequenceDiagram
    actor Buyer as 🛍️ Покупець
    participant Front as 🌐 Frontend
    participant Back as ⚙️ Backend
    participant DB as 🐘 PostgreSQL
    participant LP as 💳 LiqPay
    participant NP as 🚚 Нова Пошта

    Buyer->>Front: Відкриває каталог
    Front->>Back: GET /api/products?size=38
    Back->>DB: SELECT з фільтрами
    DB-->>Back: Товари + залишки
    Back-->>Front: JSON список
    Front-->>Buyer: Каталог з наявністю

    Buyer->>Front: Додає товар у кошик
    Buyer->>Front: Оформлює замовлення

    alt Оплата карткою
        Front->>Back: POST /api/orders
        Back->>DB: Створює Order + зменшує quantity
        Back->>LP: Створює платіж
        LP-->>Buyer: Форма оплати
        Buyer->>LP: Оплачує
        LP-->>Back: Callback (success)
        Back->>DB: status → paid
    else Накладений платіж
        Front->>Back: POST /api/orders (payment=cod)
        Back->>DB: Створює Order, status → new
    end

    Back->>NP: Створює ТТН
    NP-->>Back: Номер ТТН
    Back->>DB: Зберігає np_ttn
    Back-->>Buyer: Підтвердження замовлення + ТТН
```

---

## 6. Ключові архітектурні рішення

### 6.1. Облік складу в реальному часі
- Наявність у таблиці `ProductSize` (запис для кожного розміру).
- При замовленні `quantity` зменшується автоматично.
- `quantity = 0` → розмір «Немає в наявності» на сайті і в адмін-панелі.

### 6.2. Підготовка до конструктора (Фаза 2)
- Таблиця `ProductVariant` закладена в ER-схему.
- Комбінації: підошва × матеріал × підклад × колір.

### 6.3. Безпека
- bcrypt для паролів, JWT для авторизації.
- HTTPS (SSL від Vercel / Let's Encrypt).
- Валідація через Prisma ORM (захист від SQL-ін'єкцій).

### 6.4. Mobile-first
- Більшість покупців Наталії приходять з Instagram → з телефону.
- Адаптивний дизайн: мобільний → планшет → десктоп.

---

## 7. Зовнішні інтеграції

| Інтеграція | Провайдер | Що робить | Sprint |
|---|---|---|:---:|
| Оплата карткою | LiqPay | Прийом онлайн-платежів | 2 |
| Накладений платіж | Власна логіка | Позначка "оплата при отриманні" | 2 |
| Доставка НП | Нова Пошта API | Вибір відділення, ТТН | 2 |
| Кур'єр | Власна логіка | Фіксована ціна по місту | 2 |
| Popup-чат | Tawk.to | Зв'язок з менеджером | 2 |
| Фото товарів | Cloudinary | Завантаження, оптимізація | 1 |
