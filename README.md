# NavSmart: Automated Transit & Route Management Ecosystem

NavSmart is an enterprise-grade Automated Bus Scheduling and Route Management System. It was originally developed as a **Finalist solution for the Smart India Hackathon (SIH)**, specifically targeting the Delhi Transport Corporation's (DTC) operational bottlenecks.

The system replaces manual, resource-intensive logistics with a highly automated micro-frontend architecture. It specializes in two core transit problems:
1. **Algorithmic Duty Scheduling:** Managing both *Linked* (fixed crew/bus) and *Unlinked* (dynamic handovers/rest periods) crew schedules.
2. **GIS Route Management:** Utilizing real-world map data to map existing networks, draw new routes, and highlight geographical overlaps to reduce congestion.

Powered by a centralized Node.js/Express API gateway and a Supabase PostgreSQL database, the ecosystem serves distinct Vue 3/Quasar frontends tailored for admins, drivers, conductors, and passengers.
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

This ecosystem is divided into specialized applications based on user roles and hardware requirements. 

| Portal / Service | Primary Role | Source Code | Live Deployment |
| :--- | :--- | :--- | :--- |
| **Admin Portal** | Command center for transit administrators to manage data, schedules, and monitor live GIS overlaps. | [📂 `/admin-portal`](https://github.com/RoniMaity/navsmart-admin-portal) | [🔗 Launch App](https://navsmart-admin-portal.vercel.app/) |
| **Passenger Portal** | Public web application for passengers to search routes, book tickets, and generate digital QR codes. | [📂 `/everything-portal`](https://github.com/RoniMaity/navsmart-everything-portal) | [🔗 Launch App](https://navsmart-everything-portal.vercel.app/#/) |
| **Driver Portal** | Mobile interface for drivers to initiate linked/unlinked laps and follow live map navigation via OSRM. | [📂 `/driver-portal`](https://github.com/RoniMaity/navsmart-driver-portal) | [🔗 Launch App](https://navsmart-driver-portal.vercel.app/) |
| **Conductor Portal** | Mobile tool utilizing device cameras to scan passenger QR codes and issue manual tickets dynamically. | [📂 `/conductor-portal`](https://github.com/RoniMaity/navsmart-conductor-portal) | [🔗 Launch App](https://navsmart-conductor-portal.vercel.app/) |
| **Bus Stop Signage** | Unattended digital signage auto-polling real-time ETA and status to waiting passengers. | [📂 `/bus-stop-portal`](https://github.com/RoniMaity/navsmart-bus-stop-portal) | [🔗 Launch App](https://navsmart-bus-stop-portal.vercel.app/) |
| **Backend API** | Secure Node.js middleware enforcing business logic and custom JWT authentication. | [📂 `/backend`](https://github.com/RoniMaity/navsmart-backend) | *Internal Service (Render)* |

## Core Data Flow Example (Starting a Trip)

1. **Definition:** An admin creates a recurring schedule in the `admin-portal`. This fires an Axios POST to `/api/schedules` on the backend, writing to Supabase.
2. **Expansion:** The driver opens the `driver-portal`. The frontend fetches the schedules, and a local Javascript algorithm expands that single schedule into multiple "virtual slots" (laps) based on interval timings.
3. **Execution:** The driver clicks "Start Trip" on Lap 1. The `driver-portal` sends a POST to `/api/trips` with the `in_progress` status, creating an active trip row in Supabase.
4. **Broadcast:**
* The `bus-stop-portal`, which auto-polls `/api/trips` every 30 seconds, detects the new `in_progress` trip and instantly displays the bus as arriving on the public screen.
* The `conductor-portal` fetches `/api/trips`, identifies the `in_progress` trip assigned to their `conductor_id`, and automatically unlocks the QR scanning and manual ticketing UI for that specific route.
