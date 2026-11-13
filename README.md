# Actus — Inventory Management System

**A modern, organization-centric inventory and point-of-sale management platform for small businesses.**

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](LICENSE)
[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)](https://github.com/Mohanraj-1107/actus-client)
[![TypeScript](https://img.shields.io/badge/TypeScript-6-3178C6?logo=typescript&logoColor=white)](https://github.com/Mohanraj-1107/actus-client)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.1.1-6DB33F?logo=springboot&logoColor=white)](https://github.com/Mohanraj-1107/actus-server)
[![Java](https://img.shields.io/badge/Java-17-ED8B00?logo=openjdk&logoColor=white)](https://github.com/Mohanraj-1107/actus-server)
[![MySQL](https://img.shields.io/badge/MySQL-Database-4479A1?logo=mysql&logoColor=white)](https://github.com/Mohanraj-1107/actus-server)

Actus helps small businesses replace spreadsheets and disconnected tools with a single system for managing products, stock, customers, suppliers, purchases, and sales. Every business operates inside its own **organization workspace**, keeping its data isolated and secure.

> **Project status:** Active development. Core modules are implemented end-to-end across the client and server repositories.

---

## Repository Navigation

This is the **parent repository** and project landing page. The application source lives in two independent repositories:

| Repository | Role | Technology | Link |
|------------|------|------------|------|
| **Client** | React single-page application and admin dashboard | React 19, TypeScript, Vite, Tailwind CSS | [actus-client](https://github.com/Mohanraj-1107/actus-client) |
| **Server** | REST API, business logic, authentication, and persistence | Java 17, Spring Boot, MySQL | [actus-server](https://github.com/Mohanraj-1107/actus-server) |

| Resource | Location |
|----------|----------|
| Client setup and commands | [Client README → Getting Started](https://github.com/Mohanraj-1107/actus-client#getting-started) |
| Client architecture and conventions | [Client Architecture.md](https://github.com/Mohanraj-1107/actus-client/blob/master/Architecture.md) |
| Server setup and configuration | [Server README → Quick Start](https://github.com/Mohanraj-1107/actus-server#quick-start) |
| Server API and domain documentation | [Server docs/docs.md](https://github.com/Mohanraj-1107/actus-server/blob/master/docs/docs.md) |

---

## Overview

### The problem

Small businesses often track inventory, purchasing, and sales across spreadsheets, paper records, and separate tools. This leads to inaccurate stock counts, missed low-stock items, duplicate customer or supplier records, and no clear view of revenue or spending.

### The solution

Actus provides a centralized, multi-tenant management system where each organization can:

- Maintain a catalog of products with pricing, tax, units, and minimum stock levels.
- Track stock movements and identify low-stock or out-of-stock items early.
- Manage customer and supplier records in one place.
- Record purchase orders and sales, with inventory updated automatically as orders are completed or confirmed.
- Monitor business performance through a consolidated dashboard.

### Key capabilities

- Organization registration, login, and session management
- Product catalog management
- Inventory tracking with low-stock alerts
- Customer and supplier management
- Purchase order lifecycle management
- Sales order lifecycle management
- Consolidated Business dashboard
- Organization-scoped data isolation across all modules

---

## Architecture Overview

Actus follows a **decoupled client–server architecture**. The React frontend communicates with the Spring Boot backend over a versioned REST API.

```text
┌──────────────────────────────────────────────┐
│              React Client (SPA)              │
│  React 19 · TypeScript · Vite · Tailwind CSS │
│                                              │
│  Pages → Feature Components → UI Components  │
│  TanStack Query · Axios · React Router       │
└───────────────────────┬──────────────────────┘
                        │  HTTP(S) / JSON
                        │  HttpOnly auth cookies
                        ▼
┌──────────────────────────────────────────────┐
│           Spring Boot REST API               │
│  JwtFilter → Controller → Service → Repo     │
│                                              │
│  Multi-tenant OrgContext · Bean Validation   │
│  JWT access tokens · Rotating refresh tokens │
└───────────────────────┬──────────────────────┘
                        │  Spring Data JPA
                        ▼
┌──────────────────────────────────────────────┐
│                    MySQL                     │
│         Flyway-managed schema                │
└──────────────────────────────────────────────┘
```

### How the pieces communicate

- The client calls the backend at `http://localhost:8080/erp/api` (configurable with `VITE_API_BASE`).
- API endpoints are versioned under `/v1`; the servlet context path adds the `/erp/api` prefix.
- Requests are sent as JSON with `withCredentials: true` so the browser includes authentication cookies.
- The backend authenticates requests through a JWT access token stored in an **HttpOnly cookie**.
- When an access token expires, the Axios interceptor calls `POST /v1/refresh` to rotate the refresh token and retries the original request.
- The `JwtFilter` validates the token, derives the organization identity, and exposes it through `OrgContext`.
- Controllers delegate to services, which enforce business rules and use Spring Data JPA repositories to read and write MySQL data.
- Every business entity is scoped to an organization, so queries never cross tenant boundaries.
- Database schema changes are managed with Flyway migrations.

### Application structure

| Layer | Client | Server |
|-------|--------|--------|
| Presentation | Pages, feature components, reusable UI components | REST controllers under `/v1` |
| Application logic | React hooks, TanStack Query, API modules | Services with validation and business rules |
| Integration | Axios instance with auth interceptor | `JwtFilter`, `OrgContext`, global exception handler |
| Data | Typed API responses | JPA repositories and entities |
| Storage | Browser state and cookies | MySQL with Flyway migrations |

---

## Key Features

| Module | What it does |
|--------|--------------|
| **Products** | Create and manage organization-scoped products with product code, name, unit, purchase and selling prices, tax rate, and minimum stock. Product codes and names are unique within an organization. |
| **Inventory** | Automatically tracks stock per product. Supports adding and removing stock, prevents negative quantities, and surfaces low-stock and out-of-stock items. |
| **Customers** | Manage customer records with contact details and addresses. Customer emails are unique within an organization. |
| **Suppliers** | Manage supplier records with contact details and addresses. Supplier emails are unique within an organization. |
| **Purchase Management** | Create draft purchase orders with multiple line items, taxes, and discounts. Completing a purchase increases inventory; cancelling a completed purchase reverses the stock movement. |
| **Sales Management** | Create draft sales with multiple line items, taxes, and discounts. Confirming a sale validates available stock and decreases inventory; cancelling a completed sale restores it. |
| **Dashboard** | Consolidated KPIs for products, customers, suppliers, low/out-of-stock items, completed sales revenue, and completed purchase spend, plus sales, purchase, and inventory breakdowns. |
| **Organization & Auth** | Organization signup, login, JWT-based sessions, rotating refresh tokens, logout, and organization-scoped access control. |


---

## Technology Stack

### Frontend — Client

| Category | Technology |
|----------|------------|
| Framework | React 19 |
| Language | TypeScript |
| Build tool | Vite 8 |
| Styling | Tailwind CSS v4 |
| Routing | React Router v7 |
| Server state | TanStack Query v5 |
| HTTP client | Axios |
| Charts | Recharts |
| Icons | Lucide React |
| Utilities | clsx, tailwind-merge |
| Linting | ESLint |
| CI | GitHub Actions (install, lint, build) |

### Backend — Server

| Category | Technology |
|----------|------------|
| Language | Java 17 |
| Framework | Spring Boot 4.1.1 |
| Web | Spring Web MVC |
| Persistence | Spring Data JPA, MySQL |
| Validation | Jakarta Bean Validation |
| Authentication | JWT (`jjwt` 0.11.5), HttpOnly cookies |
| Password hashing | BCrypt via Spring Security Crypto |
| Refresh tokens | Opaque tokens hashed with SHA-256, rotated on refresh |
| Migrations | Flyway |
| Boilerplate reduction | Lombok |
| Build tool | Maven (Maven Wrapper included) |

---

## Documentation and Setup

Each repository contains its own installation, configuration, and execution instructions. Start there for full details.

### Client

```bash
git clone https://github.com/Mohanraj-1107/actus-client.git
cd actus-client
npm install
npm run dev
```

The client runs at `http://localhost:5173`. Set `VITE_API_BASE` if the API is not available at `http://localhost:8080/erp/api`.

- [Client README](https://github.com/Mohanraj-1107/actus-client#readme)
- [Client setup and commands](https://github.com/Mohanraj-1107/actus-client#getting-started)
- [Client architecture](https://github.com/Mohanraj-1107/actus-client/blob/master/Architecture.md)

### Server

Prerequisites: **Java 17** and **MySQL** running on `localhost:3306`.

```bash
git clone https://github.com/Mohanraj-1107/actus-server.git
cd actus-server
./mvnw spring-boot:run
```

The API runs at `http://localhost:8080/erp/api`. Configure the database, JWT secret, and cookie settings through the environment variables documented in the server repository.

- [Server README](https://github.com/Mohanraj-1107/actus-server#readme)
- [Server quick start](https://github.com/Mohanraj-1107/actus-server#quick-start)
- [Server technical documentation](https://github.com/Mohanraj-1107/actus-server/blob/master/docs/docs.md)

### Running the full stack

1. Start MySQL and configure the server database connection.
2. Start the Spring Boot server on port `8080`.
3. Start the React client on port `5173`.
4. Open `http://localhost:5173`, create an organization, and sign in.

---

## Contributing

Contributions, bug reports, and suggestions are welcome. Please open an issue or pull request in the relevant repository:

- **Client changes:** [actus-client issues](https://github.com/Mohanraj-1107/actus-client/issues)
- **Server changes:** [actus-server issues](https://github.com/Mohanraj-1107/actus-server/issues)

---

## License

This project is licensed under the **GNU Affero General Public License v3.0**. See [LICENSE](LICENSE) for the full text.

---

**Author:** Mohanraj Subramani · **Repository:** [Mohanraj-1107/actus](https://github.com/Mohanraj-1107/actus)

