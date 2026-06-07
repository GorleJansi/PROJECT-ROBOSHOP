# Roboshop 3 Tier Architecture

Roboshop is a microservices-based e-commerce application designed around a 3 tier architecture. The project separates the user-facing frontend, backend business services, and supporting database/message services.

## Architecture Overview

The application is organized into three main tiers:

1. **Frontend Tier**
   - Serves the web application to users.
   - Sends API requests to backend services.

2. **Backend Tier**
   - Contains the application microservices.
   - Handles product catalog, users, cart, shipping, payments, and dispatch workflows.

3. **Database and Messaging Tier**
   - Stores application data.
   - Provides caching and asynchronous message processing.

## Components

| Tier | Component | Purpose |
| --- | --- | --- |
| Frontend | Frontend | Web UI and API gateway for user requests |
| Backend | Catalogue | Product catalogue service |
| Backend | User | User registration, login, and profile service |
| Backend | Cart | Shopping cart service |
| Backend | Shipping | Shipping calculation and order shipment service |
| Backend | Payment | Payment processing service |
| Backend | Dispatch | Order dispatch service |
| Database | MongoDB | Stores catalogue and user data |
| Database | Redis | Caches cart/session related data |
| Database | MySQL | Stores shipping data |
| Messaging | RabbitMQ | Message broker for payment and dispatch workflows |

## Service Flow

- Users access the application through the frontend.
- The frontend sends API calls to backend services.
- Backend services communicate with the required databases.
- Payment events are sent through RabbitMQ.
- Dispatch consumes messages from RabbitMQ and processes order dispatch.

## Technology Stack

- **Frontend:** Web application service
- **Backend:** Microservices architecture
- **Databases:** MongoDB, Redis, MySQL
- **Messaging:** RabbitMQ
- **Cloud Platform:** AWS

## Deployment Notes

This project is intended for DevOps practice around deploying and managing a full 3 tier Roboshop application. Typical deployment work includes:

- Provisioning infrastructure
- Configuring application services
- Managing databases and message queues
- Setting service dependencies
- Validating end-to-end application flow

