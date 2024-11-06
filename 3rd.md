# Работа с параметрами, асинхронность и подключение к PostgreSQL в разработке API на Rust

## 1. Работа с параметрами

### Path-параметры

Path-параметры используются для передачи переменных значений в URL-адресе. Например, в маршруте `/users/{id}` параметр `{id}` будет заменен на конкретный идентификатор пользователя.

#### Пример с Actix-web

1. **Редактируйте `src/main.rs`:**

```rust
use actix_web::{get, web, App, HttpResponse, HttpServer, Responder};
use serde::Serialize;

#[derive(Serialize)]
struct User {
    id: u32,
    name: String,
}

#[get("/users/{id}")]
async fn get_user(path: web::Path<u32>) -> impl Responder {
    let user_id = path.into_inner();
    let user = User {
        id: user_id,
        name: format!("User {}", user_id),
    };
    HttpResponse::Ok().json(user)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(get_user)
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

2. **Запустите сервер:**

```bash
cargo run
```

3. **Проверьте маршрут:**

```bash
curl http://127.0.0.1:8080/users/1
```

**Ответ:**

```json
{
  "id": 1,
  "name": "User 1"
}
```

### Query-параметры

Query-параметры используются для передачи дополнительных данных в запросе через URL после символа `?`. Например, `/search?query=rust`.

#### Пример с Actix-web

1. **Редактируйте `src/main.rs`:**

```rust
use actix_web::{get, web, App, HttpResponse, HttpServer, Responder};
use serde::Serialize;

#[derive(Serialize)]
struct SearchResult {
    query: String,
    results: Vec<String>,
}

