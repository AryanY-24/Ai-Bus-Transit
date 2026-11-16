# INFORMATION.md — Comprehensive Project Notes & Architectural Guide

> **Project:** Real-Time Bus Tracking & Machine Learning Arrival Prediction Platform  
> **Geographic Corridor:** Belman ↔ Udupi (Karnataka, India)  
> **Target Audience:** Developers, Students, System Architects, and Technical Evaluators  
> **Documentation Style:** Detailed Notes, Architectural Insights, and Educational Explanations (No Raw Source Code Pasted)

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Project Goals and Requirements](#2-project-goals-and-requirements)
3. [Technology Stack](#3-technology-stack)
4. [Architecture Overview](#4-architecture-overview)
5. [Complete Directory and File Structure](#5-complete-directory-and-file-structure)
6. [Detailed Notes for Every Relevant File](#6-detailed-notes-for-every-relevant-file)
   - [Root Directory Files](#61-root-directory-files)
   - [Backend Application Core (`aaaa/backend/app/core/`)](#62-backend-application-core)
   - [Backend Database & Models (`aaaa/backend/app/db/` & `aaaa/backend/app/models/`)](#63-backend-database--models)
   - [Backend Routers (`aaaa/backend/app/routers/`)](#64-backend-routers)
   - [Backend Schemas (`aaaa/backend/app/schemas/`)](#65-backend-schemas)
   - [Backend Root Files & Scripts (`aaaa/backend/`)](#66-backend-root-files--scripts)
   - [Machine Learning Assets & Dataset (`aaaa/backend/models/` & `aaaa/backend/dataset/`)](#67-machine-learning-assets--dataset)
   - [Frontend Root & Configuration (`aaaa/frontend/`)](#68-frontend-root--configuration)
   - [Frontend Components (`aaaa/frontend/src/components/`)](#69-frontend-components)
   - [Frontend Contexts & Hooks (`aaaa/frontend/src/context/` & `aaaa/frontend/src/hooks/`)](#610-frontend-contexts--hooks)
   - [Frontend Pages (`aaaa/frontend/src/pages/`)](#611-frontend-pages)
   - [Frontend Services & Utilities (`aaaa/frontend/src/services/` & `aaaa/frontend/src/config.js`)](#612-frontend-services--utilities)
   - [Mobile Container Files (`aaaa/frontend/android/`)](#613-mobile-container-files)
7. [Folder-by-Folder Explanation](#7-folder-by-folder-explanation)
8. [Application Startup and Initialization](#8-application-startup-and-initialization)
9. [Complete End-to-End Project Workflow](#9-complete-end-to-end-project-workflow)
10. [File Dependencies and Module Relationships](#10-file-dependencies-and-module-relationships)
11. [Feature-by-Feature Analysis](#11-feature-by-feature-analysis)
12. [Frontend Workflow](#12-frontend-workflow)
13. [Backend Workflow](#13-backend-workflow)
14. [Database and Data Flow](#14-database-and-data-flow)
15. [APIs and External Integrations](#15-apis-and-external-integrations)
16. [Configuration and Environment Variables](#16-configuration-and-environment-variables)
17. [Authentication and Security](#17-authentication-and-security)
18. [Error Handling and Logging](#18-error-handling-and-logging)
19. [Testing and Quality Assurance](#19-testing-and-quality-assurance)
20. [Installation and Setup](#20-installation-and-setup)
21. [Running, Building, and Deployment](#21-running-building-and-deployment)
22. [Troubleshooting Guide](#22-troubleshooting-guide)
23. [Performance and Scalability](#23-performance-and-scalability)
24. [Limitations and Improvement Opportunities](#24-limitations-and-improvement-opportunities)
25. [Important Concepts and Study Notes](#25-important-concepts-and-study-notes)
26. [Frequently Asked Questions (FAQ)](#26-frequently-asked-questions-faq)
27. [Glossary of Technical Terms](#27-glossary-of-technical-terms)
28. [Verification Status and Documentation Gaps](#28-verification-status-and-documentation-gaps)

---

## 1. Project Overview

### What is this project?
The **Bus Tracking & ML Arrival Prediction Platform** is an end-to-end, full-stack transit management application designed to solve transit unpredictability along regional bus corridors. Specifically tailored for the **Belman ↔ Udupi** route in Karnataka, India, it integrates **real-time driver GPS tracking** with an **offline-trained machine learning intelligence engine** to compute estimated arrival times (ETAs), anticipated road delays, and bus passenger crowd levels.

### The Problem It Solves
1. **Unpredictable Bus Schedules:** Commuters in regional and suburban corridors frequently wait at rural and suburban bus stops without knowing when the next bus will arrive, whether it has already passed, or whether it is stuck in traffic.
2. **Lack of Live Information:** Unlike metropolitan areas with centralized Intelligent Transportation Systems (ITS) and GPS units hardwired into city fleets, regional private and public bus lines lack live digital tracking.
3. **Overcrowding & Commuter Discomfort:** Passengers cannot tell in advance whether the arriving bus will be jam-packed or relatively empty, making trip planning stressful.
4. **Flawed Pure-ML ETA Calculations:** Standard naive machine learning models that attempt to predict arrival times directly based on the passenger's current clock time fail because they confuse "when the passenger opened the app" with "when the bus was actually scheduled to run." This project implements a robust **two-stage architecture** that looks up the bus's genuine timetable arrival first and uses ML exclusively to predict traffic delay variations.

### Core System Structure
The project consists of three tightly coupled operational tiers:
- **FastAPI Python Backend:** Handles REST authentication, active trip lifecycles, SQLite persistence, and high-frequency real-time WebSocket communication. It also hosts the scikit-learn machine learning inference pipeline.
- **React 19 Frontend Web & Mobile App:** Built with Vite and Tailwind CSS, featuring interactive maps powered by Leaflet, animated live vehicle tracking markers, user role toggling, and prediction queries.
- **Capacitor Android Shell:** Converts the modern React web application into a native Android APK that taps directly into device GPS hardware through native Android location managers.

---

## 2. Project Goals and Requirements

### Functional Goals
- **Dual User Roles:** Clear division of responsibilities between **Drivers** (who initiate trips and broadcast live location pings) and **Passengers** (who view live bus markers on a map and query arrival predictions).
- **Sub-Second GPS Broadcasting:** Drivers stream their GPS latitude and longitude coordinates over a persistent WebSocket connection every 4 seconds.
- **Live Passenger Map Visualization:** Commuters observe buses moving smoothly across the map with auto-interpolated movement animations and relative time-ago indicators.
- **Two-Stage Machine Learning Inferences:**
  1. Determine the exact next scheduled bus from a fine-grained timetable lookup table derived from historical trip data.
  2. Feed normal arrival time and contextual features into a `RandomForestRegressor` to predict arrival delay in minutes.
  3. Feed the feature vector into a `GradientBoostingClassifier` to estimate passenger crowd density (`Low`, `Medium`, `High`).
- **Automatic Stale-Trip Pruning:** Automatically detect when a driver abandons a trip or loses connection for more than 5 minutes, marking the trip as `ended` so ghost buses do not linger on the map.

### Non-Functional Requirements
- **Low Latency:** WebSocket broadcast overhead kept under 50 milliseconds on local network connections.
- **Offline / Low-Resource Deployability:** Embedded SQLite database and pre-serialized Joblib ML models eliminate dependencies on external cloud clusters or heavy external database engines during development and local testing.
- **Cross-Platform Compatibility:** Runs on desktop web browsers, mobile responsive viewports, and native Android handsets.

---

## 3. Technology Stack

| Layer / Role | Technology / Library | Version | Purpose in this Project |
|---|---|---|---|
| **Backend Framework** | FastAPI | >= 0.100.0 | High-performance asynchronous REST API and WebSocket server. |
| **ASGI Web Server** | Uvicorn (standard) | >= 0.22.0 | Asynchronous Server Gateway Interface server running the FastAPI app. |
| **Database ORM** | SQLAlchemy | >= 2.0.0 | Object-Relational Mapper modeling users, trips, and live positions. |
| **Database Engine** | SQLite3 | 3.x (Embedded) | Local serverless SQL storage (`bus_tracker.db`). |
| **Authentication / JWT** | python-jose & passlib (bcrypt) | 3.3.0 & 1.7.4 | Password hashing and JSON Web Token generation/verification. |
| **Data Validation** | Pydantic & Pydantic-Settings | >= 2.0.0 | Request/response data validation, typed schemas, and `.env` parsing. |
| **Machine Learning** | Scikit-Learn | >= 1.2.0 | Random Forest Regressor (delays) and Gradient Boosting (crowd). |
| **Data Processing** | Pandas, NumPy, openpyxl | >= 2.0.0 | Reading Excel dataset, feature engineering, timetable construction. |
| **Model Serialization** | Joblib | >= 1.2.0 | Persisting and loading trained estimators and label encoders. |
| **Frontend Framework** | React | 19.2.8 | Declarative reactive UI components and state management. |
| **Build Tool & Dev Server** | Vite | 8.2.0 | Next-generation fast frontend bundling and Hot Module Replacement (HMR). |
| **CSS Styling Engine** | Tailwind CSS & @tailwindcss/vite | 4.3.3 | Utility-first CSS engine with modern CSS-native configuration. |
| **Mapping Engine** | Leaflet & React-Leaflet | 1.9.4 & 5.0.0 | OpenStreetMap tile rendering, custom SVG marker pins, and smooth map transitions. |
| **HTTP Client** | Axios | 1.19.0 | Promise-based REST client with automated auth token interceptors. |
| **Client Routing** | React Router DOM | 7.18.2 | Single Page Application (SPA) client-side URL routing and route guards. |
| **Mobile Runtime** | Capacitor Core & Android | 8.5.0 | Hybrid mobile container wrapping the web build into an Android Studio project. |
| **Native Geolocation** | @capacitor/geolocation | 8.2.2 | Native hardware GPS access on Android with runtime permission handling. |

---

## 4. Architecture Overview

### Architectural Pattern
The application follows a **Decoupled Client-Server Architecture** augmented by an **Event-Driven WebSocket Pub/Sub Layer** and a **Two-Stage Machine Learning Pipeline**.

```
+-----------------------------------------------------------------------------------+
|                                 CLIENT TIER                                       |
|                                                                                   |
|   +------------------------------------+    +---------------------------------+   |
|   |          Driver Client             |    |        Passenger Client         |   |
|   |  - HTML5 / Capacitor Geolocation   |    |  - Leaflet Map & Stop Overlays  |   |
|   |  - Driver Console State Machine    |    |  - Prediction Query Form        |   |
|   |  - 4s GPS Interval Ping Loop       |    |  - Smooth Marker Interpolation  |   |
|   +-----------------+------------------+    +----------------+----------------+   |
+---------------------|----------------------------------------|--------------------+
                      | HTTP REST (Start/End)                  | HTTP REST (/predict)
                      | WS (Location Ping Stream)              | WS (Live Map Stream)
                      v                                        v
+-----------------------------------------------------------------------------------+
|                              FASTAPI BACKEND TIER                                 |
|                                                                                   |
|   +--------------------+   +---------------------+   +------------------------+   |
|   |  /api/v1/auth      |   |  /api/v1/trips      |   |  /api/v1/predict       |   |
|   |  - Signup & Login  |   |  - Start / End Trip |   |  - Binary Search Table |   |
|   |  - JWT Issuance    |   |  - Query Active     |   |  - RF Delay Inference  |   |
|   |  - Role Validation |   |  - Garbage Clean    |   |  - GB Crowd Inference  |   |
|   +--------------------+   +----------+----------+   +------------------------+   |
|                                       |                                           |
|                  +--------------------+--------------------+                      |
|                  |     ConnectionManager (ws_manager.py)    |                      |
|                  |  - driver_sockets: trip_id -> WebSocket  |                      |
|                  |  - passenger_sockets: route -> Set[WS]   |                      |
|                  +--------------------+--------------------+                      |
|                                       |                                           |
+---------------------------------------|-------------------------------------------+
                                        |
                 +----------------------+----------------------+
                 |                                             |
                 v                                             v
+----------------------------------+       +----------------------------------------+
|          PERSISTENCE             |       |        ML ASSETS IN-MEMORY             |
|                                  |       |                                        |
|   SQLite (bus_tracker.db)        |       |   - delay_model.pkl (Random Forest)    |
|   - Table: users                 |       |   - crowd_model.pkl (Gradient Boost)   |
|   - Table: trips                 |       |   - encoders.pkl (Feature Encoders)    |
|   - Table: live_positions        |       |   - schedule_timetable.json (10k runs) |
+----------------------------------+       +----------------------------------------+
```

### Subsystem Boundaries
1. **Authentication Boundary:** Statistically stateless via Bearer JWT tokens. Passwords stored using one-way bcrypt hashing. Role claims (`driver` vs `passenger`) enforced at router level.
2. **Real-Time Pub/Sub Room Boundary:** Built around route keys (`belman_to_udupi` and `udupi_to_belman`). Drivers publish to their trip; the backend automatically fans out updates to all passengers subscribed to that specific route key.
3. **Machine Learning Serving Boundary:** ML models are loaded into memory once during application startup. Inferences do not spawn subprocesses or re-read disk files.

---

## 5. Complete Directory and File Structure

Below is the verified structural tree of the entire workspace, displaying every file and directory:

```
c:\Users\boazs\Downloads\aaaa
│
├── bus_tracker.db                                [Root SQLite DB generated if uvicorn runs at root]
├── LOCAL_SETUP.md                                [Windows localhost setup instructions for root]
├── start-dev.bat                                 [One-click batch launcher for backend + frontend]
├── start-dev.ps1                                 [One-click PowerShell launcher with port logging]
├── stop-dev.bat                                  [One-click batch script terminating ports 8000 & 5173]
├── stop-dev.ps1                                  [PowerShell script to kill lingering node/python PIDs]
│
└── aaaa\                                         [Primary project source wrapper]
    ├── LOCAL_SETUP.md                            [Setup guide referencing subfolder paths]
    ├── README.md                                 [Deployment, architecture, and Docker/Android manual]
    ├── start-dev.bat                             [Local launcher inside project wrapper]
    ├── start-dev.ps1                             [Local PowerShell launcher inside project wrapper]
    ├── stop-dev.bat                              [Local stop batch script inside project wrapper]
    ├── stop-dev.ps1                              [Local stop PowerShell script inside project wrapper]
    │
    ├── backend\                                  [FastAPI REST API & Machine Learning engine]
    │   ├── .dockerignore                         [Exclusions for Docker builds]
    │   ├── .env                                  [Active runtime environment variables]
    │   ├── .env.example                          [Template environment variable configuration]
    │   ├── bus_tracker.db                        [Populated SQLite database: users, trips, pings]
    │   ├── Dockerfile                            [Container recipe based on python:3.13-slim]
    │   ├── requirements.txt                      [Python package dependency manifest]
    │   ├── test_pipeline_validation.py           [40-point automated prediction validation suite]
    │   │
    │   ├── app\                                  [FastAPI application package]
    │   │   ├── main.py                           [Server entry point, CORS config, router mounting]
    │   │   │
    │   │   ├── core\                             [Core configuration and singleton services]
    │   │   │   ├── config.py                     [Pydantic BaseSettings: URLs, secrets, CORS]
    │   │   │   ├── ml_models.py                  [In-memory loader for serialized ML models & JSON]
    │   │   │   ├── security.py                   [JWT encoding/decoding, bcrypt, dependency guards]
    │   │   │   ├── stops.py                      [Static stop coordinates, orders, and route names]
    │   │   │   └── ws_manager.py                 [Singleton WebSocket connection registry]
    │   │   │
    │   │   ├── db\                               [Database infrastructure]
    │   │   │   └── database.py                   [SQLAlchemy engine, sessionmaker, get_db dependency]
    │   │   │
    │   │   ├── models\                           [SQLAlchemy ORM entity models]
    │   │   │   ├── __init__.py                   [Package exports for User, Trip, LivePosition]
    │   │   │   ├── trip.py                       [Trip and LivePosition table definitions]
    │   │   │   └── user.py                       [User table definition with role flag]
    │   │   │
    │   │   ├── routers\                          [REST and WebSocket API endpoints]
    │   │   │   ├── __init__.py                   [Aggregates and exposes auth, trips, predict routers]
    │   │   │   ├── auth.py                       [POST /signup, POST /login, GET /me endpoints]
    │   │   │   ├── predict.py                    [POST /predict ML engine, GET /stops endpoint]
    │   │   │   └── trips.py                      [Trip REST APIs & Driver/Passenger WebSocket routes]
    │   │   │
    │   │   └── schemas\                          [Pydantic request & response data models]
    │   │       ├── __init__.py                   [Package exports for user schemas]
    │   │       ├── predict.py                    [PredictRequest, PredictResponse, LiveBus models]
    │   │       ├── trip.py                       [TripStart, TripOut, ActiveTripOut, LocationPing]
    │   │       └── user.py                       [UserCreate, UserLogin, UserOut, Token schemas]
    │   │
    │   ├── dataset\                              [Raw historical transportation data]
    │   │   └── Master_Bus_Passenger_Flow.xlsx    [50,130-row master Excel dataset for the corridor]
    │   │
    │   ├── models\                               [Trained ML estimators and precomputed lookups]
    │   │   ├── bus_name_lookup.json              [Fallback bus lookup by route, stop, time slot]
    │   │   ├── crowd_model.pkl                   [Scikit-learn GradientBoostingClassifier binary]
    │   │   ├── delay_model.pkl                   [Scikit-learn RandomForestRegressor binary]
    │   │   ├── encoders.pkl                      [Label and category encoders dictionary]
    │   │   ├── schedule_lookup.json              [Median, mean, min, max scheduled arrival stats]
    │   │   └── schedule_timetable.json           [Fine-grained discrete bus timetable array]
    │   │
    │   └── scripts\                              [Offline data processing and training pipelines]
    │       └── train_models.py                   [End-to-end model training & timetable builder script]
    │
    └── frontend\                                 [React 19 + Vite + Tailwind CSS Single Page App]
        ├── .env                                  [Frontend runtime URLs: VITE_API_BASE_URL, etc.]
        ├── .env.example                          [Frontend template environment variables]
        ├── .env.production                       [Production/Android Wi-Fi host IP overrides]
        ├── .gitignore                            [Git ignore rules for web and mobile dependencies]
        ├── capacitor.config.json                 [Capacitor mobile container configuration file]
        ├── index.html                            [HTML single page container with React entry mount]
        ├── package.json                          [Node.js package manifest and development scripts]
        ├── package-lock.json                     [Locked exact dependency tree for npm]
        ├── tsconfig.json                         [TypeScript compiler configuration]
        ├── vite.config.js                        [Vite configuration with React & Tailwind plugins]
        │
        ├── public\                               [Static public assets served as-is]
        │   ├── favicon.svg                       [Browser tab transit bus favicon]
        │   └── icons.svg                         [Vite template SVG icons sprite]
        │
        ├── src\                                  [React application source code]
        │   ├── App.jsx                           [Root application component, routes, context trees]
        │   ├── config.js                         [Dynamic runtime IP and port resolution logic]
        │   ├── counter.ts                        [Legacy Vite boilerplate counter (unused in React)]
        │   ├── index.css                         [Tailwind CSS v4 import and custom UI animations]
        │   ├── main.jsx                          [React DOM root mounting file with StrictMode]
        │   ├── main.ts                           [Legacy Vite boilerplate main (unused in React)]
        │   ├── style.css                         [Legacy Vite boilerplate styling (unused in React)]
        │   │
        │   ├── assets\                           [Static media bundled with frontend]
        │   │   ├── hero.png                      [Vite boilerplate image asset]
        │   │   ├── typescript.svg                [TypeScript logo icon]
        │   │   └── vite.svg                      [Vite build tool logo]
        │   │
        │   ├── components\                       [Reusable UI elements]
        │   │   ├── Button.jsx                    [Themed button supporting 4 distinct variants]
        │   │   ├── Card.jsx                      [Glassmorphic panel container with header support]
        │   │   ├── LoadingSpinner.jsx            [Animated spinning indicator with customizable size]
        │   │   ├── Navbar.jsx                    [Responsive top navigation bar with role badges]
        │   │   ├── ProtectedRoute.jsx            [Route guard checking auth token and allowed role]
        │   │   └── SkeletonBlock.jsx             [Shimmering placeholders for loading states]
        │   │
        │   ├── context\                          [React global state providers]
        │   │   ├── AuthContext.jsx               [User auth, token storage, login/signup/logout]
        │   │   └── ToastContext.jsx              [Transient UI alert notifications with auto-timer]
        │   │
        │   ├── hooks\                            [Custom reusable React hooks]
        │   │   └── useAuth.js                    [Convenience consumer hook for AuthContext]
        │   │
        │   ├── pages\                            [Primary application views]
        │   │   ├── DriverPage.jsx                [Driver portal: start trip, stream GPS via WS]
        │   │   ├── LoginPage.jsx                 [Authentication screen with sign-in/sign-up tabs]
        │   │   └── PassengerPage.jsx             [Passenger portal: live Leaflet map & ML ETA tab]
        │   │
        │   └── services\                         [Network and HTTP abstraction services]
        │       └── api.js                        [Axios instance with token headers & 401 interceptor]
        │
        └── android\                              [Capacitor-generated native Android project]
            ├── build.gradle                      [Top-level Gradle build configuration]
            ├── capacitor.settings.gradle         [Capacitor-managed plugin inclusions]
            ├── gradle.properties                 [JVM and Android build flags]
            ├── gradlew / gradlew.bat             [Unix and Windows Gradle build wrapper binaries]
            ├── settings.gradle                   [Subproject definitions]
            ├── variables.gradle                  [Android SDK and support library version variables]
            └── app\                              [Android application module]
                ├── build.gradle                  [App module dependencies and SDK levels]
                ├── capacitor.build.gradle        [Capacitor dependency sync script]
                ├── proguard-rules.pro            [Code obfuscation and shrinking rules]
                └── src\main\
                    ├── AndroidManifest.xml       [App permissions: FINE_LOCATION, INTERNET]
                    └── java\com\akash\bustracker\
                        └── MainActivity.java     [Capacitor BridgeActivity entry point]
```

---

## 6. Detailed Notes for Every Relevant File

### 6.1 Root Directory Files

#### [start-dev.bat](file:///C:/Users/boazs/Downloads/aaaa/start-dev.bat) & [start-dev.ps1](file:///C:/Users/boazs/Downloads/aaaa/start-dev.ps1)
- **Role:** One-click development environment launchers for Windows users.
- **Why Needed:** Avoids requiring developers to open two separate terminal windows, manually navigate directories, activate Python virtual environments, and remember port parameters.
- **Responsibilities:**
  - Auto-detects whether the project structure has files directly in the root or wrapped inside the `aaaa/` subfolder.
  - Verifies that `.venv` exists; if missing, automatically runs `python -m venv` and installs `requirements.txt`.
  - Verifies that `frontend/node_modules` exists; if missing, invokes `npm.cmd install`.
  - Spawns two independent console windows: one running Uvicorn on `http://0.0.0.0:8000` and the other running Vite on `http://localhost:5173`.
- **Dependencies:** Windows Command Prompt / PowerShell, Python, Node.js (`npm.cmd`).
- **Risks & Edge Cases:** If ports 8000 or 5173 are already bound by lingering zombie processes from a previous crash, the startup commands fail with `Errno 10048`. Run `stop-dev.bat` to resolve this.

#### [stop-dev.bat](file:///C:/Users/boazs/Downloads/aaaa/stop-dev.bat) & [stop-dev.ps1](file:///C:/Users/boazs/Downloads/aaaa/stop-dev.ps1)
- **Role:** Process cleanup and port termination utilities.
- **Why Needed:** On Windows, closing a command prompt window running a Node or Uvicorn server can leave underlying background child processes running in detached states, locking ports 8000 and 5173.
- **Responsibilities:**
  - Queries `netstat -aon` or `Get-NetTCPConnection` for any process listening on ports 8000 and 5173.
  - Forcibly kills matching process IDs (`taskkill /F /PID` or `Stop-Process -Force`).
- **Risks & Edge Cases:** If another unrelated application (e.g., an alternate web service) is legitimately using port 8000, running this script will forcibly kill it.

#### [LOCAL_SETUP.md](file:///C:/Users/boazs/Downloads/aaaa/LOCAL_SETUP.md)
- **Role:** Operational runbook and configuration instructions for the local developer environment.
- **Why Needed:** Provides exact version requirements, prerequisite commands, environment variable explanations, and troubleshooting steps.
- **Dependencies:** Relies on accurate relative paths pointing to `aaaa/backend` and `aaaa/frontend`.

#### [bus_tracker.db](file:///C:/Users/boazs/Downloads/aaaa/bus_tracker.db) (Root Instance)
- **Role:** Empty SQLite database file at workspace root.
- **Why Needed / Origin:** When `uvicorn` is executed directly from the root workspace directory with default setting `DATABASE_URL=sqlite:///./bus_tracker.db`, SQLite creates this database in the current working directory.
- **Important Note:** The **active, populated** database with historical user and trip records is located inside [aaaa/backend/bus_tracker.db](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/bus_tracker.db). The root file is redundant but harmless.

---

### 6.2 Backend Application Core

#### [app/main.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/main.py)
- **Role:** Entry point for the FastAPI backend application.
- **Why Needed:** Assembles all modular components into a runnable ASGI application instance.
- **Responsibilities:**
  - Initializes database schema on startup via `Base.metadata.create_all(bind=engine)`.
  - Configures Cross-Origin Resource Sharing (`CORSMiddleware`) to allow requests from `http://localhost:5173`, `capacitor://localhost`, and local network IPs.
  - Mounts routers: `auth_router`, `trips_router`, and `predict_router` under prefix `/api/v1`.
  - Defines the `GET /health` sanity check endpoint returning `{"status": "ok"}`.
- **Dependencies:** FastAPI, SQLAlchemy ORM models, configuration settings, and routers.
- **What Depends on It:** Uvicorn ASGI server targets `app.main:app`. Docker container runs this file.

#### [app/core/config.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/core/config.py)
- **Role:** Centralized configuration management via Pydantic `BaseSettings`.
- **Why Needed:** Eliminates hardcoded environment constants and parses values from environment variables or `.env` files with type enforcement.
- **Responsibilities:**
  - Defines `PROJECT_NAME`, `API_V1_STR` (`/api/v1`), `SECRET_KEY`, `ALGORITHM` (`HS256`), and `ACCESS_TOKEN_EXPIRE_MINUTES` (1440 minutes / 24 hours).
  - Implements a Pydantic `field_validator` (`assemble_cors_origins`) that cleanly transforms comma-separated origin strings or JSON arrays into Python lists.
- **Dependencies:** `pydantic-settings`, `pydantic`.
- **What Depends on It:** Imported by almost every backend file requiring system settings, database URLs, or authentication keys.

#### [app/core/security.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/core/security.py)
- **Role:** Cryptographic security and authentication dependency module.
- **Why Needed:** Centralizes password hashing, JWT token creation/decoding, and route-level authorization guards.
- **Responsibilities:**
  - Encapsulates Passlib `CryptContext` using the `bcrypt` hashing algorithm.
  - Implements `create_access_token` to sign payloads containing user `id`, `role`, and expiration timestamp.
  - Implements FastAPI dependency `get_current_user`, which validates incoming HTTP `Authorization: Bearer <token>` headers.
  - Implements role-based access control dependencies: `get_current_driver` and `get_current_passenger`, raising HTTP 403 Forbidden if a user attempts unauthorized actions.
- **Dependencies:** `jose` (JWT), `passlib`, `fastapi`, SQLAlchemy `Session`.
- **What Breaks if Removed:** All authentication, trip creation, and role protection mechanisms immediately fail.

#### [app/core/stops.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/core/stops.py)
- **Role:** Corridor geographic data definitions and stop order indexer.
- **Why Needed:** Encodes the 9 physical stops along the Belman ↔ Udupi corridor and their sequential progression (1 to 9 or 9 to 1) based on travel direction.
- **Responsibilities:**
  - Maps `("belman_to_udupi", "Belman") -> 1` up to `("belman_to_udupi", "Udupi") -> 9`.
  - Maps reverse direction: `("udupi_to_belman", "Udupi") -> 1` down to `("udupi_to_belman", "Belman") -> 9`.
  - Defines `ALL_STOP_NAMES` for frontend dropdown population.
  - Maps API route keys (`belman_to_udupi`) to dataset route strings (`Belman → Udupi`).
- **Dependencies:** Pure Python dictionaries and helper functions.
- **What Depends on It:** Used by [predict.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/routers/predict.py) to look up `Stop_Order` feature values for the ML model.

#### [app/core/ws_manager.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/core/ws_manager.py)
- **Role:** In-memory WebSocket connection registry and pub/sub broadcast manager.
- **Why Needed:** Manages open, concurrent bi-directional communication channels between drivers and passengers.
- **Responsibilities:**
  - Maintains `passenger_sockets`: a dictionary mapping route strings (`belman_to_udupi`) to sets of active passenger `WebSocket` objects.
  - Maintains `driver_sockets`: a dictionary mapping active `trip_id` integers to specific driver `WebSocket` connections.
  - Implements `broadcast_to_route(route, payload)`: loops through connected passenger sockets, asynchronously sends JSON location payloads, and automatically purges dead sockets.
- **Dependencies:** Python `asyncio`, `json`, `fastapi.WebSocket`.
- **Risks & Limitations:** State is stored purely in the server process memory. If the backend is scaled horizontally across multiple instances (e.g. multi-worker Uvicorn or Kubernetes pods), a Redis Pub/Sub adapter would be necessary to synchronize broadcasts across instances.

#### [app/core/ml_models.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/core/ml_models.py)
- **Role:** Singleton machine learning model and lookup table loader.
- **Why Needed:** Prevents reloading multi-megabyte model binary files (`delay_model.pkl` is ~20 MB) on every individual prediction HTTP request.
- **Responsibilities:**
  - Loads `delay_model.pkl`, `crowd_model.pkl`, `encoders.pkl`, `bus_name_lookup.json`, `schedule_lookup.json`, and `schedule_timetable.json` into memory once during application startup.
  - Sets boolean flag `MODELS_LOADED = True` upon success; logs warnings and sets graceful fallbacks if files cannot be read.
- **Dependencies:** `joblib`, `json`, `pathlib`.
- **What Depends on It:** The `/predict` router imports these pre-loaded models directly.

---

### 6.3 Backend Database & Models

#### [app/db/database.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/db/database.py)
- **Role:** Database engine and session lifecycle manager.
- **Why Needed:** Manages database connectivity, connection pooling, and request-scoped session creation.
- **Responsibilities:**
  - Instantiates SQLAlchemy `engine` using `settings.DATABASE_URL`.
  - Configures `connect_args={"check_same_thread": False}` when SQLite is in use to allow FastAPI's multithreaded workers to interact with the database safely.
  - Defines `SessionLocal` factory and `get_db` generator dependency that guarantees database connections are closed (`db.close()`) after each HTTP request finishes.
  - Exposes declarative `Base` class for ORM entity mapping.
- **Dependencies:** `sqlalchemy`.

#### [app/models/user.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/models/user.py)
- **Role:** ORM entity representing registered users.
- **Why Needed:** Persists user credentials, names, and assigned access roles.
- **Responsibilities:**
  - Maps to table `users`.
  - Fields: `id` (integer primary key), `name` (string), `email` (unique string index), `hashed_password` (string), `role` (string: `"driver"` or `"passenger"`), and `created_at` (datetime).
- **Dependencies:** `sqlalchemy`, [app/db/database.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/db/database.py).

#### [app/models/trip.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/models/trip.py)
- **Role:** ORM entities representing physical bus trips and real-time GPS coordinates.
- **Why Needed:** Tracks active transit sessions and maintains the latest known spatial position of each bus.
- **Responsibilities:**
  - Defines `Trip`: maps to `trips` table. Includes `id`, `driver_id` (foreign key to `users.id`), `bus_name`, `route` (`belman_to_udupi` or `udupi_to_belman`), `status` (`"active"` or `"ended"`), `started_at`, and `ended_at`. Relates to `LivePosition` via a 1-to-1 relationship.
  - Defines `LivePosition`: maps to `live_positions` table with a unique constraint on `trip_id`. Includes `id`, `trip_id` (foreign key), `latitude` (float), `longitude` (float), and `updated_at` (datetime).
- **Dependencies:** `sqlalchemy`, [app/db/database.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/db/database.py).

#### [app/models/__init__.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/models/__init__.py)
- **Role:** Model registry index.
- **Why Needed:** Imports all models into one namespace so that `Base.metadata.create_all(bind=engine)` discovers all schemas before table generation occurs.

---

### 6.4 Backend Routers

#### [app/routers/auth.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/routers/auth.py)
- **Role:** Authentication REST controller.
- **Why Needed:** Handles user registration, credential verification, and profile queries.
- **Endpoints:**
  - `POST /api/v1/auth/signup`: Accepts name, email, password, and role. Validates email uniqueness, hashes the password with bcrypt, persists the user, and returns a signed JWT access token.
  - `POST /api/v1/auth/login`: Verifies user email and password; returns an access token, user role, and user name.
  - `GET /api/v1/auth/me`: Protected endpoint returning profile details of the authenticated caller.
- **Dependencies:** SQLAlchemy `Session`, [app/core/security.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/core/security.py), [app/schemas/user.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/schemas/user.py).

#### [app/routers/trips.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/routers/trips.py)
- **Role:** Trip lifecycle management and WebSocket communication endpoints.
- **Why Needed:** Coordinates the starting and ending of driver journeys, serves active bus snapshots, and drives real-time bi-directional streaming.
- **Responsibilities:**
  - Implements `cleanup_and_get_active_trips(db, route)`: identifies active trips with no GPS updates in the past 5 minutes, automatically updates their status to `ended`, and commits the change.
  - `POST /api/v1/trips/start`: Role-guarded to drivers. Marks previous active trips for this driver as ended, initializes a new active `Trip` row, and returns trip metadata.
  - `POST /api/v1/trips/{trip_id}/end`: Validates that the caller owns the trip, sets status to `ended`, and broadcasts a `trip_ended` event to all subscribed passengers.
  - `GET /api/v1/trips/active`: Returns all currently active trips and their latest latitude/longitude coordinates (filtered by route if requested).
  - `WebSocket /api/v1/trips/ws/driver/{trip_id}`: High-frequency driver endpoint. Validates token query parameter; receives location pings `{lat, lng, timestamp}`, upserts coordinates into `live_positions`, and calls `manager.broadcast_to_route`.
  - `WebSocket /api/v1/trips/ws/passenger/live`: Passenger map endpoint. Accepts route parameter, transmits an initial JSON snapshot of all live buses on the corridor, and subscribes the client to incoming live coordinate broadcasts.
- **Dependencies:** FastAPI WebSockets, SQLAlchemy ORM, [app/core/ws_manager.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/core/ws_manager.py), [app/core/security.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/core/security.py).

#### [app/routers/predict.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/routers/predict.py)
- **Role:** Machine learning inference controller.
- **Why Needed:** Serves ETA predictions, expected delays, and crowd estimations to passengers.
- **Responsibilities:**
  - `GET /api/v1/predict/stops`: Returns the static list of 9 corridor stops for UI dropdowns.
  - `POST /api/v1/predict`: Executes the project's signature **two-stage prediction pipeline**:
    1. Parses passenger input: stop name, route direction, reference time (`HH:MM`), and day type (`Weekday`/`Weekend`).
    2. Converts time to minutes since midnight; performs binary search over `schedule_timetable.json` to find the next scheduled departure minutes and scheduled bus name.
    3. Checks whether a live bus is currently running on the route via database query.
    4. Constructs a 12-feature vector using the **scheduled arrival minutes** (never the passenger's search time).
    5. Calls `delay_model.predict(feature_vector)` and adds corridor progression heuristics (peak hours, downstream stop congestion) to predict delay.
    6. Calls `crowd_model.predict(feature_vector)` to classify crowd level.
    7. Computes `predicted_arrival_time = normal_arrival + delay` and `passenger_wait_minutes = predicted_arrival - passenger_time`.
- **Dependencies:** NumPy, Bisect, Scikit-learn estimators, [app/core/ml_models.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/core/ml_models.py), [app/core/stops.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/core/stops.py).

---

### 6.5 Backend Schemas

#### [app/schemas/user.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/schemas/user.py)
- **Role:** Pydantic models for authentication contracts.
- **Models:** `UserCreate` (signup request), `UserLogin` (login credentials), `UserOut` (public profile omitting password), `Token` (JWT bearer response), and `TokenPayload` (internal decoded token contents).

#### [app/schemas/trip.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/schemas/trip.py)
- **Role:** Pydantic models for trip actions and live positions.
- **Models:** `TripStart` (validates bus name and route key), `TripOut` (basic trip metadata), `ActiveTripOut` (includes latest lat/lng and ISO updated timestamp), `LocationPing` (driver GPS packet `{lat, lng, timestamp}`), and `LivePositionOut`.

#### [app/schemas/predict.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/app/schemas/predict.py)
- **Role:** Pydantic models for ML prediction requests and responses.
- **Models:**
  - `PredictRequest`: Validates stop name, route key, time format (regex `^\d{2}:\d{2}$`), and day type.
  - `LiveBus`: Describes an actively running physical bus detected on the route.
  - `PredictResponse`: Complete output payload containing next bus name, normal scheduled arrival, predicted arrival, delay minutes, passenger wait minutes, predicted crowd level, and explanatory confidence notes.

---

### 6.6 Backend Root Files & Scripts

#### [requirements.txt](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/requirements.txt)
- **Role:** Python package dependency manifest.
- **Why Needed:** Specifies all required third-party libraries for pip installation.

#### [Dockerfile](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/Dockerfile) & [.dockerignore](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/.dockerignore)
- **Role:** Containerization specification for the backend.
- **Why Needed:** Allows building a portable Linux container image (`python:3.13-slim`) for Docker or cloud deployment platforms.
- **Design Decisions:** Explicitly copies only `app/` and `models/`, excluding the heavy raw Excel dataset (`.xlsx`) to maintain minimal image size.

#### [test_pipeline_validation.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/test_pipeline_validation.py)
- **Role:** Integration testing script for the prediction pipeline.
- **Responsibilities:** Executes 14 scenario requests against `http://localhost:8000/api/v1/predict` (covering both routes, morning/evening peak hours, intermediate stops, weekends, and late-night out-of-service edge cases) and validates response integrity across 40 distinct assertion checks.

#### [scripts/train_models.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/scripts/train_models.py)
- **Role:** Offline machine learning training script and timetable generator.
- **Why Needed:** Reads the 50,130-row master Excel file, engineers features, trains the regression and classification models, and compiles JSON timetable lookups.
- **Processing Steps:**
  1. Computes `Scheduled_Minutes = Arrival_Minutes - Delay_Minutes`.
  2. Encodes routes (binary), day types (binary), stop names (LabelEncoder), time slots (ordinal 0-4), weather (one-hot), holidays (binary), and crowd levels (ordinal).
  3. Performs an 80/20 temporal train/test split sorted strictly by calendar date to prevent data leakage across trips.
  4. Trains `RandomForestRegressor(n_estimators=200, max_depth=12)` on delays (achieves MAE ~0.8 minutes).
  5. Trains `GradientBoostingClassifier(n_estimators=200, max_depth=5)` on crowd density (achieves ~85% accuracy).
  6. Compiles and exports `schedule_timetable.json`: groups scheduled minutes by route, stop, and day type, merges departures closer than 8 minutes to maintain realistic 10-20 minute transit headways.
  7. Serializes all artefacts to `models/` using Joblib and JSON.

---

### 6.7 Machine Learning Assets & Dataset

#### [dataset/Master_Bus_Passenger_Flow.xlsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/dataset/Master_Bus_Passenger_Flow.xlsx)
- **Role:** Primary historical transit dataset (50,130 rows, 25 columns).
- **Features Included:** Date, Day, Day_Type, Route (`Belman → Udupi`, `Udupi → Belman`), Bus_Name (18 unique private fleet names), Bus_Capacity (65), Stop_Name (9 stops), Stop_Order (1-9), Arrival_Time, Arrival_Minutes, Time_Slot, Is_Peak_Hour, Passengers, Occupancy_Pct, Crowd_Label (`Low`, `Medium`, `High`), Weather (`Sunny`, `Cloudy`, `Rainy`), Holiday, Delay_Minutes.

#### Serialized Artefacts in `models/`
- **[delay_model.pkl](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/models/delay_model.pkl):** Serialized scikit-learn Random Forest model predicting continuous arrival delay in minutes.
- **[crowd_model.pkl](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/models/crowd_model.pkl):** Serialized Gradient Boosting Classifier predicting crowd category (0: Low, 1: Medium, 2: High).
- **[encoders.pkl](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/models/encoders.pkl):** Dictionary of mapping tables and LabelEncoder classes ensuring that production inference transforms match training transforms exactly.
- **[schedule_timetable.json](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/models/schedule_timetable.json):** Sorted master departure schedule array per `route|stop|day_type`.
- **[schedule_lookup.json](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/models/schedule_lookup.json):** Aggregate arrival metrics (median, mean, min, max scheduled minutes) by time slot.
- **[bus_name_lookup.json](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/models/bus_name_lookup.json):** Fallback dominant bus name mapping by time slot.

---

### 6.8 Frontend Root & Configuration

#### [package.json](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/package.json) & [package-lock.json](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/package-lock.json)
- **Role:** Node.js project manifest. Defines scripts (`dev`, `build`, `preview`) and dependencies including React 19, Tailwind CSS v4, Leaflet, Axios, and Capacitor.

#### [vite.config.js](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/vite.config.js)
- **Role:** Vite configuration file. Configures `@vitejs/plugin-react` and `@tailwindcss/vite` plugins. Sets development server host to `0.0.0.0` (accessible from external devices on LAN) on port 5173.

#### [capacitor.config.json](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/capacitor.config.json)
- **Role:** Capacitor mobile runtime configuration.
- **Attributes:** Package ID `com.akash.bustracker`, App Name `BusTracker ML`, Web directory target `dist`, and Android scheme `https`.

#### [index.html](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/index.html)
- **Role:** Single Page Application HTML entry container. Hosts `<div id="root"></div>` and loads `/src/main.jsx`.

#### [.env](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/.env) & [.env.production](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/.env.production)
- **Role:** Frontend environment variables configuring `VITE_API_BASE_URL` and `VITE_WS_BASE_URL`.

---

### 6.9 Frontend Components

#### [src/components/Navbar.jsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/components/Navbar.jsx)
- **Role:** Top navigation header.
- **Responsibilities:** Displays branding, role indicator pills (`DRIVER` vs `PASSENGER`), direct navigation links, and a global logout button.

#### [src/components/ProtectedRoute.jsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/components/ProtectedRoute.jsx)
- **Role:** React Router authorization guard.
- **Responsibilities:** Evaluates `isAuthenticated` and `role`. Displays a loading spinner while session rehydration is pending, redirects unauthenticated visitors to `/`, and redirects users attempting to access the incorrect dashboard (e.g. passenger attempting `/driver`) to their assigned role route.

#### [src/components/Button.jsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/components/Button.jsx)
- **Role:** Standardized UI button component supporting 4 style variants (`primary`, `secondary`, `danger`, `outline`) with hover animations and disabled state styling.

#### [src/components/Card.jsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/components/Card.jsx)
- **Role:** Glassmorphic container panel supporting headers, action slots, and uniform rounded borders.

#### [src/components/LoadingSpinner.jsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/components/LoadingSpinner.jsx)
- **Role:** Visual spinner element for asynchronous operations with customizable sizes (`sm`, `md`, `lg`) and status messages.

#### [src/components/SkeletonBlock.jsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/components/SkeletonBlock.jsx)
- **Role:** Shimmering animated placeholders (`PredictionSkeleton`, `TripStatusSkeleton`, `MapLoadingSkeleton`) used during data fetching to eliminate jarring layout shifts.

---

### 6.10 Frontend Contexts & Hooks

#### [src/context/AuthContext.jsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/context/AuthContext.jsx)
- **Role:** Global authentication state provider.
- **Responsibilities:**
  - Manages `user`, `token`, `role`, and `isAuthenticated` states.
  - Automatically rehydrates sessions on browser reload by reading `localStorage` and calling `GET /api/v1/auth/me`.
  - Listens for global window event `auth:unauthorized` dispatched by the Axios interceptor, triggering clean automatic logout upon token expiration.

#### [src/context/ToastContext.jsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/context/ToastContext.jsx)
- **Role:** Transient alert notification system.
- **Responsibilities:** Renders toast messages (`success`, `error`, `warning`, `info`) fixed at the top right of the viewport with timed dismissals and progress bar animations.

#### [src/hooks/useAuth.js](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/hooks/useAuth.js)
- **Role:** Consumer hook providing convenient access to `AuthContext` with validation guarding against usage outside `AuthProvider`.

---

### 6.11 Frontend Pages

#### [src/pages/LoginPage.jsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/pages/LoginPage.jsx)
- **Role:** Authentication view.
- **Responsibilities:** Features a tab switcher for Sign In vs Sign Up, form input validation (email format, 6-character password rules), interactive role selector (`Passenger` vs `Driver`), and automatic redirect upon successful login.

#### [src/pages/DriverPage.jsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/pages/DriverPage.jsx)
- **Role:** Driver operations portal.
- **Responsibilities:**
  - Provides trip creation form (bus name/number and route selection).
  - Taps into device GPS using Capacitor native Geolocation (on mobile) or HTML5 Geolocation API (on web).
  - Connects to driver WebSocket: `ws://host/api/v1/trips/ws/driver/{trip_id}?token=...`.
  - Pings location every ~4 seconds.
  - Implements exponential backoff reconnection logic (1s, 2s, 4s delays, max 3 attempts) and manual retry buttons.
  - Displays real-time trip status, elapsed timer, last sent coordinates, and an "End Trip" termination control.

#### [src/pages/PassengerPage.jsx](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/pages/PassengerPage.jsx)
- **Role:** Commuter portal.
- **Responsibilities:**
  - **Live Map Tab:**
    - Renders an interactive OpenStreetMap via React-Leaflet with markers for all 9 corridor stops.
    - Connects to passenger WebSocket: `ws://host/api/v1/trips/ws/passenger/live?route=...`.
    - Receives initial bus snapshots and streaming coordinate updates.
    - Uses `AnimatedBusMarker` with `requestAnimationFrame` and ease-out interpolation to smoothly glide bus markers across the map between 4-second GPS pings.
    - Optional passenger geolocation layer allowing commuters to pinpoint their own position relative to oncoming buses.
  - **Prediction Tab:**
    - Form to query arrival predictions by stop, route, passenger time, and day type.
    - Calls `POST /api/v1/predict` and renders structured cards displaying next bus name, normal arrival, predicted arrival, expected delay, passenger wait time, crowd density badge, and confidence notes.

---

### 6.12 Frontend Services & Utilities

#### [src/services/api.js](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/services/api.js)
- **Role:** Preconfigured Axios instance.
- **Responsibilities:**
  - Injects `Authorization: Bearer <token>` into outgoing request headers if a token exists in `localStorage`.
  - Intercepts 401 Unauthorized responses, clears local credentials, and dispatches the `auth:unauthorized` custom event.

#### [src/config.js](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/config.js)
- **Role:** Dynamic host and protocol resolver.
- **Why Needed:** Solves mobile testing difficulties where hardcoded `localhost` would point to the phone itself rather than the host computer running the backend server.
- **Responsibilities:** Dynamically checks `window.location.hostname` and `window.location.protocol`, automatically formatting `http://<LAN-IP>:8000` or `ws://<LAN-IP>:8000`.

#### Unused Vite Boilerplate Files in `src/`
- **[counter.ts](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/counter.ts), [main.ts](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/main.ts), [style.css](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/style.css), [assets/hero.png](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/assets/hero.png), [assets/typescript.svg](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/src/assets/typescript.svg):**
  - **Origin:** Leftover template files from initial Vite project scaffolding (`npm create vite@latest`).
  - **Status:** Unused by the active React runtime (which uses `main.jsx` and `App.jsx`). They can be safely archived or ignored.

---

### 6.13 Mobile Container Files (`android/`)

#### [android/app/src/main/AndroidManifest.xml](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/android/app/src/main/AndroidManifest.xml)
- **Role:** Native Android application declaration manifest.
- **Responsibilities:** Configures required permissions: `android.permission.INTERNET`, `android.permission.ACCESS_FINE_LOCATION`, `android.permission.ACCESS_COARSE_LOCATION`, and `android.hardware.location.gps`. Enables `android:usesCleartextTraffic="true"` to permit local HTTP/WS communication during testing.

#### [android/app/src/main/java/com/akash/bustracker/MainActivity.java](file:///C:/Users/boazs/Downloads/aaaa/aaaa/frontend/android/app/src/main/java/com/akash/bustracker/MainActivity.java)
- **Role:** Native Android activity entry point extending Capacitor's `BridgeActivity`.

---

## 7. Folder-by-Folder Explanation

| Folder Path | Purpose | Primary File Types | Key Role in Project |
|---|---|---|---|
| **`/` (Root)** | Workspace root | `.bat`, `.ps1`, `.db`, `.md` | Environment bootstrapping, process management, setup runbooks. |
| **`aaaa/`** | Core source wrapper | `.md`, scripts, directories | Holds the entire backend, frontend, and deployment configurations. |
| **`aaaa/backend/`** | Python server root | `.py`, `.env`, `.txt`, Dockerfile | Hosts FastAPI server, virtual environment, and test suites. |
| **`aaaa/backend/app/`** | Application code | Python modules (`.py`) | Core application logic, routers, database ORM, and schemas. |
| **`aaaa/backend/app/core/`** | Core singletons | Python modules (`.py`) | Security, config, ML model loaders, stop tables, WebSocket manager. |
| **`aaaa/backend/app/db/`** | Database layer | Python modules (`.py`) | SQLAlchemy engine creation and database session generator. |
| **`aaaa/backend/app/models/`**| Database schemas | Python modules (`.py`) | Declarative table mappings for User, Trip, and LivePosition entities. |
| **`aaaa/backend/app/routers/`| API endpoints | Python modules (`.py`) | REST route handlers and WebSocket listener functions. |
| **`aaaa/backend/app/schemas/`| Data contracts | Python modules (`.py`) | Pydantic validation models for requests and responses. |
| **`aaaa/backend/dataset/`** | Raw training data | Excel (`.xlsx`) | Contains the 50,130-row master passenger and trip flow dataset. |
| **`aaaa/backend/models/`** | Serialized ML models | `.pkl`, `.json` | Trained Random Forest, Gradient Boosting, encoders, and timetable lookups. |
| **`aaaa/backend/scripts/`** | Offline pipelines | Python modules (`.py`) | Standalone ML training script (`train_models.py`). |
| **`aaaa/frontend/`** | Web application root| `.json`, `.js`, `.html`, `.env` | Vite and npm configuration, Capacitor project definition. |
| **`aaaa/frontend/public/`** | Static assets | `.svg` | Unprocessed static files served at the web server root. |
| **`aaaa/frontend/src/`** | Frontend source | `.jsx`, `.js`, `.css` | React components, pages, contexts, hooks, and services. |
| **`aaaa/frontend/src/components/`** | Reusable UI | React JSX (`.jsx`) | Reusable buttons, cards, skeletons, spinners, and navbar. |
| **`aaaa/frontend/src/context/`** | Global state | React JSX (`.jsx`) | React Context providers for authentication and toast alerts. |
| **`aaaa/frontend/src/hooks/`** | Custom hooks | JavaScript (`.js`) | Convenience wrappers around React Context hooks. |
| **`aaaa/frontend/src/pages/`** | Page views | React JSX (`.jsx`) | Primary views: LoginPage, DriverPage, and PassengerPage. |
| **`aaaa/frontend/src/services/`**| HTTP abstraction | JavaScript (`.js`) | Axios client with request and response interceptors. |
| **`aaaa/frontend/android/`** | Native mobile shell | Java, Gradle, XML | Native Android Studio project generated by Capacitor. |

---

## 8. Application Startup and Initialization

### Startup Sequence Diagram

```
[Developer / start-dev.bat]
         │
         ├── 1. Spawns Backend Process (Uvicorn)
         │        │
         │        ├── 2. Reads backend/.env via pydantic BaseSettings
         │        ├── 3. Initializes SQLite engine (bus_tracker.db)
         │        ├── 4. Calls Base.metadata.create_all(bind=engine) -> registers users, trips, live_positions
         │        ├── 5. Loads delay_model.pkl, crowd_model.pkl, encoders.pkl, and timetable JSON into RAM
         │        ├── 6. Mounts CORS, auth_router, trips_router, predict_router under /api/v1
         │        └── 7. Begins listening on http://0.0.0.0:8000
         │
         └── 8. Spawns Frontend Process (Vite)
                  │
                  ├── 9. Reads frontend/.env (VITE_API_BASE_URL, etc.)
                  ├── 10. Compiles Tailwind CSS v4 and React JSX
                  ├── 11. Serves Single Page App on http://localhost:5173
                  │
[User Opens Browser / http://localhost:5173]
         │
         ├── 12. React loads App.jsx inside AuthProvider & ToastProvider
         ├── 13. AuthContext checks localStorage for existing JWT token
         │         └── If token found -> issues GET /api/v1/auth/me to rehydrate profile
         └── 14. ProtectedRoute evaluates authentication status and renders appropriate page
```

---

## 9. Complete End-to-End Project Workflow

### Scenario A: Driver Broadcasts Live Trip
1. **Login:** Driver signs in via `LoginPage.jsx`. Credentials validated by `POST /api/v1/auth/login`. Server returns JWT token with role `"driver"`.
2. **Setup:** Driver redirected to `DriverPage.jsx`. Driver enters bus name (e.g., `"KA-20-B-1234"`), selects route direction (`"belman_to_udupi"`), and taps **Start Trip**.
3. **Trip Record Creation:** Frontend calls `POST /api/v1/trips/start`. Backend marks any prior active trip for this driver as `"ended"` and inserts a new `Trip` row with status `"active"`.
4. **WebSocket Connection:** Driver client opens a WebSocket connection to `ws://host/api/v1/trips/ws/driver/{trip_id}?token=...`. The backend decodes the token, verifies that the driver owns the trip, accepts the connection, and registers the socket in `ConnectionManager`.
5. **GPS Streaming:** The browser or mobile device initiates `navigator.geolocation.watchPosition` or Capacitor `Geolocation.watchPosition`.
6. **Throttled Pings:** Every 4 seconds, the client sends `{lat, lng, timestamp}` over the WebSocket.
7. **Database Upsert & Broadcast:** Backend receives the ping, updates the latitude, longitude, and timestamp in `live_positions`, and calls `manager.broadcast_to_route`.
8. **Ending the Trip:** Driver taps **End Trip**. Frontend calls `POST /api/v1/trips/{trip_id}/end`, closes the WebSocket, and stops the GPS watcher. Backend sets status to `"ended"` and broadcasts a `trip_ended` event to all passengers.

### Scenario B: Passenger Live Map Tracking
1. **Navigation:** Passenger logs in and navigates to `PassengerPage.jsx`, landing on the **Live Map** tab.
2. **WebSocket Subscription:** Component initiates a WebSocket connection to `ws://host/api/v1/trips/ws/passenger/live?route=belman_to_udupi`.
3. **Snapshot Delivery:** Backend runs `cleanup_and_get_active_trips`, marks any stale trips as ended, and transmits an immediate JSON snapshot containing all live buses on the selected route.
4. **Map Rendering:** React-Leaflet renders the Belman ↔ Udupi map with pins for all 9 stops and bus markers for active trips.
5. **Real-Time Moving Buses:** As driver pings arrive over the WebSocket, `AnimatedBusMarker` catches coordinate changes and executes a smooth 2.5-second `requestAnimationFrame` ease-out glide, preventing jerky map hopping.

### Scenario C: Commuter Queries Arrival & Crowd Prediction
1. **Query Entry:** Passenger opens the **Prediction** tab, selects a stop (e.g. `"Bantakal"`), route (`"belman_to_udupi"`), reference time (`"08:20"`), and day type (`"Weekday"`).
2. **API Request:** Frontend calls `POST /api/v1/predict`.
3. **Timetable Lookup (Stage 1):** Backend converts `08:20` to minute 500; binary-searches `schedule_timetable.json` and finds the next scheduled departure at minute 514 (normal arrival `08:34`, bus name `"Prince"`).
4. **Machine Learning Inference (Stage 2):** Backend formats the 12-feature vector using minute 514 and stop order 4. `delay_model.predict` evaluates expected road delay (e.g., 2.0 minutes). `crowd_model.predict` evaluates passenger occupancy (e.g., `"High"`).
5. **Live Verification:** Backend checks if a live bus is currently running on the route; if found, attaches live details to the response.
6. **Display:** Passenger receives results: Next Bus: *Prince*, Scheduled: *08:34*, Predicted: *08:36* (+2 min delay), Commuter Wait: *16 minutes*, Crowd: *High*.

---

## 10. File Dependencies and Module Relationships

### Import & Dependency Flow Table

| Source File | What It Imports / Depends On | Why This Connection Exists |
|---|---|---|
| `app/main.py` | `app/core/config.py`, `app/db/database.py`, `app/routers/*` | Server entry point; mounts routes and sets CORS. |
| `app/db/database.py` | `app/core/config.py` | Requires `DATABASE_URL` to instantiate engine. |
| `app/models/trip.py` | `app/db/database.py`, `app/models/user.py` | Inherits ORM `Base`; foreign-keys `User.id`. |
| `app/core/security.py` | `app/core/config.py`, `app/db/database.py`, `app/models/user.py` | Needs secret keys and queries `User` table to validate tokens. |
| `app/routers/auth.py` | `app/core/security.py`, `app/models/user.py`, `app/schemas/user.py` | Handles password hashing, token generation, user persistence. |
| `app/routers/trips.py` | `app/core/ws_manager.py`, `app/core/security.py`, `app/models/trip.py` | Coordinates GPS streaming, live positions, and WebSocket broadcasts. |
| `app/routers/predict.py`| `app/core/ml_models.py`, `app/core/stops.py`, `app/models/trip.py` | Uses loaded ML estimators, stop tables, and active trips. |
| `frontend/src/App.jsx` | `AuthContext`, `ToastContext`, `Navbar`, `ProtectedRoute`, Pages | Root component defining client-side routes. |
| `frontend/src/pages/DriverPage.jsx` | `api.js`, `useAuth`, `useToast`, `config.js`, Capacitor Geolocation | Driver console; handles GPS pings and WebSocket stream. |
| `frontend/src/pages/PassengerPage.jsx` | `api.js`, `useAuth`, `useToast`, `react-leaflet`, `config.js` | Commuter portal; renders map and prediction tab. |

### What Breaks If an Important File Is Removed?
- **Removing `app/core/ws_manager.py`:** Backend startup crashes immediately; both driver and passenger WebSocket channels fail completely.
- **Removing `app/models/schedule_timetable.json`:** Prediction router crashes on import; ML predictions become non-functional.
- **Removing `frontend/src/config.js`:** Mobile devices and LAN clients fail to resolve backend endpoints, breaking all network requests on mobile.
- **Removing `app/core/security.py`:** All protected routes, password hashing, and user authentication break entirely.

---

## 11. Feature-by-Feature Analysis

### Feature 1: Driver Real-Time GPS Streaming
- **Purpose:** Allows drivers to broadcast high-frequency spatial locations without dedicated GPS hardware.
- **Implementation Files:** `DriverPage.jsx`, `trips.py` (`driver_ws`), `ws_manager.py`.
- **Inputs:** Device GPS hardware coordinates (`latitude`, `longitude`), Bearer JWT token, `trip_id`.
- **Processing:** Client throttles pings to 4-second intervals. Backend upserts `LivePosition` in SQLite and broadcasts JSON payloads to connected route subscribers.
- **Outputs:** Real-time location acknowledgment sent back to driver; position broadcast to passengers.
- **Edge Cases:** Location access denied by browser, network dropouts, device sleep mode. Handled via exponential backoff reconnects (1s, 2s, 4s) and UI warning banners.

### Feature 2: Commuter Live Interactive Map
- **Purpose:** Gives passengers an immediate visual overview of bus positions along the corridor.
- **Implementation Files:** `PassengerPage.jsx`, `AnimatedBusMarker`, Leaflet, `trips.py` (`passenger_ws`).
- **Inputs:** Selected route (`belman_to_udupi` / `udupi_to_belman`).
- **Processing:** Establishes WebSocket subscription. On open, receives snapshot of current active buses. On location updates, smoothly shifts bus markers using requestAnimationFrame interpolation.
- **Outputs:** Interactive map with 9 labeled stop markers and animated bus icons displaying trip ID, route, and relative time-ago timestamps.

### Feature 3: Two-Stage Machine Learning Arrival & Delay Prediction
- **Purpose:** Provides accurate bus arrival times and expected delays based on historical data.
- **Implementation Files:** `predict.py`, `train_models.py`, `ml_models.py`, `schedule_timetable.json`.
- **Inputs:** Stop name, route direction, passenger reference time (`HH:MM`), day type (`Weekday`/`Weekend`).
- **Processing:**
  1. Binary search finds next scheduled departure in `schedule_timetable.json`.
  2. Constructs 12-feature vector using the scheduled minute.
  3. `RandomForestRegressor` evaluates expected road delay.
  4. Heuristic traffic adjustments added for peak hours and downstream progression.
- **Outputs:** Next bus name, scheduled arrival time, predicted arrival time, delay minutes, passenger wait minutes, and confidence note.

### Feature 4: Commuter Crowd Density Classification
- **Purpose:** Informs passengers of anticipated bus crowding so they can plan their travel comfortably.
- **Implementation Files:** `predict.py`, `crowd_model.pkl`, `PassengerPage.jsx`.
- **Inputs:** 12-feature vector derived from scheduled trip parameters.
- **Processing:** `GradientBoostingClassifier` computes class probabilities and outputs class 0 (`Low`), 1 (`Medium`), or 2 (`High`).
- **Outputs:** Visual crowd badge displayed in the prediction result card with custom color coding (Green: Low, Amber: Medium, Red: High).

### Feature 5: Stale-Trip Garbage Collection
- **Purpose:** Prevents "ghost buses" from lingering on the map when drivers close their browser or lose battery without ending their trip.
- **Implementation Files:** `cleanup_and_get_active_trips` in `trips.py`.
- **Logic:** Whenever active trips are listed or a passenger connects to the map, the system checks whether the trip's latest GPS ping is older than 5 minutes. If so, it automatically updates the trip's status to `"ended"` and sets `ended_at = datetime.utcnow()`.

---

## 12. Frontend Workflow

```
       [User opens frontend]
                 │
                 ▼
       [index.html / main.jsx]
                 │
                 ▼
       [App.jsx / AuthProvider] ──► Reads localStorage token & calls GET /auth/me
                 │
        ┌────────┴────────┐
        ▼                 ▼
[Unauthenticated]   [Authenticated]
        │                 │
        ▼                 ▼
  [LoginPage.jsx]   [ProtectedRoute]
                          │
                  ┌───────┴───────┐
                  ▼               ▼
           (role="driver")  (role="passenger")
                  │               │
                  ▼               ▼
          [DriverPage.jsx]  [PassengerPage.jsx]
           - Start Trip      - Tab 1: Live Leaflet Map
           - GPS 4s Ping     - Tab 2: ML Arrival Prediction
           - WS Stream       - Smooth AnimatedBusMarker
```

---

## 13. Backend Workflow

```
        [Incoming HTTP / WS Request]
                     │
                     ▼
             [Uvicorn Server]
                     │
                     ▼
          [FastAPI Middleware & CORS]
                     │
         ┌───────────┼───────────┐
         ▼           ▼           ▼
   [/api/v1/auth] [/api/v1/trips] [/api/v1/predict]
         │           │           │
         │           │           ├── 1. Binary search schedule_timetable.json
         │           │           ├── 2. Build 12-feature vector
         │           │           ├── 3. delay_model.predict()
         │           │           ├── 4. crowd_model.predict()
         │           │           └── 5. Return PredictResponse
         │           │
         │           ├── [REST: /start, /end, /active] ──► SQLite (bus_tracker.db)
         │           │
         │           ├── [WS: /ws/driver/{trip_id}]
         │           │     ├── Validate JWT token
         │           │     ├── Upsert LivePosition in DB
         │           │     └── ws_manager.broadcast_to_route()
         │           │
         │           └── [WS: /ws/passenger/live]
         │                 ├── Send initial snapshot
         │                 └── Register socket in passenger_sockets
         │
         ▼
[Bcrypt / JWT / SQLite (users table)]
```

---

## 14. Database and Data Flow

### Database Engine & Storage
The application utilizes an **embedded SQLite database** stored at `aaaa/backend/bus_tracker.db`. SQLite requires no separate background service or port binding.

### Entity Relationship Diagram

```
+-------------------------------------------------------+
|                        users                          |
+-------------------------------------------------------+
| id              : INTEGER (Primary Key, Indexed)      |
| name            : VARCHAR                             |
| email           : VARCHAR (Unique, Indexed)           |
| hashed_password : VARCHAR                             |
| role            : VARCHAR ("driver" | "passenger")     |
| created_at      : DATETIME                            |
+---------------------------+---------------------------+
                            | 1
                            |
                            | has many
                            v *
+-------------------------------------------------------+
|                        trips                          |
+-------------------------------------------------------+
| id              : INTEGER (Primary Key, Indexed)      |
| driver_id       : INTEGER (Foreign Key -> users.id)   |
| bus_name        : VARCHAR                             |
| route           : VARCHAR ("belman_to_udupi", etc.)   |
| status          : VARCHAR ("active" | "ended")        |
| started_at      : DATETIME                            |
| ended_at        : DATETIME (Nullable)                 |
+---------------------------+---------------------------+
                            | 1
                            |
                            | has one
                            v 1 (Unique Constraint)
+-------------------------------------------------------+
|                    live_positions                     |
+-------------------------------------------------------+
| id              : INTEGER (Primary Key, Indexed)      |
| trip_id         : INTEGER (Foreign Key -> trips.id)   |
| latitude        : FLOAT                               |
| longitude       : FLOAT                               |
| updated_at      : DATETIME                            |
+-------------------------------------------------------+
```

---

## 15. APIs and External Integrations

### REST Endpoints Summary

| Method | Path | Authentication | Description |
|---|---|---|---|
| `GET` | `/health` | None | System health check returning `{"status": "ok"}`. |
| `POST` | `/api/v1/auth/signup` | None | Register new user account; returns JWT token. |
| `POST` | `/api/v1/auth/login` | None | Authenticate user credentials; returns JWT token. |
| `GET` | `/api/v1/auth/me` | Bearer Token | Fetch current user profile. |
| `POST` | `/api/v1/trips/start` | Bearer (Driver) | Create a new active trip. |
| `POST` | `/api/v1/trips/{trip_id}/end` | Bearer (Driver) | Terminate an active trip. |
| `GET` | `/api/v1/trips/active` | None | List active trips and current coordinates. |
| `GET` | `/api/v1/predict/stops` | None | Return list of 9 corridor stops. |
| `POST` | `/api/v1/predict` | None | Execute ML arrival and crowd prediction. |

### WebSocket Endpoints Summary

| Protocol | Path | Parameters | Role |
|---|---|---|---|
| `ws://` | `/api/v1/trips/ws/driver/{trip_id}` | `token` (JWT query param) | Receives driver GPS pings and broadcasts to route subscribers. |
| `ws://` | `/api/v1/trips/ws/passenger/live` | `route` (Route key query param) | Transmits bus snapshot on connect and streams live updates. |

### External Integrations
1. **OpenStreetMap Tile Servers:** Leaflet fetches free map tiles from `https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png`.
2. **Capacitor Hardware Geolocation Bridge:** `@capacitor/geolocation` invokes Android's native `FusedLocationProviderClient` to acquire high-accuracy GPS fixes on physical devices.

---

## 16. Configuration and Environment Variables

### Backend Configuration (`aaaa/backend/.env`)
- `SECRET_KEY`: Cryptographic secret string used to sign and verify HMAC-SHA256 JWT tokens.
- `DATABASE_URL`: Connection string for SQLAlchemy (defaults to `sqlite:///./bus_tracker.db`).
- `CORS_ORIGINS`: Comma-separated list of allowed origins (e.g. `http://localhost:5173,capacitor://localhost`).

### Frontend Configuration (`aaaa/frontend/.env` & `.env.production`)
- `VITE_API_BASE_URL`: Base HTTP URL for REST calls (e.g., `http://localhost:8000`).
- `VITE_WS_BASE_URL`: Base WebSocket URL for streaming channels (e.g., `ws://localhost:8000`).

---

## 17. Authentication and Security

- **Stateless JWT Tokens:** Authentication relies on RFC 7519 JSON Web Tokens signed with HS256. The server does not maintain server-side user session tables.
- **Password Protection:** Plaintext passwords are never stored; they are hashed with `bcrypt` (work factor 12) via Passlib.
- **Role-Based Access Control (RBAC):** Backend route dependencies verify user role claims before allowing trip creation.
- **Client Route Guards:** `ProtectedRoute.jsx` intercepts unauthorized page transitions in the browser.
- **Cleartext Traffic Policy:** `AndroidManifest.xml` enables `usesCleartextTraffic="true"` solely for local testing over unencrypted LAN IP addresses (`http://` and `ws://`).

---

## 18. Error Handling and Logging

- **HTTP Status Codes:** Standardized codes used throughout: `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `503 Service Unavailable`.
- **Axios Interceptor:** Catches `401 Unauthorized` responses globally, wipes stale credentials from `localStorage`, and dispatches an event that returns the UI to the login screen.
- **WebSocket Reconnection:** Drivers implement exponential backoff reconnection (1s, 2s, 4s) up to 3 attempts, showing clear status banners and a manual retry button.
- **Graceful ML Fallbacks:** If input falls outside scheduled service hours, the API returns a structured `no_prediction: true` payload rather than crashing.

---

## 19. Testing and Quality Assurance

### Automated Backend Pipeline Validation
The backend includes an automated validation suite in [test_pipeline_validation.py](file:///C:/Users/boazs/Downloads/aaaa/aaaa/backend/test_pipeline_validation.py).
- **Execution Command:**
  ```powershell
  cd aaaa\backend
  .\.venv\Scripts\python.exe test_pipeline_validation.py
  ```
- **Coverage:** Runs 14 comprehensive test queries across peak hours, intermediate stops, both route directions, and late-night conditions, validating 40 distinct assertion checks.

### Frontend Type-Checking & Build Verification
- **Execution Command:**
  ```powershell
  cd aaaa\frontend
  npm.cmd run build
  ```
- **Validation:** Verifies JSX compilation, Tailwind CSS bundling, and production asset generation into `frontend/dist/`.

---

## 20. Installation and Setup

### System Prerequisites
- **Python:** Version 3.13 or 3.14 (64-bit).
- **Node.js:** Version 20.x or 22.x with npm.
- **Operating System:** Windows 10/11, macOS, or Linux.

### Step-by-Step Installation

#### 1. Backend Setup
```powershell
cd aaaa\backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

#### 2. Frontend Setup
```powershell
cd aaaa\frontend
npm.cmd install
```

---

## 21. Running, Building, and Deployment

### Option A: One-Click Development Launch (Recommended)
From the project root:
```powershell
.\start-dev.bat
```
To shut down all running services:
```powershell
.\stop-dev.bat
```

### Option B: Manual Command-Line Launch

**Terminal 1 — Backend:**
```powershell
cd aaaa\backend
.\.venv\Scripts\python.exe -m uvicorn app.main:app --host 0.0.0.0 --port 8000
```

**Terminal 2 — Frontend:**
```powershell
cd aaaa\frontend
npm.cmd run dev
```

### Option C: Docker Containerization
```bash
cd aaaa/backend
docker build -t bus-tracker-api .
docker run -d -p 8000:8000 --name bus-tracker-backend bus-tracker-api
```

### Option D: Android Mobile APK Build (Capacitor)
```bash
cd aaaa/frontend
npm run build
npx cap sync android
npx cap open android
```
In Android Studio, connect your physical Android device with USB Debugging enabled and click **Run ▶**.

---

## 22. Troubleshooting Guide

| Issue / Symptom | Root Cause | Solution |
|---|---|---|
| `Errno 10048: error while attempting to bind on address ('0.0.0.0', 8000)` | Port 8000 is still bound by an orphaned background Python process. | Run `stop-dev.bat` or `stop-dev.ps1` to terminate the lingering process. |
| `File npm.ps1 cannot be loaded because running scripts is disabled` | Windows PowerShell Execution Policy restricts unsigned scripts. | Use `npm.cmd` instead of `npm`, or launch through `start-dev.bat`. |
| `InconsistentVersionWarning: Trying to unpickle estimator ...` | Scikit-learn minor version difference between training and runtime. | Harmless warning. To retrain cleanly, run: `python scripts\train_models.py`. |
| `GPS position unavailable` / Location permission denied | Browser or device location permissions not granted. | Allow location access in browser/device settings; ensure GPS/location is toggled ON. |
| Live buses not appearing on mobile app | Mobile app configured to point to `localhost` rather than host PC's Wi-Fi IP. | Configure `frontend/.env.production` with your computer's local LAN IP (e.g., `192.168.1.XXX:8000`). |

---

## 23. Performance and Scalability

- **In-Memory ML Inference:** Models are held in RAM, enabling predictions to complete in under 15 milliseconds.
- **WebSocket Broadcast Efficiency:** In-memory set dispatch allows broadcasting to hundreds of connected passengers with minimal overhead.
- **SQLite Concurrency Considerations:** SQLite is suitable for demonstrations and local fleets. If scaling to production with hundreds of drivers streaming GPS simultaneously, update `DATABASE_URL` to PostgreSQL to handle concurrent write transactions smoothly.
- **Smooth Frontend Animations:** Marker interpolation using `requestAnimationFrame` decouples rendering from network ping frequency, ensuring smooth visual motion.

---

## 24. Limitations and Improvement Opportunities

1. **Static Timetable Lookups:** Currently derived from historical dataset departures. Integrating an administrative timetable editing interface would allow transit managers to update schedules dynamically.
2. **Dynamic Environmental Features:** Weather and holiday inputs currently default to `"Sunny"` and `"No"`. Integrating a live weather API (e.g. OpenWeatherMap) would enrich real-time model inputs.
3. **Background Mobile Geolocation:** When the mobile screen is locked, standard web location watchers may throttle. Upgrading to a native background geolocation service plugin would ensure uninterrupted GPS broadcasting.
4. **Horizontal Scaling:** Transitioning `ConnectionManager` to a Redis Pub/Sub backend would allow the server to scale across multiple container nodes seamlessly.

---

## 25. Important Concepts and Study Notes

### 1. Two-Stage Arrival Prediction Architecture
- **Concept:** Naive systems attempt to train an ML model to predict arrival time directly from `(Current_Time, Stop)`. This fails because the model learns to output the passenger's current time rather than the bus's schedule.
- **Solution:** Stage 1 uses binary search on a fine-grained timetable to determine the bus's true **normal scheduled arrival time**. Stage 2 uses ML solely to predict **delay minutes** relative to that schedule.

### 2. Stateless Authentication with JWT
- **Concept:** JSON Web Tokens allow the backend to verify caller identity without checking a database session table on every request.
- **Mechanism:** The server signs a payload using HMAC-SHA256 and `SECRET_KEY`. Any modification to the payload invalidates the signature.

### 3. Coordinate Interpolation (Dead Reckoning)
- **Concept:** GPS pings arrive every 4 seconds. Plotting raw pings causes bus markers to jump abruptly.
- **Mechanism:** The frontend uses an ease-out mathematical curve over 2.5 seconds to glide the marker smoothly from its old coordinate to its new target coordinate.

---

## 26. Frequently Asked Questions (FAQ)

#### Q1: Why does the project have both `start-dev.bat` and `start-dev.ps1`?
**A:** Batch scripts (`.bat`) run natively in Windows Command Prompt without script execution restrictions. PowerShell scripts (`.ps1`) offer advanced process inspection and logging. Both achieve the same result.

#### Q2: Why are there two `bus_tracker.db` files in the repository?
**A:** The database at `aaaa/backend/bus_tracker.db` is the populated database containing 15 user accounts, 42 trips, and 33 live coordinate pings. The file at the root was generated when Uvicorn was executed from the root directory.

#### Q3: How do drivers and passengers access different portals?
**A:** The system enforces role-based routing. Upon logging in, drivers are routed to `/driver` and passengers are routed to `/passenger`. `ProtectedRoute.jsx` prevents cross-role access.

#### Q4: Does the machine learning model require an internet connection?
**A:** No. The scikit-learn models and timetable lookup tables are stored locally in `aaaa/backend/models/` and execute offline in memory.

---

## 27. Glossary of Technical Terms

- **ASGI (Asynchronous Server Gateway Interface):** The modern Python specification for asynchronous web servers, enabling concurrent WebSockets and HTTP requests.
- **Bcrypt:** A password-hashing function designed to protect stored credentials against brute-force attacks.
- **Capacitor:** An open-source mobile runtime created by Ionic that packages web apps into native iOS and Android binaries.
- **Dead Reckoning / Interpolation:** Estimating or smoothing an object's position between discrete GPS fixes.
- **FastAPI:** A high-speed, modern Python web framework based on standard Python type hints.
- **Leaflet:** An open-source JavaScript library for interactive mobile-friendly maps.
- **Pydantic:** A data parsing and validation library that enforces type hints in Python.
- **Random Forest Regressor:** An ensemble learning algorithm that constructs multiple decision trees during training to output accurate continuous numerical predictions.
- **Gradient Boosting Classifier:** An ML algorithm that builds trees sequentially, with each tree correcting errors made by preceding trees, used here for crowd classification.
- **WebSocket:** A computer communications protocol providing full-duplex, persistent communication channels over a single TCP connection.

---

## 28. Verification Status and Documentation Gaps

### Verified Implementations
- Full file inventory across root, backend, frontend, dataset, and android trees confirmed.
- Database schemas, table column definitions, foreign keys, and row counts verified directly via SQLite query scripts.
- Machine learning inference pipeline verified via direct Python execution: input `Bantakal` at `08:23` correctly returned next bus *Prince*, normal arrival `08:34`, predicted arrival `08:36`, delay `2.0 min`, wait `13.0 min`, crowd `High`.
- Launcher scripts, process cleaners, and environment variable files verified.

### Explicit Exclusions & Unused Files
- `aaaa/frontend/src/counter.ts`, `main.ts`, `style.css`, and Vite logo assets are remnants of initial Vite scaffolding and do not participate in the React application runtime.
- `aaaa/backend/.venv` and `aaaa/frontend/node_modules` are excluded from documentation inventory as standard third-party dependency directories.
