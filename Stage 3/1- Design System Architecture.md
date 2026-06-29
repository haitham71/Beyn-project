Add Live chat - Add Sign system api

## 1. Design System Architecture

### Project Name

**Bayn – بين**

### Purpose

The purpose of this section is to define the high-level system architecture of Bayn MVP and explain how the main components interact with each other. The architecture shows the frontend, backend, database, storage, external services, and data flow between them.

---

## High-Level Architecture Diagram

```text
                                    ┌────────────────────┐
                                    │       Users        │
                                    │ Idea Owners & Teams│
                                    └─────────┬──────────┘
                                              │
                                              │ Interact with platform
                                              ▼
                                    ┌────────────────────┐
                                    │  Frontend (React)  │
                                    │ Web & Dashboard UI │
                                    └─────────┬──────────┘
                                              │
                                              │ HTTPS API Requests
                                              ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                           Backend API (FastAPI)                              │
│                                                                              │
│ Authentication • Verification • Permissions • Projects • Contracts • Meetings│
│ Collaboration Requests • Dashboard Data • Notifications                      │
└──────┬──────────────┬──────────────┬──────────────┬──────────────┬───────────┘
       |              │              │              │              │
       |  SQL Queries │ File Uploads │ API Calls    │ API Calls    │ Email/OTP
       |              ▼              ▼              ▼              ▼
       |        ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐
       |        │Cloudflare  │ │  Cal.com   │ │  Daily.co  │ │ SMTP       │
       |        │    R2      │ │ Scheduling │ │ Meetings   │ │ Email      │
       |        └─────┬──────┘ └─────┬──────┘ └─────┬──────┘ └────────────┘
       │              │              │              │
       │ Metadata     │ Contracts    │ Meeting      │ Meeting Rooms
       │              │ Files &      │ Scheduling   │ Recordings
       │              │ Uploads      │              │ & Archive
       │              │              │              ▼
       │              │              │      ┌────────────────┐
       │              │              │      │ Cloudflare R2  │
       │              │              │      │  Recording &   │
       │              │              │      │  Archive Store │
       │              │              │      └───────┬────────┘
       │              │              │              │
       └──────────────┴──────────────┴──────────────┘
                              │
                              │ Metadata / URLs
                              ▼
                        ┌──────────────┐
                        │ PostgreSQL   │
                        │ Database     │
                        └──────────────┘
```

---

## Main System Components

| Component                     | Technology                                             | Responsibility                                                                                                                                                            |
| ----------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend                      | React                                                  | Provides the user interface, dashboard, forms, project pages, contract pages, meeting pages, and progress tracking views                                                  |
| Backend API                   | FastAPI                                                | Handles business logic, API requests, authentication, verification, permissions, projects, collaboration requests, contracts, meetings, dashboard data, and notifications |
| Database                      | PostgreSQL                                             | Stores users, ideas, collaboration requests, contracts metadata, meeting metadata, permissions, recordings metadata, and progress tracking data                           |
| Authentication & Verification | Email/Password + Email Verification + SMS Verification | Allows users to register, login, and verify their accounts using email verification and SMS verification                                                                  |
| File Storage                  | Cloudflare R2                                          | Stores uploaded files, contracts, and meeting recordings with secure access, versioning, and CDN delivery                                                                |
| Scheduling Service            | Cal.com                                                | Handles meeting scheduling, booking links, and meeting time organization                                                                                                  |
| Meeting Service               | Daily.co                                               | Creates and manages meeting rooms inside the platform based on user permissions with automatic recording integration                                                    |
| Recording Storage & Archive   | Cloudflare R2                                          | Stores and archives meeting recordings, provides recording links, metadata, and secure access control with automatic cleanup policies                                    |
| Email Service                 | Gmail SMTP                                             | Sends verification emails, OTP messages, notifications, and system emails                                                                                                 |

---

## Data Flow Explanation

### 1. User Registration and Verification Flow

The user registers through the React frontend. The frontend sends the registration data to the FastAPI backend through an HTTPS API request. The backend stores the user information in PostgreSQL and starts the verification process using email verification and SMS verification.