#[get("/search")]
async fn search(query: web::Query<std::collections::HashMap<String, String>>) -> impl Responder {
    let query_str = query.get("query").unwrap_or(&"".to_string()).clone();
    let results = vec![
        format!("Result for {}", query_str),
        format!("Another result for {}", query_str),
    ];
    let response = SearchResult {
        query: query_str,
        results,
    };
    HttpResponse::Ok().json(response)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(search)
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

2. **Запустите сервер:**

```bash
cargo run
```

3. **Проверьте маршрут:**

```bash
curl "http://127.0.0.1:8080/search?query=rust"
```

**Ответ:**

```json
{
  "query": "rust",
  "results": [
    "Result for rust",
    "Another result for rust"
  ]
}
```

---
## 2. Асинхронность в запросах

### Основы асинхронного программирования в Rust

Rust предоставляет мощную поддержку асинхронного программирования с помощью `async`/`await`. Это позволяет эффективно обрабатывать множество запросов без блокировки потока выполнения.

### Использование `async`/`await` в обработчиках

В Actix-web все обработчики являются асинхронными функциями, что позволяет использовать преимущества асинхронного выполнения.

#### Пример асинхронного обработчика

1. **Редактируйте `src/main.rs`:**

```rust
use actix_web::{get, web, App, HttpResponse, HttpServer, Responder};
use serde::Serialize;
use tokio::time::{sleep, Duration};

#[derive(Serialize)]
struct DelayedResponse {
    message: String,
    delay_seconds: u64,
}

#[get("/delay/{seconds}")]
async fn delay(path: web::Path<u64>) -> impl Responder {
    let seconds = path.into_inner();
    sleep(Duration::from_secs(seconds)).await;
    let response = DelayedResponse {
        message: format!("Waited for {} seconds", seconds),
        delay_seconds: seconds,
    };
    HttpResponse::Ok().json(response)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(delay)
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

2. **Запустите сервер:**

```bash
cargo run
```

3. **Проверьте маршрут:**

```bash
curl http://127.0.0.1:8080/delay/5
```

**Ответ (после 5 секунд задержки):**

```json
{
  "message": "Waited for 5 seconds",
  "delay_seconds": 5
}
```

---
## 3. Подключение к базе данных PostgreSQL

Для взаимодействия с PostgreSQL мы будем использовать библиотеку [`sqlx`](https://github.com/launchbadge/sqlx), которая предоставляет асинхронный ORM и поддержку миграций.

### Шаг 1: Установка `sqlx`

1. **Добавьте зависимости в `Cargo.toml`:**

```toml
[dependencies]
actix-web = "4"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
sqlx = { version = "0.6", features = ["postgres", "runtime-tokio-native-tls", "macros"] }
dotenv = "0.15"
tokio = { version = "1", features = ["full"] }
```

2. **Установите `sqlx-cli` для миграций:**

```bash
cargo install sqlx-cli --no-default-features --features postgres,rustls
```

### Шаг 2: Настройка подключения к PostgreSQL

1. **Создайте файл `.env` в корне проекта и добавьте строку подключения:**

```
DATABASE_URL=postgres://username:password@localhost/database_name
```

*Замените `username`, `password` и `database_name` на ваши реальные данные.*

2. **Редактируйте `src/main.rs`:**

```rust
use actix_web::{get, web, App, HttpResponse, HttpServer, Responder};
use serde::Serialize;
use sqlx::postgres::PgPoolOptions;
use std::env;
use dotenv::dotenv;

#[derive(Serialize)]
struct User {
    id: i32,
    name: String,
}

#[get("/users/{id}")]
async fn get_user(path: web::Path<i32>, pool: web::Data<sqlx::PgPool>) -> impl Responder {
    let user_id = path.into_inner();
    let user = sqlx::query_as!(
        User,
        "SELECT id, name FROM users WHERE id = $1",
        user_id
    )
    .fetch_one(pool.get_ref())
    .await;

    match user {
        Ok(user) => HttpResponse::Ok().json(user),
        Err(_) => HttpResponse::NotFound().body("User not found"),
    }
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    dotenv().ok();
    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL must be set");

    // Создаем пул подключений
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(&database_url)
        .await
        .expect("Failed to create pool");

    HttpServer::new(move || {
        App::new()
            .app_data(web::Data::new(pool.clone()))
            .service(get_user)
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

### Шаг 3: Создание таблицы пользователей

1. **Создайте директорию `migrations`:**

```bash
mkdir migrations
```

2. **Создайте миграцию для создания таблицы `users`:**

```bash
sqlx migrate add create_users_table
```

Это создаст файл миграции в папке `migrations`. Откройте его и добавьте SQL-запрос для создания таблицы:

```sql
-- migrations/20240101000000_create_users_table.sql

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);
```

3. **Примените миграцию:**

```bash
sqlx migrate run
```

Если все настроено правильно, таблица `users` будет создана в вашей базе данных.

---
## 4. Миграции базы данных

Миграции позволяют управлять изменениями в структуре базы данных. Мы уже создали первую миграцию для создания таблицы `users`. Рассмотрим, как создавать и применять дополнительные миграции.

### Создание новой миграции

1. **Создайте миграцию для добавления столбца `email` в таблицу `users`:**

```bash
sqlx migrate add add_email_to_users
```

2. **Редактируйте созданный файл миграции:**

```sql
-- migrations/20240101000100_add_email_to_users.sql

ALTER TABLE users
ADD COLUMN email TEXT NOT NULL DEFAULT '';
```

### Применение миграций

1. **Примените миграцию:**

```bash
sqlx migrate run
```

2. **Проверьте структуру таблицы:**

Подключитесь к базе данных и выполните команду:

```sql
\d users
```

Вы должны увидеть, что таблица `users` теперь имеет столбец `email`.

### Откат миграции

Если необходимо откатить последнюю миграцию:

```bash
sqlx migrate revert
```

---
## 5. Работа с окружением

### Использование переменных окружения

Переменные окружения позволяют хранить конфиденциальные данные и настройки, такие как строки подключения к базе данных, ключи API и другие параметры.

### Работа с `.env` файлами

Для удобства можно использовать файл `.env`, который хранит переменные окружения. Библиотека [`dotenv`](https://crates.io/crates/dotenv) автоматически загружает переменные из этого файла.

### Настройка конфигурации приложения

1. **Создайте файл `.env` в корне проекта:**

```
DATABASE_URL=postgres://username:password@localhost/database_name
SERVER_ADDRESS=127.0.0.1
SERVER_PORT=8080
```

2. **Редактируйте `src/main.rs` для использования дополнительных переменных:**

```rust
use actix_web::{get, web, App, HttpResponse, HttpServer, Responder};
use serde::Serialize;
use sqlx::postgres::PgPoolOptions;
use std::env;
use dotenv::dotenv;

#[derive(Serialize)]
struct User {
    id: i32,
    name: String,
}

#[get("/users/{id}")]
async fn get_user(path: web::Path<i32>, pool: web::Data<sqlx::PgPool>) -> impl Responder {
    let user_id = path.into_inner();
    let user = sqlx::query_as!(
        User,
        "SELECT id, name FROM users WHERE id = $1",
        user_id
    )
    .fetch_one(pool.get_ref())
    .await;

    match user {
        Ok(user) => HttpResponse::Ok().json(user),
        Err(_) => HttpResponse::NotFound().body("User not found"),
    }
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    dotenv().ok();
    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    let server_address = env::var("SERVER_ADDRESS").unwrap_or_else(|_| "127.0.0.1".to_string());
    let server_port: u16 = env::var("SERVER_PORT")
        .unwrap_or_else(|_| "8080".to_string())
        .parse()
        .expect("SERVER_PORT must be a number");

    // Создаем пул подключений
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(&database_url)
        .await
        .expect("Failed to create pool");

    let bind_address = format!("{}:{}", server_address, server_port);

    HttpServer::new(move || {
        App::new()
            .app_data(web::Data::new(pool.clone()))
            .service(get_user)
    })
    .bind(bind_address)?
    .run()
    .await
}
```

3. **Запустите сервер:**

```bash
cargo run
```

Теперь сервер будет запускаться на адресе и порту, указанном в файле `.env`.

### Безопасность переменных окружения

- **Не добавляйте файл `.env` в систему контроля версий.** Добавьте его в `.gitignore`:

```bash
echo ".env" >> .gitignore
```

---

## Дополнительные ресурсы

- [Документация Actix-web](https://actix.rs/docs/)
- [Документация sqlx](https://docs.rs/sqlx/)
- [Документация dotenv](https://docs.rs/dotenv/)
- [The Rust Programming Language](https://doc.rust-lang.org/book/)
- [SQLx Migrations](https://docs.rs/sqlx/latest/sqlx/migrate/index.html)
