# NavSmart: Smart Public Transit Ecosystem

NavSmart is a comprehensive, multi-portal smart public transit management system. It coordinates real-time bus tracking, automated scheduling, mobile ticketing, and driver/conductor operations into a single unified platform.

The entire system is powered by a centralized Node.js/Express backend that acts as an API gateway over a Supabase PostgreSQL database, with distinct Vue 3/Quasar frontends tailored for different user roles (admins, drivers, conductors, and passengers).

## System Architecture

```mermaid
flowchart TD
    subgraph Frontends ["Vue 3 / Quasar Client Portals"]
        A[Admin Portal]
        D[Driver Portal]
        C[Conductor Portal]
        P[Passenger Portal]
        B[Bus Stop Signage]
    end
    
    BE["Node.js / Express Backend (API Gateway)"]
    DB[("Supabase (PostgreSQL)")]
    OSRM["OSRM API (Routing)"]

    A -->|"REST API + JWT"| BE
    D -->|"REST API + JWT"| BE
    C -->|"REST API + JWT"| BE
    P -->|"REST API + JWT"| BE
    B -->|"REST API (30s Polling)"| BE

    D -.->|"Coordinate Data"| OSRM

    BE <-->|"supabase-js (CRUD & Auth)"| DB

```

## The Micro-Repositories

This ecosystem is divided into specialized client applications. You can test the live deployments below.

| Portal | Primary Role | Live Link |
| :--- | :--- | :--- |
| **Admin Portal** | Command center for transit administrators to manage data and monitor live maps. | [navsmart-admin-portal.vercel.app](https://navsmart-admin-portal.vercel.app/) |
| **Customer Portal** | Passenger web app for journey planning and QR ticket generation. | [navsmart-everything-portal.vercel.app](https://navsmart-everything-portal.vercel.app/#/) |
| **Driver Portal** | App for drivers to start trips and view exact route polylines. | [navsmart-driver-portal.vercel.app](https://navsmart-driver-portal.vercel.app/) |
| **Conductor Portal** | App for conductors to issue tickets and scan passenger QR codes. | [navsmart-conductor-portal.vercel.app](https://navsmart-conductor-portal.vercel.app/) |
| **Bus Stop Signage** | Digital signage that auto-polls the API to display real-time ETA to passengers. | [navsmart-bus-stop-portal.vercel.app](https://navsmart-bus-stop-portal.vercel.app/) |
| **Backend API** | Secure Node.js middleware enforcing business logic and custom JWT authentication. | *Internal API Service (Render)* |

## Core Data Flow Example (Starting a Trip)

1. **Definition:** An admin creates a recurring schedule in the `admin-portal`. This fires an Axios POST to `/api/schedules` on the backend, writing to Supabase.
2. **Expansion:** The driver opens the `driver-portal`. The frontend fetches the schedules, and a local Javascript algorithm expands that single schedule into multiple "virtual slots" (laps) based on interval timings.
3. **Execution:** The driver clicks "Start Trip" on Lap 1. The `driver-portal` sends a POST to `/api/trips` with the `in_progress` status, creating an active trip row in Supabase.
4. **Broadcast:**
* The `bus-stop-portal`, which auto-polls `/api/trips` every 30 seconds, detects the new `in_progress` trip and instantly displays the bus as arriving on the public screen.
* The `conductor-portal` fetches `/api/trips`, identifies the `in_progress` trip assigned to their `conductor_id`, and automatically unlocks the QR scanning and manual ticketing UI for that specific route.
