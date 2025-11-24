# E-commerce-quick-bitefood-
QuickBite is a food delivery app that lets users browse restaurants, explore menus, add items to cart, and place orders with live tracking. Built with clean, scalable architecture, it offers a smooth and responsive experience across devices.
# 🍔 Quickbite – Food Delivery Backend (Spring Boot)

Quickbite is a **RESTful backend API** for a simple food-ordering platform.  
It models **users, restaurants, menus, carts, orders, and coupons**, following a clean, layered architecture with Spring Boot and Spring Data JPA.

---

## 📚 Table of Contents

1. [Project Overview](#-project-overview)
2. [Main Features](#-main-features)
3. [Tech Stack](#-tech-stack)
4. [Architecture & Package Structure](#-architecture--package-structure)
5. [Domain Model (Entities)](#-domain-model-entities)
6. [DTOs & Mappers](#-dtos--mappers)
7. [Repositories Layer](#-repositories-layer)
8. [Service Layer & Business Logic](#-service-layer--business-logic)
9. [REST API Layer (Resources/Controllers)](#-rest-api-layer-resourcescontrollers)
10. [Exception Handling](#-exception-handling)
11. [Database Configuration & Profiles](#-database-configuration--profiles)
12. [Seed/Test Data (TestConfig)](#-seedtest-data-testconfig)
13. [Running the Project](#-running-the-project)
14. [API Endpoint Overview](#-api-endpoint-overview)
15. [Future Improvements](#-future-improvements)
16. [License](#-license)

---

## 🧾 Project Overview

**Quickbite** is a **Java Spring Boot** application that exposes REST APIs for a food-delivery system.  
It demonstrates:

- Proper **layered architecture** (Controller → Service → Repository → Entity).
- Use of **DTOs** and **mappers** to separate persistence models from API models.
- **Spring Data JPA** for database access.
- Basic **error handling** using custom exceptions and a global exception handler.
- Use of **H2 in-memory database** for testing / demo.

The project is designed as a **backend only** service, which can be consumed by any frontend (web, mobile, etc.).

---

## ✨ Main Features

- 🔐 **User Management**
  - Create users.
  - Fetch user details.
  - List a user’s orders.
- 🍽️ **Restaurant & Menu Management**
  - Create / update / delete restaurants.
  - List restaurants.
  - View products (menu items) for a restaurant.
- 🛒 **Shopping Cart**
  - Add items to cart for a user.
  - View current cart (items, quantities, total).
  - Remove items or clear cart.
  - Checkout cart to create an order.
- 📦 **Orders**
  - Place new order from cart.
  - List all orders or single order by ID.
  - Update order status.
- 🎟️ **Coupons**
  - Create / update / delete coupons.
  - Apply coupons to orders/carts (business logic in services).
- ⚠️ **Robust Error Handling**
  - Standard JSON error structure.
  - 404 for not found, 400/409 for validation or database issues.

---

## 🛠 Tech Stack

- **Language**: Java
- **Framework**: Spring Boot `3.5.4`
- **Build tool**: Maven
- **Persistence**: Spring Data JPA
- **Databases**:
  - In-memory **H2** (test/demo profile)
  - Support for **PostgreSQL** (runtime dependency in `pom.xml`)
- **Testing**: Spring Boot Starter Test
- **Dev tools**: Spring Boot DevTools (optional runtime dependency)

Dependencies (from `pom.xml`):

- `spring-boot-starter-data-jpa`
- `spring-boot-starter-web`
- `spring-boot-starter-test`
- `com.h2database:h2`
- `org.postgresql:postgresql`
- `spring-boot-devtools`

---

## 🧱 Architecture & Package Structure

**Base package**: `com.mathdev.quickbite`

Key sub-packages:

- `config` – configuration and test data seeding.
- `dto` – Data Transfer Objects (request/response models).
- `entities` – JPA entities (domain model).
- `entities.auth` – authentication-related entities (e.g., `AppUser`).
- `entities.enums` – enums such as `OrderStatus`, `Role`.
- `entities.pk` – composite primary key classes for join entities.
- `mapper` – entity–DTO mapping logic.
- `repositories` – Spring Data JPA repository interfaces.
- `resource` – REST controllers (exposed API endpoints).
- `resource.exceptions` – global exception handler and error response model.
- `services` – business logic.
- `services.exceptions` – custom service-layer exceptions.

This layered approach provides:

- **Separation of concerns** (each layer has one responsibility).
- **Maintainability** (easy to change one layer without breaking others).
- **Testability** (services can be unit tested without controllers).

---

## 🧩 Domain Model (Entities)

Located in: `src/main/java/com/mathdev/quickbite/entities`

Main entities:

- `User`
  - Represents a customer.
  - Fields: id, name, email, address, etc.
  - Relations: has many `Order`, has one `Cart`.
  - Uses `Role` enum for authorization roles (via `entities.enums.Role`).
- `Restaurant`
  - Represents a restaurant in the platform.
  - Fields: id, name, description, address, etc.
  - Relations: has many `Product` (menu items).
- `Product`
  - A menu item offered by a restaurant.
  - Fields: id, name, description, price, etc.
  - Relations: belongs to a `Restaurant`.
- `Cart`
  - Temporary shopping cart for a `User`.
  - Relations: has many `CartItem`.
- `CartItem`
  - Join entity between `Cart` and `Product`.
  - Uses composite key `CartItemPK` (in `entities.pk`).
  - Holds quantity and price information.
- `Order`
  - Represents a confirmed order.
  - Fields: id, moment (`Instant`), status (`OrderStatus`), total, etc.
  - Relations: belongs to a `User`, has many `OrderItem`.
- `OrderItem`
  - Join entity between `Order` and `Product`.
  - Uses composite key `OrderItemPK`.
  - Stores quantity and price at the moment of order.
- `Coupon`
  - Fields: id, code, discount (percentage or amount), active, expiry (`Instant`).
  - Used to apply discounts to orders/cart.
- `auth.AppUser`
  - Wrapper for user auth details separate from business `User`.
- Enums in `entities.enums`:
  - `OrderStatus` – e.g. PENDING, PAID, CANCELED, DELIVERED, etc.
  - `Role` – user roles (e.g., ADMIN, CUSTOMER).

Each entity uses:

- `@Entity`, `@Id`, `@GeneratedValue` (JPA annotations)
- Relationship annotations such as `@OneToMany`, `@ManyToOne`, etc.
- `Serializable` implementation and `equals`/`hashCode` for identity.

---

## 📦 DTOs & Mappers

### DTOs (`dto` package)

DTOs define what is **exposed to the API clients**, separate from the database entities.

Examples:

- `UserDTO`, `UserInsertDTO`
- `RestaurantDTO`
- `ProductDTO`
- `CartDTO`, `CartItemDTO`, `AddToCartDTO`
- `OrderDTO`, `OrderItemDTO`
- `CouponDTO`

Purposes:

- Hide internal fields (like passwords, internal IDs).
- Provide exactly the data needed by the frontend.
- Support validation and different shapes for input vs output (e.g. `UserInsertDTO` vs `UserDTO`).

### Mappers (`mapper` package)

Mappers convert between **Entity ↔ DTO**.

Examples:

- `UserMapper`
- `RestaurantMapper`
- `ProductMapper`
- `OrderMapper`
- `CouponMapper`

Typical methods:

- `toDTO(Entity entity)`: converts JPA entity to DTO.
- `toEntity(DTO dto)`: converts DTO to entity.
- `updateEntity(DTO dto, Entity entity)`: updates an existing entity from DTO.

This removes mapping logic from controllers/services and keeps the code cleaner.

---

## 🗄 Repositories Layer

Located in: `repositories` package.

Examples:

- `UserRepository`
- `RestaurantRepository`
- `ProductRepository`
- `OrderRepository`
- `CartRepository`
- `CartItemRepository`
- `CouponRepository`

All extend `JpaRepository<ENTITY, ID>` which provides:

- Basic CRUD methods:
  - `findAll()`, `findById()`, `save()`, `deleteById()`, etc.
- Optional **custom queries**, using `@Query` and `@Modifying` in some repos (e.g. for cart behavior).

The repositories isolate **data access** from the rest of the application.

---

## 🧠 Service Layer & Business Logic

Located in: `services` package.

Services use repositories to implement **application use-cases**:

- `UserService`
  - Create, read, update, delete users.
  - List orders for a user.
- `RestaurantService`
  - Manage restaurants and their products.
- `ProductService`
  - Manage products (menu items).
- `CartService`
  - Add products to cart (`AddToCartDTO`).
  - List current cart (`CartDTO`).
  - Remove items or clear the cart.
  - Convert cart into an order (checkout).
- `OrderService`
  - Create orders from cart.
  - Fetch orders, update status (`OrderStatus`).
- `CouponService`
  - Create / update / delete coupons.
  - Validate coupons (active/expired, etc.).

Services also:

- Throw custom exceptions like `ResourceNotFoundException` and `DatabaseException` (in `services.exceptions`).
- Encapsulate rules like:
  - Calculating totals.
  - Handling coupon discounts.
  - Ensuring entities exist before operations.

---

## 🌐 REST API Layer (Resources/Controllers)

Located in: `resource` package.

Each `*Resource` class is a **REST controller**:

- Annotated with `@RestController`.
- Base path via `@RequestMapping`.
- HTTP methods via `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`.
- Returns `ResponseEntity<DTO>` objects.

### UserResource

- Base path: `/users`
- Endpoints:
  - `GET /users` – list all users.
  - `GET /users/{id}` – get user by ID.
  - `GET /users/{id}/orders` – list orders for given user.
  - `POST /users` – create a new user.
  - `PUT /users/{id}` – update user.
  - `DELETE /users/{id}` – delete user.

### RestaurantResource

- Base path: `/restaurants`
- Endpoints:
  - `GET /restaurants` – list restaurants.
  - `GET /restaurants/{id}` – restaurant by ID.
  - `GET /restaurants/{id}/products` – list products for a restaurant.
  - `POST /restaurants` – create restaurant.
  - `PUT /restaurants/{id}` – update restaurant.
  - `DELETE /restaurants/{id}` – delete restaurant.

### Product APIs

Products are typically accessed via restaurant endpoints or dedicated Resource (depending on implementation), but core logic is under `ProductService`.

### CartResource

- Base path: `/cart`
- Endpoints:
  - `GET /cart` – view current cart details (e.g., for a user – depends on implementation).
  - `POST /cart/{userId}/add` – add product(s) to a user’s cart using `AddToCartDTO`.
  - `POST /cart/{userId}/checkout` – checkout cart and create order.
  - `DELETE /cart/remove/{productId}` – remove single product from cart.
  - `DELETE /cart/clear` – clear the entire cart.

### OrdersResource

- Base path: `/orders`
- Endpoints:
  - `GET /orders` – list all orders.
  - `GET /orders/{id}` – get order by ID.
  - `POST /orders` – create order (usually from cart).
  - `PUT /orders/{id}` – update order status or details.

### CouponsResource

- Base path: `/coupons`
- Endpoints:
  - `GET /coupons` – list all coupons.
  - `GET /coupons/{id}` – coupon details.
  - `POST /coupons` – create coupon.
  - `PUT /coupons/{id}` – update coupon.
  - `DELETE /coupons/{id}` – delete coupon.

These controllers:

- Receive DTOs from the client.
- Call corresponding service methods.
- Map entities back to DTOs.
- Build correct HTTP responses (201 for created, 200 for OK, 204 for delete, etc.).

---

## 🚨 Exception Handling

Located in: `resource.exceptions` and `services.exceptions`.

- `ResourceNotFoundException`
  - Thrown when an entity is not found (e.g., user ID does not exist).
- `DatabaseException`
  - Thrown on database constraint violations or similar errors.

`ResourceExceptionHandler` (in `resource.exceptions`):

- Annotated with `@ControllerAdvice`.
- Catches exceptions globally.
- Converts them to a standard JSON response using `StandardError` class, providing:
  - timestamp
  - HTTP status code
  - error message
  - path (endpoint)

This ensures consistent and user-friendly error responses.

---

## 🗃 Database Configuration & Profiles

`src/main/resources/application.properties`:

```properties
spring.application.name=Quickbite
spring.profiles.active=test
spring.jpa.open-in-view=true

