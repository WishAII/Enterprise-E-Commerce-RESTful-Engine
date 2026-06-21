# 🛒 Enterprise E-Commerce RESTful Engine

A production-ready, modular e-commerce REST API backend built with **Java 25** and **Spring Boot 3.5**, featuring JWT authentication, Stripe payment integration, Flyway database migrations, and OpenAPI documentation.

---

## 🧰 Tech Stack

![Java](https://img.shields.io/badge/Java_25-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot_3.5-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white)
![Spring Security](https://img.shields.io/badge/Spring_Security_6-6DB33F?style=for-the-badge&logo=spring-security&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Hibernate](https://img.shields.io/badge/Hibernate_6-59666C?style=for-the-badge&logo=hibernate&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-626CD9?style=for-the-badge&logo=stripe&logoColor=white)
![Flyway](https://img.shields.io/badge/Flyway-CC0200?style=for-the-badge&logo=flyway&logoColor=white)
![Swagger](https://img.shields.io/badge/OpenAPI_3-85EA2D?style=for-the-badge&logo=swagger&logoColor=black)
![Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apache-maven&logoColor=white)

---

## ✨ Features

- **JWT Authentication** — Stateless token-based auth with role-based access control (Admin / User)
- **Modular Security** — Each domain module registers its own security rules via a `SecurityRules` interface
- **Stripe Payments** — Checkout session creation and cryptographic webhook signature verification
- **Transactional Integrity** — Orphaned orders automatically rolled back on payment failure
- **Flyway Migrations** — 5 versioned SQL migration scripts (V1–V5) with dev/prod profile separation
- **MapStruct DTOs** — Compile-time, reflection-free DTO ↔ entity mapping
- **Domain Encapsulation** — Business logic lives in entities, not services (e.g. `Cart.addItem()`)
- **Global Exception Handling** — Structured JSON errors for validation failures and malformed bodies
- **OpenAPI 3 / Swagger UI** — Interactive API docs with JWT bearer auth integration

---

## 🏗️ Architecture

The project follows **Package-by-Feature (Domain-Driven Design)** — each module is fully self-contained:

```
com.ecom.store
├── auth/          # JWT login, token generation & validation
├── users/         # Registration, profiles, addresses, password management
├── products/      # Product catalog and category management
├── carts/         # Cart CRUD with domain-level item logic
├── orders/        # Order creation from cart
├── payments/      # Stripe checkout + webhook handling (PaymentGateway interface)
├── admin/         # Admin-restricted endpoints
└── common/        # Global exception handler, logging filter, security base
```

### Security Architecture

Each module provides its own `SecurityRules` implementation. All implementations are auto-injected as a `List<SecurityRules>` into `SecurityFilterChain` — no centralized route config to maintain:

```java
// Example: CartSecurityRules.java
@Component
public class CartSecurityRules implements SecurityRules {
    @Override
    public void configure(AuthorizeHttpRequestsConfigurer<?>.AuthorizationManagerRequestMatcherRegistry c) {
        c.requestMatchers("/api/carts/**").authenticated();
    }
}
```

### Payment Architecture

The `PaymentGateway` interface decouples business logic from the payment provider. Swapping Stripe for Razorpay/PayPal requires only a new implementation — no service changes:

```java
public interface PaymentGateway {
    CheckoutSession createCheckoutSession(Order order);
    Optional<PaymentResult> parseWebhookRequest(WebhookRequest request);
}
```

---

## 🗄️ Database Schema

Managed via **Flyway** versioned migrations:

| Migration | Description |
|-----------|-------------|
| `V1__initial_migration.sql` | Users, profiles (1:1), addresses (1:N), products, categories, wishlist (M:N) |
| `V2__create_cart_tables.sql` | Cart and cart_items tables |
| `V3__add_role_to_users.sql` | Role column (USER / ADMIN) |
| `V4__add_order_tables.sql` | Orders and order_items tables |
| `V5__populate_database.sql` | Seed data |

**Key relationships:**
- `users` ↔ `profiles` — one-to-one (profile shares user PK)
- `users` → `addresses` — one-to-many
- `products` → `categories` — many-to-one
- `users` ↔ `products` (wishlist) — many-to-many with cascade delete

---

## 🚀 Getting Started

### Prerequisites

- Java 25
- Maven 3.9+
- MySQL 8+

### Setup

**1. Clone the repository**
```bash
git clone https://github.com/WishAII/ecommerce-backend.git
cd ecommerce-backend
```

**2. Create a `.env` file** in the project root (see [Environment Variables](#environment-variables)):
```bash
cp .env.example .env
```

**3. Create the database**

Flyway will auto-create and migrate the schema on startup. Just ensure your MySQL user has `CREATE DATABASE` permission, or create it manually:
```sql
CREATE DATABASE store_api;
```

**4. Run the application**
```bash
./mvnw spring-boot:run
```

The API will start at `http://localhost:8080`.

**5. Open Swagger UI**

Navigate to: `http://localhost:8080/swagger-ui.html`

Click **Authorize** → paste your JWT token (obtained from `POST /api/auth/login`) to test protected endpoints.

---

## 🔐 Environment Variables

Create a `.env` file in the project root with the following keys:

| Variable | Description |
|----------|-------------|
| `DB_URL` | JDBC connection string, e.g. `jdbc:mysql://localhost:3306/store_api` |
| `DB_USERNAME` | MySQL username |
| `DB_PASSWORD` | MySQL password |
| `JWT_SECRET` | Base64-encoded secret key for JWT signing |
| `JWT_EXPIRATION` | Token expiry in milliseconds, e.g. `86400000` (24h) |
| `STRIPE_SECRET_KEY` | Stripe secret key (`sk_test_...`) |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook signing secret (`whsec_...`) |

> ⚠️ Never commit your `.env` file. It is already in `.gitignore`.

---

## 📡 API Endpoints

| Module | Method | Endpoint | Auth |
|--------|--------|----------|------|
| Auth | `POST` | `/api/auth/register` | Public |
| Auth | `POST` | `/api/auth/login` | Public |
| Users | `GET` | `/api/users/{id}` | User |
| Users | `PUT` | `/api/users/{id}` | User |
| Users | `PATCH` | `/api/users/{id}/password` | User |
| Products | `GET` | `/api/products` | Public |
| Products | `POST` | `/api/products` | Admin |
| Cart | `POST` | `/api/carts` | User |
| Cart | `POST` | `/api/carts/{id}/items` | User |
| Cart | `PUT` | `/api/carts/{id}/items/{productId}` | User |
| Cart | `DELETE` | `/api/carts/{id}/items/{productId}` | User |
| Checkout | `POST` | `/api/checkout` | User |
| Checkout | `POST` | `/api/checkout/webhook` | Stripe |
| Orders | `GET` | `/api/orders/{id}` | User |
| Admin | `GET` | `/api/admin/users` | Admin |

> Full interactive documentation available at `/swagger-ui.html`

---

## 🧪 Running Tests

```bash
./mvnw test
```

---

## 📁 Project Structure

```
src/main/java/com/ecom/store/
├── StoreApplication.java
├── admin/
│   ├── AdminController.java
│   └── AdminSecurityRules.java
├── auth/
│   ├── AuthController.java
│   ├── AuthService.java
│   ├── JwtAuthenticationFilter.java
│   ├── JwtService.java
│   └── SecurityConfig.java
├── carts/
│   ├── Cart.java              # Domain logic: addItem, removeItem, clear
│   ├── CartController.java
│   ├── CartService.java
│   └── CartMapper.java
├── orders/
│   ├── Order.java
│   ├── OrderController.java
│   └── OrderService.java
├── payments/
│   ├── PaymentGateway.java        # Interface (DIP)
│   ├── StripePaymentGateway.java  # Stripe implementation
│   └── CheckoutService.java
├── products/
├── users/
└── common/
    ├── GlobalExceptionHandler.java
    └── SecurityRules.java         # Base interface for module security
```

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
