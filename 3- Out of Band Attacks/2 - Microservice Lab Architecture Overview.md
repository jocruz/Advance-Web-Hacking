# GraphQL Microservice Lab Architecture Overview

## High-Level Summary

This document provides a breakdown of the lab environment architecture to support vulnerability analysis and exploitation in a microservice-based application using **GraphQL**. The application consists of a **Moleculer-based microservice backend**, **customer and admin frontends**, and a **centralized database service**. Understanding this structure is crucial for determining **attack surfaces**, identifying which components are externally exposed, and how business logic is distributed across the application layers.

---

## 🧱 Application Structure Overview

The application is split into **three main layers**:

### 1. **Core Backend Services (Microservices)**

> [!info]
> The backend uses **Moleculer.js**, a Node.js framework for building microservices.  
> Services are defined in `services/` and bootstrapped in `app.js`.

#### Microservices include:
- **orders.service**: Handles order creation and management.
- **invoice.service**: Handles invoice generation and payment tracking.
- **stock.service**: Manages inventory.
- **database.service**: Centralized DB logic.

> [!warning]
> Unlike typical microservice patterns where each service has its own datastore, this architecture uses a **centralized `database.service`**, which may introduce a single point of failure or choke point for exploitation.

---

### 2. **Server-Side Rendering (SSR) Applications**

> [!note]
> The application does **not expose the microservices directly**.  
> Instead, two SSR applications act as intermediaries:  

- **Customer Frontend** (`port 3001`)
- **Admin Frontend** (`port 3002`)

These SSR apps interface with the microservices to:
- Render HTML views
- Generate PDFs
- Proxy GraphQL/API requests to backend services

> [!info]
> This design **hides backend microservices from direct external access**, increasing the need for client-side attack vectors (e.g., SSRF via SSR app or injection through proxies).

---

### 3. **Frontend Application Roles**

#### Customer Frontend (Port `3001`):
- Exposed to the public internet
- Handles:
  - Product browsing
  - Order placement
  - Invoice viewing
  - PDF generation

#### Admin Frontend (Port `3002`):
- Intended for internal team use
- Provides:
  - Dashboard
  - Stock/order/invoice management
  - Filtering and business operations

> [!tip]
> Test each interface as a separate surface.  
> Functionality exposed in the admin app might rely on the same underlying services, but with elevated privilege paths.

---

## 🔄 Business Logic & Workflow

The lab is intentionally structured to **mirror a realistic business environment**:

- Two user roles: customer and admin
- Business operations (e.g., ordering, invoicing) are distributed across both frontends and supported by shared backend services
- Each service is modular and reflects real business logic, encouraging testing of **authorization, trust boundaries, and data flow**

---

## ⚙️ Application Deployment & Testing Notes

> [!info]
> Use `npm run start:all` to boot all services and apps simultaneously.

### Startup Behavior:
- Each application logs its output with a unique prefix (`[0]`, `[1]`, `[2]`) to differentiate between:
  - Core services (port `3000`)
  - Customer app (port `3001`)
  - Admin app (port `3002`)

### Data Initialization:
- The `database.service` includes a **startup script** that:
  - **Drops all data**
  - **Re-inserts test fixtures** each time the app is restarted

> [!warning]
> Any custom data input will be **lost on restart or crash**. Take snapshots if needed.

---

## 🔍 Exploration Tips for Pentesters

> [!tip]
> Treat the **customer app** as your initial external entry point.  
> From there, pivot internally through misconfigurations, broken access control, or exposed admin functions.

### Recommendations:
- Explore endpoints via `localhost:3001` and `localhost:3002`
- Use **Burp Suite** or a proxy to observe how SSR apps communicate with the microservices
- Analyze API calls for:
  - Hidden GraphQL endpoints
  - Insecure business logic
  - Privilege escalation paths

---

## 🧰 Tools Section

### 🔧 Moleculer.js
- **What**: A fast Node.js microservices framework.
- **Purpose**: Allows decoupled services to be built and managed individually.
- **Usage**: Services are declared inside `/services` and orchestrated via `app.js`.

---

### 🔧 Node.js + npm
- **Purpose**: Core runtime and package manager.
- **Command**:
```bash
npm run start:all
````

- Starts the entire stack: services, customer frontend, admin frontend.
    

---

### 🔧 Chrome (for visual testing)

- Access points:
    
    - `http://localhost:3001` → Customer Frontend
        
    - `http://localhost:3002` → Admin Frontend
        

Use this for:

- Browsing app behavior
    
- Identifying functionality differences
    
- Verifying exploit impact
    

---

## Final Notes

> [!important]  
> Understanding the **system architecture is foundational** before performing attacks.  
> Without knowledge of port usage, service flow, and data resets, you'll lose time and potentially misinterpret results.

Always begin testing by:

1. Mapping all services and ports
    
2. Exploring UI features per role (customer vs admin)
    
3. Observing backend request patterns
    
4. Saving any valuable payloads or findings before restart