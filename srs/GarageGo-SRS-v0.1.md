# GarageGo — Software Requirements Specification

**Document Version:** v0.1  
**Date:** 2026-05-22  
**Authors:** P. Kasturi  , K. Kajaluxmy   
**Status:** Draft — Sprint 2 Requirements Engineering  
**Project:** GarageGo – Smart Garage & Emergency Repair Platform  
**Course:** SENG 31242 System Design Project  
**Organisation:** seng31242-garagego-2026   

---

## Table of Contents

1. Introduction  
   1.1 Purpose  
   1.2 Scope  
      &emsp;1.2.1 In Scope  
      &emsp;1.2.2 Out of Scope  
   1.3 Definitions, Acronyms and Abbreviations  
   1.4 References  
   1.5 Overview   

2. Overall Description  
   2.1 Product Perspective  
   2.2 User Classes and Characteristics  
   2.3 Operating Environment  
   2.4 Design and Implementation Constraints  
   2.5 Assumptions and Dependencies  

3. System Analysis *[To be completed ]*
   
4. Functional Requirements   
   4.1 Overview   
   4.2 Functional Requirements   
   
5. Non-Functional Requirements  
   5.1 Overview    
   5.2 Performance   
   5.3 Security   
   5.4 Usability   
   5.5 Reliability   
   5.6 Scalability  
   5.7 Maintainability  

     
6. Alternative Solutions & Feasibility Study *[To be completed ]*  

---

## 1. Introduction

### 1.1 Purpose

This Software Requirements Specification (SRS) document defines the complete functional and non-functional requirements for **GarageGo**, a multi-sided digital marketplace for vehicle repair and roadside assistance in Sri Lanka. The intended audience includes the development team, project supervisors and prospective stakeholders evaluating system feasibility.