```text
User
  → React Frontend
  → FastAPI Backend
  → PostgreSQL Database

FastAPI Backend
  ├→ SMTP → Email Verification
  └→ SMS Verification Service → OTP Verification
```

---

### 2. Login Flow

The user enters their email and password in the React frontend. The frontend sends the credentials to the FastAPI backend. The backend validates the credentials against PostgreSQL and checks whether the user account is verified before allowing access to the platform.

```text
User
  → React Frontend
  → FastAPI Backend
  → PostgreSQL Database
  → FastAPI Backend
  → React Frontend
```

---

### 3. Project and Collaboration Flow

Users can publish ideas, search for members, send collaboration requests, and accept requests. These actions are sent from the React frontend to the FastAPI backend. The backend validates the request, checks permissions when needed, and stores the data in PostgreSQL.

```text
User
  → React Frontend
  → FastAPI Backend
  → PostgreSQL Database
```

Main actions included in this flow:

* Publish idea
* Search for members
* Send collaboration request
* Accept collaboration request
* Update project progress

---

### 4. Contracts and Files Flow

When a user uploads or creates a contract, the frontend sends the file to the FastAPI backend. The backend stores the actual contract or uploaded file in Cloudflare R2. The contract metadata, such as file name, owner, project ID, upload date, and file URL, is stored in PostgreSQL.

```text
React Frontend
  → FastAPI Backend
  ├→ Cloudflare R2
  └→ PostgreSQL Database
```

---

### 5. Meeting Scheduling Flow

Meetings are scheduled inside the platform using Cal.com. The user sends a scheduling request from the React frontend. The FastAPI backend checks the user permissions, communicates with Cal.com to schedule the meeting, and stores the meeting schedule information in PostgreSQL.

```text
React Frontend
  → FastAPI Backend
  → Permission Check
  ├→ Cal.com API
  └→ PostgreSQL Database
```

---

### 6. Meeting Room Flow

After the meeting is scheduled, the FastAPI backend creates or manages the meeting room using Daily.co. Only authorized team members can access the meeting based on their permissions.

```text
React Frontend
  → FastAPI Backend
  → Permission Check
  → Daily.co API
  → React Frontend
```

---

### 7. Meeting Recording Flow

Meeting recordings are automatically captured by Daily.co and stored in Cloudflare R2. After the recording is generated, the recording URL, metadata, and access controls are saved in PostgreSQL. The dashboard can later display the recording to authorized users with secure signed URLs.

```text
Daily.co Recording
  → Cloudflare R2
  → PostgreSQL Database (metadata & signed URLs)
  → FastAPI Backend
  → React Frontend Dashboard
```

---

### 8. Email Notification Flow

The FastAPI backend uses Gmail SMTP to send verification emails, OTP messages, meeting notifications, collaboration request updates, and system notifications.

```text
FastAPI Backend
  → Gmail SMTP
  → User Email
```

---

### 9. Dashboard and Progress Tracking Flow

The dashboard retrieves data related to projects, team members, collaboration requests, contracts, meetings, recordings, and progress tracking. The React frontend sends a request to the FastAPI backend, and the backend fetches the required data from PostgreSQL.

```text
React Frontend
  → FastAPI Backend
  → PostgreSQL Database
  → FastAPI Backend
  → React Frontend Dashboard
```

---

## Scalability and Efficiency

The architecture separates the system into independent components: frontend, backend, database, storage, scheduling, meetings, email, and recording storage. This separation makes the MVP easier to maintain, test, and scale in the future.

React is used to build a flexible and responsive user interface. FastAPI is used as a lightweight backend API for handling business logic and integrations. PostgreSQL provides reliable structured storage for users, projects, contracts, meetings, and dashboard data. Cal.com is used for scheduling meetings, while Daily.co is used for creating and managing meeting rooms. Cloudflare R2 provides scalable object storage for all user-uploaded files, contracts, and meeting recordings with CDN delivery, automatic versioning, and secure access control capabilities.
