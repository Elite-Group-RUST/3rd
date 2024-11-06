# Основы работы с веб-фреймворками

## 1. Установка веб-фреймворков через Cargo

### Общие шаги для установки фреймворков

Перед тем как начать работу с любым фреймворком, необходимо установить Rust и Cargo — менеджер пакетов Rust. Если вы еще не установили Rust, следуйте инструкциям на [официальном сайте Rust](https://www.rust-lang.org/tools/install).

После установки Rust и Cargo, вы можете создавать новые проекты и добавлять необходимые зависимости.

### Установка Actix-web

**Actix-web** — это мощный и быстрый веб-фреймворк, основанный на акторной модели.

1. **Создайте новый проект:**

   ```bash
   cargo new actix_web_example
   cd actix_web_example
   ```

2. **Добавьте зависимости в `Cargo.toml`:**

   Откройте файл `Cargo.toml` и добавьте следующие строки под `[dependencies]`:

   ```toml
   [dependencies]
   actix-web = "4"
   serde = { version = "1.0", features = ["derive"] }
   serde_json = "1.0"
   ```

### Установка Rocket

**Rocket** — удобный и типобезопасный веб-фреймворк, известный своей простотой и элегантностью.

1. **Создайте новый проект:**

   ```bash
   cargo new rocket_example
   cd rocket_example
   ```

2. **Добавьте зависимости в `Cargo.toml`:**

   Откройте файл `Cargo.toml` и добавьте следующие строки под `[dependencies]`:

   ```toml
   [dependencies]
   rocket = { version = "0.5.0-rc.2", features = ["json"] }
   serde = { version = "1.0", features = ["derive"] }
   serde_json = "1.0"
   ```

### Установка Axum

**Axum** — современный веб-фреймворк, разработанный командой из проекта Tokio, ориентированный на высокую производительность и простоту использования.

1. **Создайте новый проект:**

   ```bash
   cargo new axum_example
   cd axum_example
   ```

2. **Добавьте зависимости в `Cargo.toml`:**

   Откройте файл `Cargo.toml` и добавьте следующие строки под `[dependencies]`:

   ```toml
   [dependencies]
   axum = "0.6"
   tokio = { version = "1", features = ["full"] }
   serde = { version = "1.0", features = ["derive"] }
   serde_json = "1.0"
   ```

---

## 2. Создание простого сервера

### Actix-web

Создадим простой сервер, который отвечает "Hello, Actix-web!" на корневом маршруте.

1. **Редактируйте `src/main.rs`:**

   ```rust
   use actix_web::{get, App, HttpServer, Responder};

   #[get("/")]
   async fn index() -> impl Responder {
       "Hello, Actix-web!"
   }

   #[actix_web::main]
   async fn main() -> std::io::Result<()> {
       HttpServer::new(|| {
           App::new()
               .service(index)
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

3. **Проверьте работу:**

   Откройте браузер и перейдите по адресу [http://127.0.0.1:8080/](http://127.0.0.1:8080/). Вы увидите сообщение "Hello, Actix-web!".

### Rocket

Создадим простой сервер, который возвращает JSON-ответ на корневом маршруте.

1. **Редактируйте `src/main.rs`:**

   ```rust
   #[macro_use] extern crate rocket;

   use rocket::serde::json::Json;
   use serde::Serialize;

   #[derive(Serialize)]
   struct Message {
       message: String,
   }

   #[get("/")]
   fn index() -> Json<Message> {
       Json(Message {
           message: "Hello, Rocket!".to_string(),
       })
   }

   #[launch]
   fn rocket() -> _ {
       rocket::build()
           .mount("/", routes![index])
   }
   ```

2. **Запустите сервер:**

   ```bash
   cargo run
   ```

3. **Проверьте работу:**

   Откройте браузер и перейдите по адресу [http://127.0.0.1:8000/](http://127.0.0.1:8000/). Вы увидите JSON-ответ `{"message":"Hello, Rocket!"}`.

### Axum

Создадим простой сервер, который возвращает JSON-ответ на корневом маршруте.

1. **Редактируйте `src/main.rs`:**

   ```rust
   use axum::{
       routing::get,
       Router,
       Json,
   };
   use serde::Serialize;
   use std::net::SocketAddr;

   #[derive(Serialize)]
   struct Message {
       message: String,
   }

   async fn index() -> Json<Message> {
       Json(Message {
           message: "Hello, Axum!".to_string(),
       })
   }

   #[tokio::main]
   async fn main() {
       // Создаем маршруты
       let app = Router::new()
           .route("/", get(index));

       // Определяем адрес сервера
       let addr = SocketAddr::from(([127, 0, 0, 1], 8080));
       println!("Server running at http://{}", addr);

       // Запускаем сервер
       axum::Server::bind(&addr)
           .serve(app.into_make_service())
           .await
           .unwrap();
   }
   ```

2. **Запустите сервер:**

   ```bash
   cargo run
   ```

3. **Проверьте работу:**

   Откройте браузер и перейдите по адресу [http://127.0.0.1:8080/](http://127.0.0.1:8080/). Вы увидите JSON-ответ `{"message":"Hello, Axum!"}`.

---

## 3. Создание базовых маршрутов

### Actix-web

Добавим дополнительные маршруты для обработки различных HTTP-методов.

1. **Редактируйте `src/main.rs`:**

   ```rust
   use actix_web::{get, post, delete, put, web, App, HttpResponse, HttpServer, Responder};
   use serde::{Deserialize, Serialize};

   #[derive(Serialize, Deserialize)]
   struct User {
       id: u32,
       name: String,
   }

   #[get("/")]
   async fn index() -> impl Responder {
       HttpResponse::Ok().body("Hello, Actix-web!")
   }

   #[get("/users/{id}")]
   async fn get_user(path: web::Path<u32>) -> impl Responder {
       let user = User {
           id: path.into_inner(),
           name: "John Doe".to_string(),
       };
       HttpResponse::Ok().json(user)
   }

   #[post("/users")]
   async fn create_user(user: web::Json<User>) -> impl Responder {
       HttpResponse::Created().json(user.into_inner())
   }

   #[put("/users/{id}")]
   async fn update_user(path: web::Path<u32>, user: web::Json<User>) -> impl Responder {
       HttpResponse::Ok().json(User {
           id: path.into_inner(),
           name: user.name.clone(),
       })
   }

   #[delete("/users/{id}")]
   async fn delete_user(path: web::Path<u32>) -> impl Responder {
       HttpResponse::NoContent().finish()
   }

   #[actix_web::main]
   async fn main() -> std::io::Result<()> {
       HttpServer::new(|| {
           App::new()
               .service(index)
               .service(get_user)
               .service(create_user)
               .service(update_user)
               .service(delete_user)
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

3. **Проверьте маршруты:**

   - **GET /users/1**

     ```bash
     curl http://127.0.0.1:8080/users/1
     ```

     **Ответ:**

     ```json
     {
       "id": 1,
       "name": "John Doe"
     }
     ```

   - **POST /users**

     ```bash
     curl -X POST http://127.0.0.1:8080/users \
     -H "Content-Type: application/json" \
     -d '{"id":2,"name":"Jane Smith"}'
     ```

     **Ответ:**

     ```json
     {
       "id": 2,
       "name": "Jane Smith"
     }
     ```

### Rocket

Добавим дополнительные маршруты для обработки различных HTTP-методов.

1. **Редактируйте `src/main.rs`:**

   ```rust
   #[macro_use] extern crate rocket;

   use rocket::serde::{json::Json, Deserialize, Serialize};

   #[derive(Serialize, Deserialize)]
   #[serde(crate = "rocket::serde")]
   struct User {
       id: u32,
       name: String,
   }

   #[get("/")]
   fn index() -> &'static str {
       "Hello, Rocket!"
   }

   #[get("/users/<id>")]
   fn get_user(id: u32) -> Json<User> {
       Json(User {
           id,
           name: "John Doe".to_string(),
       })
   }

   #[post("/users", format = "json", data = "<user>")]
   fn create_user(user: Json<User>) -> Json<User> {
       user
   }

   #[put("/users/<id>", format = "json", data = "<user>")]
   fn update_user(id: u32, user: Json<User>) -> Json<User> {
       Json(User {
           id,
           name: user.name.clone(),
       })
   }

   #[delete("/users/<id>")]
   fn delete_user(id: u32) -> &'static str {
       "User deleted"
   }

   #[launch]
   fn rocket() -> _ {
       rocket::build()
           .mount("/", routes![index, get_user, create_user, update_user, delete_user])
   }
   ```

2. **Запустите сервер:**

   ```bash
   cargo run
   ```

3. **Проверьте маршруты:**

   - **GET /users/1**

     ```bash
     curl http://127.0.0.1:8000/users/1
     ```

     **Ответ:**

     ```json
     {
       "id": 1,
       "name": "John Doe"
     }
     ```

   - **POST /users**

     ```bash
     curl -X POST http://127.0.0.1:8000/users \
     -H "Content-Type: application/json" \
     -d '{"id":2,"name":"Jane Smith"}'
     ```

     **Ответ:**

     ```json
     {
       "id": 2,
       "name": "Jane Smith"
     }
     ```

### Axum

Добавим дополнительные маршруты для обработки различных HTTP-методов.

1. **Редактируйте `src/main.rs`:**

   ```rust
   use axum::{
       routing::{get, post, put, delete},
       Router,
       Json,
       extract::Path,
   };
   use serde::{Deserialize, Serialize};
   use std::net::SocketAddr;

   #[derive(Serialize, Deserialize)]
   struct User {
       id: u32,
       name: String,
   }

   async fn index() -> &'static str {
       "Hello, Axum!"
   }

   async fn get_user(Path(id): Path<u32>) -> Json<User> {
       Json(User {
           id,
           name: "John Doe".to_string(),
       })
   }

   async fn create_user(Json(user): Json<User>) -> Json<User> {
       Json(user)
   }

   async fn update_user(Path(id): Path<u32>, Json(user): Json<User>) -> Json<User> {
       Json(User {
           id,
           name: user.name,
       })
   }

   async fn delete_user(Path(_id): Path<u32>) -> &'static str {
       "User deleted"
   }

   #[tokio::main]
   async fn main() {
       // Создаем маршруты
       let app = Router::new()
           .route("/", get(index))
           .route("/users/:id", get(get_user))
           .route("/users", post(create_user))
           .route("/users/:id", put(update_user))
           .route("/users/:id", delete(delete_user));

       // Определяем адрес сервера
       let addr = SocketAddr::from(([127, 0, 0, 1], 8080));
       println!("Server running at http://{}", addr);

       // Запускаем сервер
       axum::Server::bind(&addr)
           .serve(app.into_make_service())
           .await
           .unwrap();
   }
   ```

2. **Запустите сервер:**

   ```bash
   cargo run
   ```

3. **Проверьте маршруты:**

   - **GET /users/1**

     ```bash
     curl http://127.0.0.1:8080/users/1
     ```

     **Ответ:**

     ```json
     {
       "id": 1,
       "name": "John Doe"
     }
     ```

   - **POST /users**

     ```bash
     curl -X POST http://127.0.0.1:8080/users \
     -H "Content-Type: application/json" \
     -d '{"id":2,"name":"Jane Smith"}'
     ```

     **Ответ:**

     ```json
     {
       "id": 2,
       "name": "Jane Smith"
     }
     ```

---

## Заключение

- **Actix-web** — идеален для высокопроизводительных приложений с гибкой маршрутизацией.
- **Rocket** — удобен благодаря своей типобезопасности и простоте использования.
- **Axum** — отлично подходит для современных асинхронных приложений и интеграции с экосистемой Tokio.


## Дополнительные ресурсы

- [Actix-web документация](https://actix.rs/docs/)
- [Rocket документация](https://rocket.rs/v0.5-rc/guide/)
- [Axum документация](https://docs.rs/axum/latest/axum/)
- [The Rust Programming Language](https://doc.rust-lang.org/book/)
- [Serde — сериализация и десериализация данных](https://serde.rs/)