GarageGo addresses two core market failures identified through customer interviews (Issue #9) and surveys:

1. **Discovery friction for customers:** Vehicle owners struggle to find trusted, available garages quickly, especially during unexpected breakdowns. Interview participants reported relying on word-of-mouth recommendations (P01: *"I usually ask my uncle—he knows a good mechanic in Jaffna"*) and facing long, uncertain search times with no transparency on pricing or quality.

2. **Operational inefficiency for garages:** Small and medium garage owners lack digital tools for capacity management, customer acquisition and real-time dispatch. Current methods—phone calls, WhatsApp messages, and paper logbooks—lead to no-shows, empty service bays, and cash-handling risks.

GarageGo digitises the entire vehicle repair lifecycle through two core functions:

- **Scheduled Booking:** Customers search nearby garages, filter by vehicle type and services, view transparent pricing and reviews, book appointments and track service progress.
- **Emergency SOS Dispatch:** Customers trigger roadside assistance with one tap; the system broadcasts to eligible garages with emergency coverage; the first acceptance within 60 seconds wins; a mechanic is dispatched with live GPS tracking.

This document specifies all requirements necessary to guide the design, implementation, and testing of the GarageGo platform.

---

### 1.2 Scope

#### 1.2.1 In Scope

The following components and features are within the scope of this SRS:

| Component | Description |
|-----------|-------------|
| **Customer Mobile Application** | iOS and Android app (React Native/Expo) for vehicle registration, garage discovery, booking creation, emergency SOS, service history, and reviews |
| **Garage Owner Management System** | System for garage registration, service management, booking acceptance/rejection, mechanic assignment for repair and emergency requests, customer communication, invoice generation, and service tracking |
| **Mechanic Mobile Application** | iOS and Android app for emergency job reception, GPS navigation, live location sharing, status updates (arrived/in-progress/complete), and job history |
| **Admin Web Dashboard** |Interface for garage approval, user management, complaint resolution, refund triggering, and platform analytics |
| **Backend API** | Node.js/Express REST API with Socket.io real-time communication, JWT authentication, and webhook handlers |
| **Location Services** | Google Maps API for garage geocoding, distance-based search, emergency dispatch routing, and mechanic GPS tracking |
| **Notification Services** | Push notifications and SMS OTP verification |
| **File Storage** | Cloudinary or Amazon S3 for garage photos, vehicle images, invoice attachments and complaint evidence |

#### 1.2.2 Out of Scope

The following are explicitly excluded from this SRS:

| Item | Rationale |
|------|-----------|
| **Physical garage operations** | Tool inventory, equipment maintenance, and workshop floor management are internal garage processes |
| **Vehicle insurance claim processing** | Insurance workflows require regulatory partnerships beyond MVP scope |
| **Third-party spare parts procurement** | Parts ordering and supply chain integration is future enhancement |
| **Multi-country expansion** | Platform targets Sri Lanka only for MVP; internationalisation is post-MVP |
| **Offline-first architecture** | Full offline functionality is future work; MVP includes basic caching for garage list browsing only |
| **AI chatbot (symptom diagnosis)** | Not part of core MVP requirements |
| **IoT predictive maintenance** | Sensor integration and anomaly detection are roadmap features beyond current scope |
| **Online card payment processing** | GarageGo will not process card payments; all payments are cash-only at the garage. PayHere integration and webhook handling are excluded from MVP. |

---

### 1.3 Definitions, Acronyms, and Abbreviations

| Term | Definition |
|------|------------|
| **SRS** | Software Requirements Specification |
| **FR** | Functional Requirement |
| **NFR** | Non-Functional Requirement |
| **UC** | Use Case |
| **SOS** | Emergency roadside assistance request triggered by customer breakdown |
| **JWT** | JSON Web Token — signed token format for stateless authentication |
| **OTP** | One-Time Password — time-limited SMS verification code |
| **ETA** | Estimated Time of Arrival — projected mechanic arrival at breakdown location |
| **2W** | Two-wheeler vehicle (motorcycle, scooter, three-wheeler) |
| **4W** | Four-wheeler vehicle (car, van, SUV, jeep) |
| **NIC** | National Identity Card — Sri Lankan government-issued identification number |
| **GPS** | Global Positioning System — satellite-based location tracking |
| **API** | Application Programming Interface — HTTP endpoints for client-server communication |
| **Webhook** | HTTP POST callback initiated by external service to notify GarageGo of events |
| **BR** | Business Rule — constraint or policy governing system behaviour |
| **MVP** | Minimum Viable Product — the smallest feature set delivering core value |
| **Socket.io** | Real-time bidirectional event-based communication library |
| **Redis** | In-memory data structure store used for caching and session management |
| **CI/CD** | Continuous Integration / Continuous Deployment — automated build and release pipeline |
| **SLA** | Service Level Agreement — measurable performance commitment |

---

### 1.4 References

SENG 31242 Course Module Guide — System Design Project - University of Kelaniya, 2026 
SENG 31242 System Design Project Guideline & Industrial Standards - University of Kelaniya, 2026 
Customer Interview Findings (Issue #9) - `research/fact-finding/interviews/findings-summary.md` 

---

### 1.5 Overview

The remainder of this document is organised into five major sections. Section 2 describes the overall product perspective, defines three primary user personas, and establishes the operating environment, design constraints, and system dependencies. Section 3 details system features through formal use case descriptions, use case diagrams, activity diagrams, and system sequence diagrams. Section 4 enumerates functional requirements (FR-01 to FR-24+) with priority levels, source traceability, and verification methods. Section 5 specifies non-functional requirements across six categories: Performance, Security, Usability, Reliability, Scalability, and Maintainability. Section 6 presents alternative solutions analysis, a four-dimensional feasibility study (Technical, Economic, Operational, Schedule), and the justified recommendation for GarageGo's chosen architecture.

---

## 2. Overall Description

### 2.1 Product Perspective

GarageGo is a new, standalone software product. It does not replace an existing automated system but digitises a currently manual process involving phone calls, WhatsApp messages and physical visits.

The system operates as a multi-sided platform with four primary client applications and one administrative interface:

- **Customer Mobile App:** Discovery, booking, emergency SOS and review
- **Garage Owner Mobile App:** Profile management, booking handling, mechanic dispatch and invoicing  
- **Mechanic Mobile App:** Job reception, GPS tracking, status updates and navigation
- **Admin Web Dashboard:** Oversight, garage approval, complaint resolution and analytics

---

### 2.2 User Classes and Characteristics

#### Persona 1: Kasun — Customer
- **Demographics:** 28 years old, software developer, residing in Colombo
- **Vehicle:** Toyota Corolla 2018 (4W), also owns a Honda Dio scooter (2W)
- **Tech Profile:** Tech-savvy daily smartphone user. active on PickMe, UberEats, and online banking apps
- **Goals:** Find reliable garages quickly, avoid overcharging, maintain digital service history, get emergency help at night
- **Pain Points:** Price opacity ("They always say one thing and then add extra parts"), quality uncertainty, emergency vulnerability
- **Frequency:** 2–3 garage visits per year (routine service + emergency)
- **Quote (from Interview #9):** *"I want to see the price before they start working. And I want to know they are actually good—not just cheap."*

#### Persona 2: Nimal — Garage Owner
- **Demographics:** 45 years old, owns and operates a 5-bay garage in Jaffna
- **Business:** Family-run garage established 2010; 3 full-time mechanics; services both two-wheel and four-wheel vehicles
- **Tech Profile:** Moderate smartphone user. currently uses WhatsApp Business for informal bookings and a paper logbook
- **Goals:** Fill empty service bays, reduce no-shows, attract new customers beyond word-of-mouth, manage mechanic schedules digitally
- **Pain Points:** No-shows waste 2–3 hours per day, no digital presence means losing customers to competitors, cash handling creates security risks
- **Frequency:** 8–12 customers on weekdays, 4–6 on weekends
- **Needs:** Real-time capacity dashboard, verified customer base to reduce no-shows, digital payment reconciliation with automatic commission deduction

#### Persona 3: Jothini — Administrator
- **Demographics:** 23 years old, platform operations manager employed by GarageGo
- **Role:** Full-time staff. not affiliated with any garage; reports to founding team
- **Tech Profile:** Advanced user. manages multiple dashboards and analytics tools. comfortable with SQL and data exports
- **Goals:** Maintain platform quality and trust, resolve disputes fairly, onboard garages efficiently, monitor for fraud patterns
- **Pain Points:** Manual verification of garage business registration documents is time-consuming, dispute evidence is scattered across chat logs and emails
- **Frequency:** Daily platform monitoring (2 hours), weekly report generation, ad-hoc complaint resolution
- **Needs:** Automated approval workflows with document checklist, structured complaint escalation paths, analytics exports for investor reporting

---

### 2.3 Operating Environment

| Component | Requirement |
|-----------|-------------|
| **Customer Mobile App** | iOS 15.0+ or Android 10.0+ (API level 29+). GPS-enabled. 4G/LTE or WiFi |
| **Garage Owner Mobile App** | iOS 15.0+ or Android 10.0+. camera for document upload. 4G/LTE or WiFi |
| **Mechanic Mobile App** | iOS 15.0+ or Android 10.0+. GPS always-on during dispatch. 4G/LTE minimum |
| **Admin Web Dashboard** | Chrome 110+, Firefox 108+, Safari 16+. 1920×1080 resolution recommended |
| **Backend Server** | Node.js 20 LTS, PostgreSQL 15, Redis 7, Docker containers |
| **Network** | 4G/LTE or WiFi required for real-time features. offline mode limited to cached garage lists |
| **Geographic** | Sri Lanka only. GPS accuracy dependent on local cell tower density (3–10m urban, 10–50m rural) |

---

### 2.4 Design and Implementation Constraints

1. **Cash-Only Payment Model:** GarageGo will not integrate online payment gateways. All transactions are settled in cash directly between the customer and the garage. The platform records payment status manually via garage owner confirmation. This eliminates payment gateway fees, chargeback risks, and PCI-DSS compliance requirements but requires trust-based reconciliation.

2. **Map Provider Dependency:** A mapping service is required for address geocoding, distance calculations, and turn-by-turn navigation. No viable local alternative provides equivalent coverage for Sri Lankan road networks.

3. **Cross-Platform Development:** A cross-platform mobile development framework is required to maintain a single codebase for iOS and Android. Native module usage is limited to framework capabilities.

4. **API Budget Constraints:** Third-party API usage (maps, notifications) may have free tier limits. System design must respect these limits or implement usage quotas.

5. **Data Residency:** User personal data (NIC, phone numbers, location history) must remain within jurisdiction-compliant infrastructure. Regional or local cloud hosting is preferred.

---

### 2.5 Assumptions and Dependencies

1. **Smartphone Penetration:** All garage owners and mechanics possess smartphones with active data plans. This will be validated in garage owner interviews.

2. **GPS Availability:** Emergency SOS assumes GPS signal is available at the customer's location. Degraded accuracy (e.g., indoors, dense urban canyons) is handled via manual location pin confirmation.

3. **Network Connectivity:** Real-time features assume intermittent but present mobile data coverage. No offline-first architecture is planned for MVP; basic caching supports garage list browsing only.

4. **Garage Cooperation:** Garages will maintain accurate capacity counts and working hours in the app. Inaccurate data may lead to booking conflicts handled via cancellation policies.

5. **Cash Payment Trust:** Customers carry sufficient cash for garage services. Garage owners manually confirm cash receipt in the app within 24 hours of service completion. No platform-held escrow, digital wallet, or deferred payment mechanism is provided in MVP.

6. **External API Stability:** Third-party service providers (mapping, notifications, SMS) maintain their current API contracts, pricing, and service levels during the development period.

7. **User Authentication:** All users complete phone OTP verification during registration. Email verification is optional for customers but mandatory for garage owners.

---

# Section 4: Functional Requirements

## 4.1 Overview

This section defines the functional requirements for the GarageGo platform. Each requirement follows the §6.2.2 format with ID, Title, Description, Priority, Source, and Verification method. Priority levels are defined as:
- **P1** — Must have (core functionality, system cannot operate without it)
- **P2** — Should have (important but not critical for launch)
- **P3** — Nice to have (future enhancement)

---

## 4.2 Functional Requirements

### FR-01: Customer Registration

| Field | Detail |
|-------|--------|
| **ID** | FR-01 |
| **Title** | Customer Registration |
| **Description** | The system shall allow a new customer to register by providing full name, phone number, email address, NIC number, and password. The system shall verify the phone number via OTP (Twilio) and verify the email address before activating the account. |
| **Priority** | P1 |
| **Source** | Interview #9 Finding 1; UC-Registration |
| **Verification** | Integration test: register a new user with valid credentials, verify OTP is sent via Twilio, confirm email verification link is sent, confirm account is activated after both verifications. |

---

### FR-02: Vehicle Registration

| Field | Detail |
|-------|--------|
| **ID** | FR-02 |
| **Title** | Vehicle Registration |
| **Description** | The system shall allow a registered customer to add one or more vehicles to their account. Each vehicle record shall include vehicle type (two-wheel, four-wheel, or heavy), brand, model, year, fuel type, transmission type, registration number, and optional notes. |
| **Priority** | P1 |
| **Source** | Interview #9 Finding 1; UC-02 |
| **Verification** | Functional test: log in as customer, add a vehicle with all required fields, verify vehicle appears in the customer's vehicle list with correct details. |

---

### FR-03: Nearby Garage Search

| Field | Detail |
|-------|--------|
| **ID** | FR-03 |
| **Title** | Nearby Garage Search |
| **Description** | The system shall display nearby approved garages on a map and in a list based on the customer's current GPS location. The customer shall be able to filter results by vehicle type, services offered, ratings, operating hours, and current availability. |
| **Priority** | P1 |
| **Source** | Survey #13 Q3; UC-02 |
| **Verification** | Functional test: enable location, open garage search, verify garages within the defined radius are displayed on map and list. Apply each filter and verify results update correctly. |

---

### FR-04: Booking Creation

| Field | Detail |
|-------|--------|
| **ID** | FR-04 |
| **Title** | Booking Creation |
| **Description** | The system shall allow a customer to create a booking by selecting a vehicle, choosing a garage, selecting a service type, choosing a date and time, and adding optional notes. The system shall check garage capacity before confirming the booking slot. |
| **Priority** | P1 |
| **Source** | UC-02; Interview #9 Finding 2 |
| **Verification** | Functional test: create a booking with valid inputs, verify booking is stored with status "pending", verify garage receives a push notification. Attempt booking when no slots are available and verify HTTP 409 is returned. |

---

### FR-05: Emergency SOS Dispatch

| Field | Detail |
|-------|--------|
| **ID** | FR-05 |
| **Title** | Emergency SOS Dispatch |
| **Description** | The system shall allow a customer to trigger an emergency SOS request by selecting their affected vehicle, describing the issue, and sharing their GPS location. The system shall identify all eligible nearby garages filtered by zone and capacity and broadcast a live alert to all of them simultaneously via Socket.io. |
| **Priority** | P1 |
| **Source** | Interview #9 Finding Q8; UC-01 |
| **Verification** | Integration test: trigger SOS from customer app, verify emergency_request is inserted in DB with status "searching", verify all eligible garages receive the broadcast within 2 seconds. |

---

### FR-06: First-Accept-Wins Emergency Assignment

| Field | Detail |
|-------|--------|
| **ID** | FR-06 |
| **Title** | First-Accept-Wins Emergency Assignment |
| **Description** | The system shall assign an emergency request to the first garage that accepts it. The assignment shall use a SELECT FOR UPDATE database lock to prevent race conditions when multiple garages accept simultaneously. All other garages shall be notified that the emergency has already been assigned. |
| **Priority** | P1 |
| **Source** | UC-01; UC-03 |
| **Verification** | Concurrency test: simulate two garages accepting the same emergency simultaneously, verify only one is assigned and the DB lock prevents duplicate assignments. |

---

### FR-07: Mechanic GPS Tracking

| Field | Detail |
|-------|--------|
| **ID** | FR-07 |
| **Title** | Mechanic GPS Tracking |
| **Description** | The system shall allow the customer to track the assigned mechanic's real-time GPS location on a map during an active emergency. The mechanic app shall transmit location coordinates every 5 seconds via Socket.io. The system shall display the mechanic's estimated time of arrival and update it continuously. |
| **Priority** | P1 |
| **Source** | UC-06; Interview #9 Finding 6 |
| **Verification** | Integration test: start an emergency, have mechanic app emit location every 5 seconds, verify customer app receives location updates and ETA is displayed correctly. Simulate signal loss for 30 seconds and verify "Tracking paused" is shown. |

---

### FR-08: Garage Capacity Management

| Field | Detail |
|-------|--------|
| **ID** | FR-08 |
| **Title** | Garage Capacity Management |
| **Description** | The system shall allow a garage owner to set the maximum number of vehicles they can service simultaneously for each vehicle category (two-wheel, four-wheel, heavy). The system shall enforce this limit when accepting bookings and emergency requests. |
| **Priority** | P1 |
| **Source** | UC-02; UC-03 |
| **Verification** | Functional test: set capacity to 1 for four-wheel vehicles, create a booking that fills the slot, attempt a second booking for the same time slot and verify it is rejected with HTTP 409. |

---

### FR-09: Booking State Machine

| Field | Detail |
|-------|--------|
| **ID** | FR-09 |
| **Title** | Booking State Machine |
| **Description** | The system shall manage booking status transitions in the following order: pending → accepted → in_service → completed → paid. The system shall also support the cancelled state reachable from pending or accepted. Each transition shall trigger appropriate notifications to the customer and garage. |
| **Priority** | P1 |
| **Source** | UC-02; UC-05 |
| **Verification** | State machine test: walk a booking through each valid state transition and verify the DB status updates correctly. Attempt invalid transitions and verify they are rejected. |

---

### FR-10: Garage Registration

| Field | Detail |
|-------|--------|
| **ID** | FR-10 |
| **Title** | Garage Registration |
| **Description** | The system shall allow a garage owner to register their garage by providing business name, address, registration number, phone number, working hours, accepted vehicle types, services offered, location coordinates, emergency service zone, service capacity, and photos. The registration shall be submitted to an administrator for approval before the garage becomes visible to customers. |
| **Priority** | P1 |
| **Source** | Interview #9 Finding 1; UC-04 |
| **Verification** | Functional test: submit garage registration, verify status is "pending" and not visible to customers. Admin approves the garage, verify status changes to "approved" and garage appears in customer search results. |

---

### FR-11: Admin Garage Approval and Suspension

| Field | Detail |
|-------|--------|
| **ID** | FR-11 |
| **Title** | Admin Garage Approval and Suspension |
| **Description** | The system shall provide the administrator with the ability to review pending garage registrations and approve or reject them. The administrator shall also be able to suspend an approved garage if fraudulent activity or policy violations are detected. Suspended garages shall not appear in search results or receive new bookings. |
| **Priority** | P1 |
| **Source** | UC-07 |
| **Verification** | Functional test: admin approves a pending garage and verifies it becomes visible. Admin suspends an approved garage and verifies it disappears from search and cannot accept new bookings. |

---

### FR-12: Service History

| Field | Detail |
|-------|--------|
| **ID** | FR-12 |
| **Title** | Service History |
| **Description** | The system shall maintain a complete service history for each registered vehicle. Each service record shall include the garage name, date of service, services performed, invoice details, and payment status. The customer shall be able to view the full history per vehicle from within the app. |
| **Priority** | P1 |
| **Source** | Interview #9 Finding Q5; UC-02 |
| **Verification** | Functional test: complete a service booking, verify the service record is stored against the correct vehicle, open vehicle service history and confirm all fields are displayed correctly. |

---

### FR-13: Rating and Review

| Field | Detail |
|-------|--------|
| **ID** | FR-13 |
| **Title** | Rating and Review |
| **Description** | The system shall prompt the customer to submit a rating (1–5 stars) and an optional written review after each completed service. Reviews shall be displayed on the garage profile. A garage's average rating shall be calculated from all submitted reviews and updated in real time. |
| **Priority** | P2 |
| **Source** | Survey #13 Q10; UC-02 |
| **Verification** | Functional test: complete a booking, submit a rating and review, verify the review appears on the garage profile and the average rating is recalculated correctly. |

---

### FR-14: Push Notifications

| Field | Detail |
|-------|--------|
| **ID** | FR-14 |
| **Title** | Push Notifications |
| **Description** | The system shall send push notifications to the relevant app via the External Messaging System for the following events: new booking request (garage), booking accepted or rejected (customer), emergency broadcast (garage), emergency assigned (customer), mechanic arrived (customer), and booking status updates. |
| **Priority** | P1 |
| **Source** | Interview #9 Finding 1; Survey #13 Q9 |
| **Verification** | Integration test: trigger each notification event and verify the correct push notification is received by the correct recipient within 5 seconds. |

---

### FR-15: Mechanic Assignment and Dispatch

| Field | Detail |
|-------|--------|
| **ID** | FR-15 |
| **Title** | Mechanic Assignment and Dispatch |
| **Description** | The system shall allow the garage owner to assign a mechanic to an accepted emergency request. The assigned mechanic shall receive the job details in the mechanic app including the customer's GPS location and vehicle information. The mechanic shall be able to open navigation in Google Maps or Waze directly from the app. |
| **Priority** | P1 |
| **Source** | UC-03; Interview #9 Finding 6 |
| **Verification** | Functional test: accept an emergency, assign a mechanic, verify the mechanic app displays the job and customer location, verify the navigation link opens correctly. |

---

### FR-16: Complaint Submission

| Field | Detail |
|-------|--------|
| **ID** | FR-16 |
| **Title** | Complaint Submission |
| **Description** | The system shall allow customers to submit complaints against a garage for issues including overcharging, poor workmanship, no-shows, or unprofessional behaviour. The complaint shall be routed to the administrator dashboard for review and resolution. |
| **Priority** | P2 |
| **Source** | Survey #13; UC-07 |
| **Verification** | Functional test: submit a complaint from the customer app, verify it appears in the admin dashboard with correct details and status "open". |

---

# Section 5: Non-Functional Requirements

## 5.1 Overview

This section defines the non-functional requirements (NFRs) for the GarageGo platform across six categories: Performance, Security, Usability, Reliability, Scalability, and Maintainability. All NFRs are measurable with specific thresholds.

---

## 5.2 Performance

### NFR-P1: API Response Time

| Field | Detail |
|-------|--------|
| **ID** | NFR-P1 |
| **Title** | API Response Time |
| **Requirement** | The system shall respond to 95% of REST API requests within 2 seconds under a load of 50 concurrent users. |
| **Threshold** | ≤ 2 seconds for 95th percentile response time at 50 concurrent users |
| **Verification** | Load test using Apache JMeter with 50 virtual users, measure p95 response time across all major endpoints. |

---

### NFR-P2: Emergency Broadcast Latency

| Field | Detail |
|-------|--------|
| **ID** | NFR-P2 |
| **Title** | Emergency Broadcast Latency |
| **Requirement** | The system shall deliver an emergency SOS broadcast to all eligible garages within 2 seconds of the customer triggering the SOS. |
| **Threshold** | ≤ 2 seconds from POST /emergency to Socket.io delivery at all eligible garage clients |
| **Verification** | Integration test: measure time from SOS trigger to Socket.io event receipt at garage app using timestamped logs. |

---

### NFR-P3: Real-Time Location Update Frequency

| Field | Detail |
|-------|--------|
| **ID** | NFR-P3 |
| **Title** | Real-Time Location Update Frequency |
| **Requirement** | The system shall relay mechanic GPS location updates to the customer within 1 second of receiving the update from the mechanic app, maintaining a maximum update interval of 6 seconds end-to-end. |
| **Threshold** | ≤ 1 second relay delay; ≤ 6 seconds total end-to-end interval |
| **Verification** | Integration test: measure timestamp difference between mechanic emit and customer receive events during active tracking. |

---

## 5.3 Security

### NFR-S1: JWT Authentication

| Field | Detail |
|-------|--------|
| **ID** | NFR-S1 |
| **Title** | JWT Authentication |
| **Requirement** | All API endpoints except /auth/register and /auth/login shall require a valid JSON Web Token. Tokens shall expire after 15 minutes. Refresh tokens shall expire after 7 days. |
| **Threshold** | 100% of protected endpoints return HTTP 401 for missing or expired tokens |
| **Verification** | Security test: call each protected endpoint without a token and with an expired token, verify HTTP 401 is returned in all cases. |

---

### NFR-S2: Password Hashing

| Field | Detail |
|-------|--------|
| **ID** | NFR-S2 |
| **Title** | Password Hashing |
| **Requirement** | All user passwords shall be hashed using bcrypt with a minimum cost factor of 12 rounds before storage. Plain-text passwords shall never be stored or logged. |
| **Threshold** | bcrypt cost factor ≥ 12; zero plain-text password occurrences in DB or logs |
| **Verification** | Code review: verify bcrypt configuration. DB audit: confirm all password values are bcrypt hashes. Log scan: verify no plain-text passwords appear in application logs. |

---

### NFR-S3: Data Encryption in Transit

| Field | Detail |
|-------|--------|
| **ID** | NFR-S3 |
| **Title** | Data Encryption in Transit |
| **Requirement** | All communication between client apps and the API server shall use HTTPS with TLS 1.2 or higher. WebSocket connections shall use WSS protocol. HTTP requests shall be automatically redirected to HTTPS. |
| **Threshold** | 100% of traffic over TLS 1.2+; zero unencrypted connections accepted |
| **Verification** | Security scan using SSL Labs, verify TLS version and certificate validity. Attempt plain HTTP connection and verify redirect to HTTPS. |

---

## 5.4 Usability

### NFR-U1: SOS Button Accessibility

| Field | Detail |
|-------|--------|
| **ID** | NFR-U1 |
| **Title** | SOS Button Accessibility |
| **Requirement** | The Emergency SOS button shall be visible on the customer app home screen without any scrolling. The button shall have a minimum tap target size of 48x48 dp as per Material Design guidelines. |
| **Threshold** | SOS button visible without scroll on all supported screen sizes; tap target ≥ 48x48 dp |
| **Verification** | UI test: open the app on the smallest supported screen size and verify the SOS button is visible without scrolling. Measure tap target dimensions. |

---

### NFR-U2: Registration Completion Time

| Field | Detail |
|-------|--------|
| **ID** | NFR-U2 |
| **Title** | Registration Completion Time |
| **Requirement** | A new customer shall be able to complete the full registration process including entering details, verifying phone OTP, and verifying email within 3 minutes under normal network conditions. |
| **Threshold** | ≤ 3 minutes from app open to account activated |
| **Verification** | Usability test: time 5 new users completing registration on a standard 4G connection, verify average time is under 3 minutes. |

---

### NFR-U3: Offline Error Handling

| Field | Detail |
|-------|--------|
| **ID** | NFR-U3 |
| **Title** | Offline Error Handling |
| **Requirement** | The system shall display a clear user-friendly error message within 3 seconds when the customer app loses internet connectivity. The message shall not display raw error codes or stack traces. |
| **Threshold** | Error message displayed ≤ 3 seconds after connectivity loss; zero technical error codes shown to user |
| **Verification** | Functional test: disable network on device mid-session, verify user-friendly message appears within 3 seconds. |

---

## 5.5 Reliability

### NFR-R1: System Uptime

| Field | Detail |
|-------|--------|
| **ID** | NFR-R1 |
| **Title** | System Uptime |
| **Requirement** | The GarageGo API server shall maintain a minimum uptime of 99.5% during peak hours (7:00 AM – 9:00 PM Sri Lanka Standard Time), measured monthly. |
| **Threshold** | ≥ 99.5% uptime during peak hours per month |
| **Verification** | Monitor uptime using Prometheus and Grafana over a 30-day period, verify monthly uptime report meets threshold. |

---

### NFR-R2: Webhook Failure Recovery

| Field | Detail |
|-------|--------|
| **ID** | NFR-R2 |
| **Title** | Webhook Failure Recovery |
| **Requirement** | If a webhook from the External Messaging System fails to arrive, the system shall trigger a reconciliation polling job within 10 minutes to verify and update the relevant booking or notification status. |
| **Threshold** | Reconciliation job triggered ≤ 10 minutes after missed webhook |
| **Verification** | Integration test: simulate a failed webhook delivery, verify reconciliation job runs within 10 minutes and status is corrected. |

---

### NFR-R3: Data Backup

| Field | Detail |
|-------|--------|
| **ID** | NFR-R3 |
| **Title** | Data Backup |
| **Requirement** | The PostgreSQL database shall be backed up automatically every 24 hours. Backups shall be retained for a minimum of 30 days. The system shall be recoverable from a backup within 2 hours in the event of data loss. |
| **Threshold** | Daily backups; 30-day retention; recovery time ≤ 2 hours |
| **Verification** | Operations test: verify automated backup runs daily, perform a restore from backup and measure recovery time. |

---

## 5.6 Scalability

### NFR-SC1: Concurrent Socket.io Connections

| Field | Detail |
|-------|--------|
| **ID** | NFR-SC1 |
| **Title** | Concurrent Socket.io Connections |
| **Requirement** | The Socket.io server shall support a minimum of 500 concurrent connections without degradation in message delivery time exceeding 1 second. |
| **Threshold** | ≥ 500 concurrent connections; message delivery ≤ 1 second at full load |
| **Verification** | Load test: simulate 500 concurrent Socket.io clients, measure message delivery latency and verify it stays under 1 second. |

---

### NFR-SC2: Database Partitioning

| Field | Detail |
|-------|--------|
| **ID** | NFR-SC2 |
| **Title** | Database Partitioning |
| **Requirement** | The PostgreSQL bookings table shall support monthly horizontal partitioning to manage data growth. Queries on the current month's data shall return results within 500 milliseconds when the table contains over 100,000 records. |
| **Threshold** | Monthly partition support; query response ≤ 500ms at 100,000+ records |
| **Verification** | Performance test: insert 100,000 booking records, run common queries on current month partition and measure response time. |

---

### NFR-SC3: Horizontal API Scaling

| Field | Detail |
|-------|--------|
| **ID** | NFR-SC3 |
| **Title** | Horizontal API Scaling |
| **Requirement** | The GarageGo API server shall be deployable as multiple instances behind a load balancer. Adding a second instance shall increase throughput by at least 80% compared to a single instance under equivalent load. |
| **Threshold** | ≥ 80% throughput increase when scaling from 1 to 2 instances |
| **Verification** | Load test: measure throughput with 1 instance, add a second instance under same load and verify throughput increase meets threshold. |

---

## 5.7 Maintainability

### NFR-M1: Unit Test Coverage

| Field | Detail |
|-------|--------|
| **ID** | NFR-M1 |
| **Title** | Unit Test Coverage |
| **Requirement** | The GarageGo backend codebase shall maintain a minimum unit test coverage of 70%, measured using Jest. Coverage reports shall be generated automatically as part of the CI/CD pipeline on every pull request. |
| **Threshold** | ≥ 70% line coverage via Jest; coverage report generated on every PR |
| **Verification** | Run Jest with coverage flag in CI pipeline, verify coverage report shows ≥ 70% and pipeline fails if threshold is not met. |

---

### NFR-M2: Container Redeployment Time

| Field | Detail |
|-------|--------|
| **ID** | NFR-M2 |
| **Title** | Container Redeployment Time |
| **Requirement** | Each GarageGo system component (API server, Socket.io server, admin dashboard) shall be independently deployable via Docker. Redeploying any single component shall complete within 5 minutes without affecting other running components. |
| **Threshold** | Redeployment time ≤ 5 minutes per component; zero downtime for other components |
| **Verification** | Operations test: redeploy each Docker container independently, measure time from deployment start to healthy state, verify other components remain operational throughout. |

---

### NFR-M3: Code Documentation

| Field | Detail |
|-------|--------|
| **ID** | NFR-M3 |
| **Title** | Code Documentation |
| **Requirement** | All public API endpoints shall be documented using OpenAPI 3.0 specification. The API documentation shall be auto-generated and accessible at /api/docs when the server is running in development mode. |
| **Threshold** | 100% of public endpoints documented in OpenAPI 3.0; /api/docs accessible in development |
| **Verification** | Code review: verify OpenAPI spec covers all endpoints. Start server in dev mode and confirm /api/docs loads with correct endpoint definitions. |
