# SENG 31242
# System Design Specification (SDS)

# GarageGo
## Smart Garage & Emergency Vehicle Repair Platform

### By

- P. Kasturi – SE/2022/053
- K. Kajaluxmy – SE/2022/056
- V. Vanushan – SE/2022/031
- S. Krishnapiriyan – SE/2022/048

---

A report submitted in partial fulfilment of the requirements for the degree of

**Bachelor of Science Honours in Software Engineering (B.Sc.SE)**

**Software Engineering Teaching Unit**  
Faculty of Science  
University of Kelaniya  
Sri Lanka

**2026**

---

# Abstract

This System Design Specification (SDS) document describes the complete technical design of the GarageGo platform, a multi-sided mobile and web application for vehicle repair, scheduled garage booking, and emergency roadside assistance in Sri Lanka.

The document is derived directly from the approved GarageGo Software Requirements Specification (SRS v0.1) and covers the high-level architectural design, Architectural Decision Records (ADRs), database schema with Entity-Relationship diagram, UML class diagram, sequence diagrams for all ten use cases, state machine diagrams for stateful entities, user interface wireframes and navigation flow, security design, external and internal interface specifications, and non-functional design decisions addressing all fourteen NFRs defined in the SRS.

The system is designed using a Layered plus Event-Driven hybrid architecture with a Node.js/Express REST API backend, Socket.io for real-time bidirectional communication, React Native (Expo) for cross-platform mobile applications, React.js for the Admin Web Dashboard, and PostgreSQL 16 as the primary relational data store with Redis 7 as the caching and session layer. All components are containerised using Docker and deployable behind a load balancer for horizontal scaling. External integrations include Google Maps Platform for geocoding, routing, and GPS tracking, Twilio for SMS OTP verification, and Firebase Cloud Messaging for push notifications.

**Keywords:** System Design, Layered Architecture, Event-Driven Architecture, Node.js, React Native, PostgreSQL, Redis, Socket.io, JWT Authentication, Docker, Google Maps, Emergency SOS Dispatch, UML, ER Diagram, OWASP.

---

# Definitions, Acronyms, and Abbreviations

**Table 1: Definitions, Acronyms, and Abbreviations**

| Term / Acronym | Definition / Description |
|----------------|--------------------------|
| SDS | System Design Specification |
| SRS | Software Requirements Specification |
| API | Application Programming Interface |
| REST | Representational State Transfer |
| JWT | JSON Web Token – stateless authentication token |
| OTP | One-Time Password – time-limited SMS verification code |
| GPS | Global Positioning System – satellite-based location tracking |
| SOS | Emergency roadside assistance request triggered by a customer breakdown |
| ETA | Estimated Time of Arrival – projected mechanic arrival time |
| FCM | Firebase Cloud Messaging – push notification service |
| Socket.io | Real-time bidirectional event-based communication library |
| Redis | In-memory data structure store for caching, sessions, and queues |
| MVC | Model-View-Controller architectural pattern |
| ADR | Architectural Decision Record |
| ER | Entity-Relationship Diagram |
| UML | Unified Modelling Language |
| RBAC | Role-Based Access Control |
| TLS | Transport Layer Security |
| OWASP | Open Web Application Security Project |
| CI/CD | Continuous Integration / Continuous Deployment |
| NFR | Non-Functional Requirement |
| FR | Functional Requirement |
| UC | Use Case |
| 2W | Two-wheeler vehicle (motorcycle, scooter, three-wheeler) |
| 4W | Four-wheeler vehicle (car, van, SUV, jeep) |
| NIC | National Identity Card – Sri Lankan government-issued identification |
| MVP | Minimum Viable Product |
| DXA | Document Exchange Absolute unit (used in Word formatting) |

---

# Chapter 1: Architectural Design

This chapter presents the high-level system architecture of GarageGo, justification for the chosen design pattern using Architectural Decision Records, component and deployment diagrams, and the technology stack with rationale. All design decisions are directly traceable to the functional and non-functional requirements documented in the approved GarageGo SRS v0.1.

## 1.1 High-Level Architecture

GarageGo adopts a **Layered + Event-Driven hybrid architecture**.

The layered component provides clear separation of concerns between the presentation, API, business logic, data access, and persistence tiers.

The event-driven component, implemented via Socket.io over WebSocket, handles all real-time interactions:

- Emergency SOS broadcasting
- First-accept-wins assignment
- Mechanic GPS tracking

This hybrid approach was selected because GarageGo's core value proposition — real-time emergency dispatch and live GPS tracking — cannot be delivered by a purely request-response REST architecture alone, while a fully event-driven architecture would complicate the structured booking and administrative workflows that are well-served by REST.

![GarageGo High-Level Architecture](../diagrams/architecture/component-diagram.png)

**Figure 1: GarageGo – High-Level System Architecture Diagram**

### Table 2: Architectural Layers and Responsibilities

| Layer | Technology / Component | Responsibility |
|---------|----------------------|---------------|
| Presentation | React Native (Expo) – Customer, Garage, Mechanic Apps; React.js – Admin Dashboard | Renders mobile and web UIs; handles user interactions; communicates with the API over HTTPS and WSS |
| API Gateway | Node.js/Express, JWT Middleware, TLS 1.2+ | Authenticates JWT tokens; enforces RBAC; validates request schema; routes to service handlers |
| Real-Time Event Bus | Socket.io (WebSocket over WSS) | Broadcasts emergency SOS alerts; streams mechanic GPS updates every 5 seconds; delivers booking status events |
| Business Logic | Express Route Handlers, EmergencyService, BookingService, GarageService, NotificationService | Implements business rules and manages workflow transitions |
| Cache / Session | Redis 7 | Stores active Socket.io sessions; caches garage lists; manages OTP expiry TTLs; queues reconciliation jobs |
| Data Access | node-postgres (pg), Sequelize ORM | Abstracts PostgreSQL queries; manages connection pooling; enforces parameterised queries |
| Database | PostgreSQL 15 (partitioned bookings table) | Persists all system data; provides ACID transactions; supports monthly horizontal partitioning |
| External Services | Google Maps API, Twilio (SMS OTP), Firebase Cloud Messaging | Geocoding, routing, phone verification, and push notification delivery |

---

## 1.2 Architectural Decision Records

All major design decisions are documented as Architectural Decision Records (ADRs) per Section 7.2 of the course guidelines.

Each ADR follows the template:

- Context
- Decision
- Rationale
- Alternatives Considered
- Consequences

### ADR-01: Architectural Pattern Selection

| Field | Content |
|---------|---------|
| Date | 2026-06-12 |
| Status | Accepted |
| Context | GarageGo requires both structured request-response workflows and real-time bidirectional communication |
| Decision | Adopt a Layered + Event-Driven hybrid architecture. REST over HTTPS for transactional endpoints and Socket.io over WSS for real-time communication |
| Rationale | Supports scalability, emergency dispatch, and live GPS tracking requirements |
| Alternatives Considered | Microservices, Pure REST Polling, Full Event-Driven Architecture |
| Consequences | Redis is required for session sharing and event synchronization |

### ADR-02: Frontend Framework – React Native vs Native

| Field | Content |
|---------|---------|
| Date | 2026-06-12 |
| Status | Accepted |
| Context | Three mobile applications must support both Android and iOS |
| Decision | React Native with Expo managed workflow |
| Rationale | Single shared codebase significantly reduces development effort |
| Alternatives Considered | Native Android/iOS, Flutter, Progressive Web App |
| Consequences | Native module access depends on Expo SDK capabilities |

### ADR-03: Database Selection – PostgreSQL vs NoSQL

| Field | Content |
|---------|---------|
| Date | 2026-06-12 |
| Status | Accepted |
| Context | The system contains highly relational data and requires ACID transactions |
| Decision | PostgreSQL 15 with monthly table partitioning |
| Rationale | Provides transaction consistency, referential integrity, and partitioning support |
| Alternatives Considered | MongoDB, MySQL, Firebase Firestore |
| Consequences | Schema migration and partition management must be maintained |

---

## 1.3 Scalability and Maintainability Considerations

- Horizontal scaling is achieved by deploying multiple Node.js/Express API containers behind a load balancer.
- Redis stores Socket.io session state shared across all backend instances.
- Stateless REST APIs allow any instance to process any request without sticky sessions.
- PostgreSQL booking tables are partitioned monthly to maintain query performance.
- API Server, Socket.io Server, and Admin Dashboard can be deployed independently as separate Docker containers.
- Containerised deployment enables rapid updates, rollback, and environment consistency.

# Chapter 2: Database Design

This chapter presents the Entity-Relationship (ER) diagram, entity descriptions, key relationships, and normalization analysis for the GarageGo database schema. The schema is implemented using PostgreSQL. All primary keys use UUIDs to ensure globally unique identifiers.

## 2.1 Entity-Relationship (ER) Diagram

The ER diagram illustrates the relationships between the main entities in the GarageGo platform, including customers, vehicles, garages, bookings, emergency requests, mechanics, reviews, notifications, and administrators.

**Figure 2:** GarageGo – Entity-Relationship Diagram  
`[see documents/diagrams/database/ER-Diagram.png]`

---

## 2.2 Entity Descriptions and Attributes

### Customer

Stores information about registered customers.

| Attribute | Data Type | Description |
|------------|-----------|-------------|
| customerId (PK) | UUID | Unique customer identifier |
| fullName | VARCHAR | Customer name |
| phone | VARCHAR | Contact number |
| email | VARCHAR | Email address |
| passwordHash | VARCHAR | Encrypted password |
| isActive | BOOLEAN | Account status |
| createdAt | TIMESTAMP | Registration date |

---

### Vehicle

Stores customer vehicle information.

| Attribute | Data Type | Description |
|------------|-----------|-------------|
| vehicleId (PK) | UUID | Unique vehicle identifier |
| customerId (FK) | UUID | Associated customer |
| vehicleType | ENUM | Vehicle category |
| brand | VARCHAR | Vehicle brand |
| model | VARCHAR | Vehicle model |
| year | SMALLINT | Manufacturing year |
| fuelType | ENUM | Fuel type |
| transmissionType | ENUM | Transmission type |
| registrationNo | VARCHAR | Registration number |
| notes | TEXT | Additional information |

---

### Garage

Stores information about registered garages.

| Attribute | Data Type | Description |
|------------|-----------|-------------|
| garageId (PK) | UUID | Unique garage identifier |
| ownerAccountId (FK) | UUID | Garage owner reference |
| businessName | VARCHAR | Garage name |
| registrationNo | VARCHAR | Business registration number |
| address | VARCHAR | Garage location |
| latitude | DECIMAL | Latitude coordinate |
| longitude | DECIMAL | Longitude coordinate |
| emergencyZoneRadius | INTEGER | Emergency service radius |
| status | ENUM | Approval status |
| averageRating | DECIMAL | Average customer rating |
| workingHours | JSONB | Operating hours |
| acceptedVehicleTypes | TEXT[] | Supported vehicle categories |
| capacity2W | SMALLINT | Two-wheeler capacity |
| capacity4W | SMALLINT | Four-wheeler capacity |
| capacityHeavy | SMALLINT | Heavy vehicle capacity |
| hasEmergencyService | BOOLEAN | Emergency service availability |

---

### GarageOwner

Stores garage owner account details.

| Attribute | Data Type | Description |
|------------|-----------|-------------|
| ownerId (PK) | UUID | Unique owner identifier |
| email | VARCHAR | Login email |
| phone | VARCHAR | Contact number |
| passwordHash | VARCHAR | Encrypted password |
| isVerified | BOOLEAN | Verification status |

---

### Booking

Stores scheduled service bookings.

| Attribute | Data Type | Description |
|------------|-----------|-------------|
| bookingId (PK) | UUID | Unique booking identifier |
| customerId (FK) | UUID | Customer reference |
| vehicleId (FK) | UUID | Vehicle reference |
| garageId (FK) | UUID | Garage reference |
| serviceType | VARCHAR | Requested service |
| scheduledAt | TIMESTAMP | Appointment date and time |
| status | ENUM | Booking status |
| estimatedCost | DECIMAL | Estimated cost |
| finalCost | DECIMAL | Final cost |
| paymentStatus | ENUM | Payment state |
| cancellationFee | DECIMAL | Cancellation charge |
| notes | TEXT | Additional notes |
| createdAt | TIMESTAMP | Creation timestamp |
| updatedAt | TIMESTAMP | Last update timestamp |

---

### EmergencyRequest

Stores roadside emergency assistance requests.

| Attribute | Data Type | Description |
|------------|-----------|-------------|
| emergencyId (PK) | UUID | Unique request identifier |
| customerId (FK) | UUID | Customer reference |
| vehicleId (FK) | UUID | Vehicle reference |
| assignedGarageId (FK) | UUID | Assigned garage |
| assignedMechanicId (FK) | UUID | Assigned mechanic |
| customerLatitude | DECIMAL | Customer latitude |
| customerLongitude | DECIMAL | Customer longitude |
| issueDescription | TEXT | Reported issue |
| status | ENUM | Emergency status |
| searchRadiusKm | INTEGER | Search radius |
| triggeredAt | TIMESTAMP | Request creation time |
| assignedAt | TIMESTAMP | Assignment time |
| completedAt | TIMESTAMP | Completion time |

---

### Mechanic

Stores mechanic information.

| Attribute | Data Type | Description |
|------------|-----------|-------------|
| mechanicId (PK) | UUID | Unique mechanic identifier |
| garageId (FK) | UUID | Associated garage |
| fullName | VARCHAR | Mechanic name |
| phone | VARCHAR | Contact number |
| passwordHash | VARCHAR | Encrypted password |
| isAvailable | BOOLEAN | Availability status |
| currentLatitude | DECIMAL | Current latitude |
| currentLongitude | DECIMAL | Current longitude |
| lastLocationAt | TIMESTAMP | Last location update |

---

### Review

Stores customer ratings and reviews.

| Attribute | Data Type | Description |
|------------|-----------|-------------|
| reviewId (PK) | UUID | Unique review identifier |
| bookingId (FK) | UUID | Booking reference |
| customerId (FK) | UUID | Customer reference |
| garageId (FK) | UUID | Garage reference |
| rating | SMALLINT | Rating value |
| reviewText | TEXT | Review content |
| createdAt | TIMESTAMP | Submission date |

---

### Notification

Stores system-generated notifications.

| Attribute | Data Type | Description |
|------------|-----------|-------------|
| notificationId (PK) | UUID | Unique notification identifier |
| recipientId | UUID | Recipient reference |
| recipientRole | ENUM | Recipient role |
| type | ENUM | Notification type |
| payload | JSONB | Notification content |
| isDelivered | BOOLEAN | Delivery status |
| sentAt | TIMESTAMP | Dispatch time |

---

### Admin

Stores administrator account information.

| Attribute | Data Type | Description |
|------------|-----------|-------------|
| adminId (PK) | UUID | Unique administrator identifier |
| email | VARCHAR | Administrator email |
| role | ENUM | System role |

---

## 2.3 Key Relationships

- A Customer can own multiple Vehicles.
- A Customer can create multiple Bookings.
- A Customer can submit multiple Emergency Requests.
- A Garage can receive multiple Bookings.
- A Garage can handle multiple Emergency Requests.
- A Garage can employ multiple Mechanics.
- A Garage Owner can manage one or more Garages.
- A Booking is associated with one Customer, one Vehicle, and one Garage.
- An Emergency Request may be assigned to one Garage and one Mechanic.
- A completed Booking may have one Review.
- A Garage can receive multiple Reviews.
- Users can receive multiple Notifications.
- Administrators manage platform operations and approvals.

---

## 2.4 Normalization

The GarageGo database schema is designed to minimize redundancy and maintain data integrity.

| Entity | 1NF | 2NF | 3NF | BCNF |
|----------|-----|-----|-----|------|
| Customer | Yes | Yes | Yes | Yes |
| Vehicle | Yes | Yes | Yes | Yes |
| Garage | Yes | Yes | Yes | Yes |
| GarageOwner | Yes | Yes | Yes | Yes |
| Booking | Yes | Yes | Yes | Yes |
| EmergencyRequest | Yes | Yes | Yes | Yes |
| Mechanic | Yes | Yes | Yes | Yes |
| Review | Yes | Yes | Yes | Yes |
| Notification | Yes | Yes | Yes | Yes |
| Admin | Yes | Yes | Yes | Yes |

All entities satisfy First Normal Form (1NF), Second Normal Form (2NF), Third Normal Form (3NF), and Boyce-Codd Normal Form (BCNF), ensuring efficient storage, reduced redundancy, and consistent relational integrity.

# Chapter 3: Class Diagram

This chapter describes the GarageGo domain class model, enumerations, and UML relationships. The class diagram is derived from the system requirements and use cases defined in the Software Requirements Specification (SRS).

## 3.1 UML Class Diagram

The UML Class Diagram represents the core entities of the GarageGo platform and their relationships. It provides a high-level view of the system structure and interactions between domain objects.

**Figure 3:** GarageGo – UML Class Diagram  
`[see documents/diagrams/class-diagram/ClassDiagram.png]`

---

## 3.2 Domain Class Descriptions

### Customer

Represents a registered user who can manage vehicles, make bookings, request emergency assistance, and submit reviews.

| Category | Details |
|-----------|---------|
| Key Attributes | customerId, fullName, phone, email, passwordHash, isActive |
| Key Methods | register(), login(), addVehicle(), createBooking(), requestEmergency(), submitReview() |

---

### Vehicle

Represents a vehicle owned by a customer.

| Category | Details |
|-----------|---------|
| Key Attributes | vehicleId, customerId, vehicleType, brand, model, registrationNo |
| Key Methods | addVehicle(), updateVehicle(), viewHistory() |

---

### Garage

Represents a service provider registered on the platform.

| Category | Details |
|-----------|---------|
| Key Attributes | garageId, businessName, status, averageRating, capacities |
| Key Methods | register(), manageServices(), acceptBooking(), handleEmergency() |

---

### GarageOwner

Represents a garage owner account responsible for managing garages and operations.

| Category | Details |
|-----------|---------|
| Key Attributes | ownerId, email, phone, isVerified |
| Key Methods | login(), manageGarage(), assignMechanic(), generateReports() |

---

### Mechanic

Represents mechanics assigned to garages and emergency requests.

| Category | Details |
|-----------|---------|
| Key Attributes | mechanicId, garageId, fullName, isAvailable, currentLocation |
| Key Methods | login(), updateLocation(), acceptTask(), completeTask() |

---

### Booking

Represents scheduled service appointments between customers and garages.

| Category | Details |
|-----------|---------|
| Key Attributes | bookingId, customerId, vehicleId, garageId, status, scheduledAt |
| Key Methods | create(), updateStatus(), cancel(), complete() |

---

### EmergencyRequest

Represents roadside assistance requests submitted by customers.

| Category | Details |
|-----------|---------|
| Key Attributes | emergencyId, customerId, vehicleId, status, location |
| Key Methods | trigger(), assignGarage(), dispatchMechanic(), complete() |

---

### Review

Represents customer feedback and ratings for completed services.

| Category | Details |
|-----------|---------|
| Key Attributes | reviewId, bookingId, customerId, garageId, rating |
| Key Methods | submit(), validate(), updateRating() |

---

### Notification

Represents system-generated notifications sent to users.

| Category | Details |
|-----------|---------|
| Key Attributes | notificationId, recipientId, type, payload, sentAt |
| Key Methods | send(), retry() |

---

### Admin

Represents platform administrators responsible for governance and management.

| Category | Details |
|-----------|---------|
| Key Attributes | adminId, email, role |
| Key Methods | approveGarage(), manageUsers(), generateReports(), monitorPlatform() |

---

## 3.3 Enumerations

Enumerations define fixed sets of values used throughout the system.

| Enumeration | Values |
|-------------|--------|
| VehicleType | TWO_WHEEL, FOUR_WHEEL, HEAVY |
| BookingStatus | PENDING, ACCEPTED, IN_SERVICE, COMPLETED, PAID, CANCELLED |
| EmergencyStatus | SEARCHING, ASSIGNED, DISPATCHED, IN_PROGRESS, COMPLETED, CANCELLED |
| GarageStatus | PENDING, APPROVED, SUSPENDED |
| PaymentStatus | UNPAID, PAID |
| NotificationType | BOOKING_REQUEST, BOOKING_ACCEPTED, BOOKING_REJECTED, EMERGENCY_BROADCAST, STATUS_UPDATE |
| UserRole | CUSTOMER, GARAGE_OWNER, MECHANIC, ADMIN |
| FuelType | PETROL, DIESEL, ELECTRIC, HYBRID |
| TransmissionType | MANUAL, AUTOMATIC |

---

## 3.4 Relationships and Multiplicity

The following relationships define how domain classes interact within the system.

| Relationship | Type | Multiplicity |
|--------------|------|-------------|
| Customer → Vehicle | Composition | 1 : Many |
| Customer → Booking | Association | 1 : Many |
| Customer → EmergencyRequest | Association | 1 : Many |
| Vehicle → Booking | Association | 1 : Many |
| GarageOwner → Garage | Association | 1 : Many |
| Garage → Mechanic | Aggregation | 1 : Many |
| Garage → Booking | Association | 1 : Many |
| Garage → EmergencyRequest | Association | 1 : Many |
| EmergencyRequest → Mechanic | Association | 0..1 |
| Booking → Review | Association | 0..1 |
| Review → Garage | Association | Many : 1 |
| Notification → User | Dependency | Many : 1 |

---

## 3.5 Class Diagram Summary

The GarageGo class model follows an object-oriented design where:

- Customers manage vehicles, bookings, emergency requests, and reviews.
- Garages provide services and employ mechanics.
- Garage Owners manage garages and operational activities.
- Mechanics handle assigned emergency and service tasks.
- Bookings and Emergency Requests form the core service workflows.
- Reviews provide service feedback and rating aggregation.
- Notifications support system-wide communication.
- Administrators oversee platform governance and approval processes.

The class diagram serves as the foundation for implementing the business logic layer and maintaining consistency between the system design and database model.
<<<<<<< draft


# Chapter 4: Sequence Diagrams

This chapter presents sequence diagrams for all ten use cases defined in the GarageGo SRS v0.1 (UC-01 to UC-10). Each diagram shows internal object interactions, method call labels, return values, loop frames, and alternative frames in UML 2.x notation. Diagrams are stored in `documents/diagrams/sequence-diagram/`.

---

## 4.1 UC-01 – Register Account

**Figure 4:** Sequence Diagram – Register Account  
`[see documents/diagrams/sequence-diagram/SD-01-Register.png]`

### Table 14: Sequence Steps – UC-01 Register Account

| # | From | To | Message / Action |
|---|------|----|------------------|
| 1 | User (App) | Customer App | Open registration screen; enter fullName, phone, email, NIC, password |
| 2 | Customer App | API Server | POST `/auth/register` { fullName, phone, email, nicNumber, password } |
| 3 | API Server | AuthService | validateRegistrationInput(dto) |
| 4 | AuthService | PostgreSQL DB | SELECT checkEmailUnique(email), checkPhoneUnique(phone) |
| 5 | PostgreSQL DB | AuthService | return: unique = true |
| 6 | AuthService | AuthService | hashPassword (BCrypt cost 12) [self-call] |
| 7 | AuthService | AuthService | encryptNIC (AES-256) [self-call] |
| 8 | AuthService | PostgreSQL DB | INSERT INTO customers (…) |
| 9 | AuthService | Twilio | sendOTP(phone) (6-digit OTP, 5-min TTL via Redis) |
| 10 | API Server | Customer App | HTTP 201 Created { customerId, message: "OTP sent" } |
| 11 | User (App) | Customer App | Enter OTP received via SMS |
| 12 | Customer App | API Server | POST `/auth/verify-otp` { customerId, otp } |
| 13 | API Server | AuthService | verifyOTP(customerId, otp) (Redis TTL check) |
| 14 | AuthService | PostgreSQL DB | UPDATE customers SET isActive = true |
| 15 | API Server | Customer App | HTTP 200 OK { accessToken, refreshToken } |
| 16 | Customer App | User | Redirect to Home Dashboard |

---

## 4.2 UC-04 – Book Service

**Figure 5:** Sequence Diagram – Book Service  
`[see documents/diagrams/sequence-diagram/SD-04-BookService.png]`

### Table 15: Sequence Steps – UC-04 Book Service

| # | From | To | Message / Action |
|---|------|----|------------------|
| 1 | Customer | Customer App | Select garage, vehicle, service type, date/time, notes |
| 2 | Customer App | API Server | POST `/bookings` { garageId, vehicleId, serviceType, scheduledAt, notes } |
| 3 | API Server | BookingService | validateBookingRequest(dto) |
| 4 | BookingService | PostgreSQL DB | checkCapacity(garageId, vehicleType, scheduledAt) |
| 5 | PostgreSQL DB | BookingService | return: available = true |
| ALT | Slot unavailable | API Server | HTTP 409 Conflict |
| 6 | BookingService | PostgreSQL DB | INSERT INTO bookings (…) status = PENDING |
| 7 | BookingService | NotificationService | sendPushNotification(garageOwnerId, "NEW_BOOKING_REQUEST") |
| 8 | NotificationService | FCM | POST `/fcm/send` |
| 9 | API Server | Customer App | HTTP 201 Created { bookingId, status: PENDING } |
| 10 | Customer App | Customer | Show "awaiting confirmation" |
| 11 | GarageOwner App | API Server | POST `/bookings/{id}/accept` |
| 12 | BookingService | PostgreSQL DB | UPDATE status = ACCEPTED |
| 13 | BookingService | NotificationService | notify customer |
| 14 | API Server | Customer App | Socket event: booking_accepted |

---

## 4.3 UC-06 & UC-07 – Emergency SOS & GPS Tracking

**Figure 6:** Sequence Diagram – Emergency SOS  
`[see documents/diagrams/sequence-diagram/SD-06-EmergencySOS.png]`

### Table 16: Sequence Steps – Emergency SOS Dispatch

| # | From | To | Message / Action |
|---|------|----|------------------|
| 1 | Customer | Customer App | Press SOS, enter issue |
| 2 | App | API Server | POST `/emergency` |
| 3 | API Server | EmergencyService | createEmergencyRequest(dto) |
| 4 | EmergencyService | DB | INSERT status = SEARCHING |
| 5 | EmergencyService | Google Maps API | findNearbyGarages |
| 6 | Google Maps API | EmergencyService | return garages |
| 7 | EmergencyService | Socket Server | broadcast emergency_alert |
| 8 | API Server | App | HTTP 201 SEARCHING |

### LOOP: 60-second acceptance window

| # | From | To | Message / Action |
|---|------|----|------------------|
| 9 | GarageOwner App | API Server | accept emergency |
| 10 | EmergencyService | DB | assign first accepter |
| 11 | EmergencyService | Socket Server | emergency_taken |
| 12 | EmergencyService | NotificationService | notify customer |

### ALT: No acceptance in 60s
- Expand radius and rebroadcast

### Dispatch & Tracking

| # | From | To | Message / Action |
|---|------|----|------------------|
| 14 | GarageOwner App | API Server | assign mechanic |
| 15 | EmergencyService | DB | status = DISPATCHED |
| 16 | Mechanic App | Socket Server | location updates (every 5s) |
| 17 | EmergencyService | DB | update GPS |
| 18 | Socket Server | Customer App | live location + ETA |
| 19 | Customer App | User | display map tracking |

### ALT: Signal loss > 30s
- Show "Tracking paused – reconnecting"

---

## 4.4 UC-09 – Rate Garage

**Figure 7:** Sequence Diagram – Rate Garage  
`[see documents/diagrams/sequence-diagram/SD-09-RateGarage.png]`

### Table 17: Sequence Steps – UC-09 Rate Garage

| # | From | To | Message / Action |
|---|------|----|------------------|
| 1 | Customer | App | Open completed booking |
| 2 | App | API Server | POST `/reviews` |
| 3 | API Server | ReviewService | validateReviewEligibility |
| 4 | ReviewService | DB | verify booking status |
| 5 | DB | ReviewService | eligible = true |
| ALT | Not eligible | API | HTTP 403 |
| 6 | ReviewService | ReviewService | content moderation |
| ALT | Offensive content | API | HTTP 422 |
| 7 | ReviewService | DB | INSERT review |
| 8 | ReviewService | DB | update average rating |
| 9 | API Server | App | HTTP 201 |
| 10 | App | Customer | confirmation message |

---

## 4.5 UC-10 & UC-21 – Garage Registration & Approval

**Figure 8:** Sequence Diagram – Garage Registration & Approval  
`[see documents/diagrams/sequence-diagram/SD-10-GarageRegistration.png]`

### Table 18: Sequence Steps – UC-10 & UC-21

| # | From | To | Message / Action |
|---|------|----|------------------|
| 1 | GarageOwner | App | Fill registration form |
| 2 | App | API Server | POST `/garages/register` |
| 3 | API Server | GarageService | validate request |
| 4 | GarageService | Cloud Storage | upload documents |
| 5 | GarageService | DB | INSERT status = PENDING |
| 6 | API Server | App | HTTP 201 |
| 7 | API Server | Admin Dashboard | notify pending approval |
| 8 | Admin | Dashboard | review application |
| 9 | Admin Dashboard | API Server | PUT `/admin/garages/{id}/approve` |
| 10 | AdminService | DB | UPDATE status = APPROVED |
| 11 | AdminService | NotificationService | email garage owner |
| 12 | API Server | Dashboard | HTTP 200 |

### ALT: Reject flow
- Update status = REJECTED  
- Notify garage owner via email



# Chapter 5: State Machine Diagrams

This chapter presents state machine diagrams for the two primary stateful entities in the GarageGo system: **Booking** and **EmergencyRequest**. Both entities have well-defined lifecycle states whose transitions are governed by business rules defined in the SRS. These diagrams directly realize **FR-09 (Booking State Machine)** and **FR-05 / FR-06 (Emergency Request lifecycle)**.

---

## 5.1 Booking State Machine

The Booking entity progresses through six states as defined in FR-09. The `CANCELLED` state is reachable from both `PENDING` and `ACCEPTED`, subject to the cancellation policy defined in UC-05.

**Figure 9:** GarageGo – Booking State Machine Diagram  
`[see documents/diagrams/state-machine/SM-Booking.png]`

### Table 19: Booking State Transitions

| From State | Trigger / Event | To State | Business Rule / FR Reference |
|------------|----------------|----------|------------------------------|
| [Initial] | Customer confirms booking request | PENDING | FR-04; booking stored with status PENDING; 2-hour garage response timer starts (BR-08) |
| PENDING | Garage owner accepts booking | ACCEPTED | FR-09; push notification sent to customer (FR-14) |
| PENDING | Garage owner rejects booking / timer expires | CANCELLED | BR-08; customer notified of rejection |
| PENDING | Customer cancels before acceptance | CANCELLED | UC-05; no cancellation fee if not yet accepted (BR-10) |
| ACCEPTED | Customer cancels after acceptance | CANCELLED | UC-05 Alt Flow 1; cancellation fee applied (BR-10) |
| ACCEPTED | Garage starts the repair work | IN_SERVICE | FR-09; cannot cancel once in service |
| IN_SERVICE | Garage marks repair complete | COMPLETED | FR-09; final cost recorded; customer prompted to pay |
| COMPLETED | Garage owner confirms cash receipt | PAID | SRS §2.5; payment confirmed within 24 hours |
| PAID | [Terminal] | — | Service history updated; customer prompted for review (FR-13) |
| CANCELLED | [Terminal] | — | Cancellation fee (if any) added to next invoice (BR-11) |

---

## 5.2 EmergencyRequest State Machine

The EmergencyRequest entity progresses through six states from the moment a customer presses the SOS button to the completion of roadside assistance. The search radius expansion mechanism is a guard-triggered automatic transition if no garage accepts within 60 seconds.

**Figure 10:** GarageGo – EmergencyRequest State Machine Diagram  
`[see documents/diagrams/state-machine/SM-EmergencyRequest.png]`

### Table 20: EmergencyRequest State Transitions

| From State | Trigger / Event | To State | Business Rule / FR Reference |
|------------|----------------|----------|------------------------------|
| [Initial] | Customer presses SOS button; GPS captured | SEARCHING | FR-05; broadcast sent to all eligible garages within radius (BR-12) |
| SEARCHING | First garage accepts within 60 seconds | ASSIGNED | FR-06; SELECT FOR UPDATE lock prevents race condition (BR-13) |
| SEARCHING | 60 seconds elapsed with no acceptance | SEARCHING | UC-06 Alt Flow 2; searchRadiusKm += 10; rebroadcast |
| SEARCHING | Customer cancels before assignment | CANCELLED | UC-06 cancellation path |
| ASSIGNED | Garage owner assigns and dispatches mechanic | DISPATCHED | FR-15; mechanic app receives job details |
| DISPATCHED | Mechanic marks arrived at customer location | IN_PROGRESS | Mechanic app "Mark Arrived" action; customer notified (FR-14) |
| IN_PROGRESS | Mechanic marks job complete | COMPLETED | Mechanic app "Mark Complete" action; service record created |
| COMPLETED | [Terminal] | — | Customer prompted to review garage; emergency closes |
| CANCELLED | [Terminal] | — | No charge applied; customer may trigger new SOS |

---



# Chapter 6: UI/UX Design

This chapter describes the user interface design principles, key screen descriptions with accessibility considerations, and the navigation flow for the GarageGo Customer App and Garage Owner App. All screens are implemented using React Native (Expo) with responsive layouts targeting iOS 15+ and Android 10+.

---

## 6.1 Design Principles

- **Emergency First:** The SOS button is the most prominent element on the Customer App home screen, visible without scrolling at a minimum tap target of 48×48 dp (NFR-U1), coloured red to signal urgency.  
- **Transparency:** All garage listings display pricing, availability, and ratings upfront, directly addressing the price opacity pain point identified in Interview #9 (Persona Kasun: *"I want to see the price before they start working"*).  
- **Feedback:** Every user action receives immediate visual feedback including loading indicators, success toasts, and error banners. Offline state is communicated within 3 seconds (NFR-U3).  
- **Simplicity:** Each screen focuses on a single primary task; secondary actions are placed in bottom sheets or secondary panels to reduce cognitive load for users in stressful breakdown situations.  
- **Accessibility:** Minimum tap target of 48×48 dp throughout (Material Design guidelines); ARIA-equivalent React Native accessibility labels on all interactive components; colour contrast ratio meeting WCAG 2.1 AA.  
- **Registration Speed:** The full registration flow is achievable within 3 minutes on a standard 4G connection (NFR-U2), with OTP auto-detection where supported by the OS.

---

## 6.2 Screen Descriptions and Accessibility Considerations

### Table 21: Customer App – Screen Descriptions and Accessibility

| Screen | Key UI Elements | Accessibility Considerations |
|--------|----------------|------------------------------|
| Registration / OTP | Phone, email, NIC, password fields; OTP input (6-digit) | All inputs have `accessibilityLabel`; OTP auto-focus on SMS; password show/hide toggle; error states described via `accessibilityHint` |
| Home Dashboard | SOS button (48×48 dp), quick actions, notifications | SOS button uses `accessibilityRole='button'`; labeled as “Emergency SOS”; focus ring enabled; notification badge exposed |
| Garage Search & Map | Map view, garage pins, filters, list toggle | Filter panel keyboard-navigable; map pins labeled with garage name & rating; list alternative for non-visual access |
| Garage Profile | Photos, ratings, reviews, service list, booking CTA | Images have descriptive labels; rating exposed via `accessibilityValue`; CTA always ≥48dp |
| Booking Creation | Vehicle selector, service picker, date/time, cost estimate | Progress indicator uses `progressbar`; native OS pickers used; cost changes announced dynamically |
| Emergency SOS Trigger | Full-screen SOS modal, GPS pin, issue description | Distraction-free layout; GPS status announced via `accessibilityLiveRegion`; manual location fallback available |
| Mechanic Live Tracking | Map with marker, ETA banner, status timeline | ETA also shown in text; updates announced via `accessibilityLiveRegion='polite'`; call mechanic uses `tel:` URI |
| Service History | Booking history list, invoices | List uses `accessibilityRole='list'`; each item announces key service info |
| Rate & Review | Star rating, comment box, submit button | Star selector uses slider role; character counter announced dynamically |

---

### Figure (UI Prototype Reference)
**Untitled – Figma**  
*(UI prototype reference for customer screens and interactions)*

---

### Table 22: Garage Owner App – Screen Descriptions

| Screen | Key UI Elements | Accessibility Considerations |
|--------|----------------|------------------------------|
| Dashboard | Active bookings, capacity gauges, revenue summary | Numeric values exposed via `accessibilityValue`; alerts use live regions |
| Booking Request | Booking details, accept/reject, cost input, countdown | Timer announced periodically; buttons ≥48dp; inline validation feedback |
| Emergency Alert | Full-screen emergency request view | High-contrast UI; 60s countdown announced; distance explicitly read aloud |
| Mechanic Assignment | Mechanic list, assign button, confirmation dialog | List uses `accessibilityRole='list'`; dialogs use alert semantics |
| Service Management | Booking lifecycle controls, invoice generation | State transition buttons clearly labeled; confirmations announced via alerts |

---

### Additional App Screens (Overview)

- Mechanic App – Active Jobs, GPS Navigation, Job Status Updates  
- Admin App – Garage Approval, User Management, Complaints, Analytics  

---

## 6.3 Navigation Flow

The navigation flow enforces role-based routing at the React Navigation level using JWT role claims. Unauthenticated users are redirected to the Login screen. An invalid or expired token triggers automatic redirect to Login with a toast notification.

**Figure 11:** GarageGo – Navigation Flow Diagram  
`[see documents/prototypes/navigation-flow.png]`

---

### Navigation Behavior

- **Unauthenticated Users**
  - Splash → Login → Register / Login flow

- **Customer Role**
  - Home Dashboard  
  → Garage Search  
  → Booking Creation  
  → Emergency SOS  
  → Live Mechanic Tracking  
  → Service History  
  → Profile  

- **Garage Owner Role**
  - Garage Dashboard  
  → Booking Requests  
  → Emergency Alerts  
  → Mechanic Assignment  
  → Service Management  
  → Analytics  

- **Mechanic Role**
  - Active Jobs Screen  
  → GPS Navigation Controls  
  → Job Status Updates  

- **Admin Role**
  - Admin Dashboard (Web)  
  → Garage Approval  
  → User Management  
  → Complaint Resolution  
  → Platform Analytics  

- **Logout Flow**
  - Any screen → Logout → Login screen

---



# Chapter 7: Security Design

The GarageGo system implements a defense-in-depth security strategy with independent protection layers: Transport Security (TLS 1.2+/WSS), Authentication (JWT + OTP), Authorisation (RBAC), Data Security (BCrypt + AES-256), and Input Validation (server-side + schema validation). A breach of one layer does not compromise the entire system.

---

## 7.1 Authentication and Authorisation Strategy

Authentication is implemented using JSON Web Tokens (JWT) issued by the Node.js/Express API upon successful credential verification plus OTP confirmation. The access token has a validity period of **15 minutes (NFR-S1)**. A refresh token valid for **7 days** is stored in an HttpOnly cookie to prevent JavaScript access and is used to obtain new access tokens transparently.

- All users (Customer, Garage Owner, Mechanic) verify their phone number via Twilio OTP during registration (FR-01; BR-01). Garage Owner email verification is also mandatory (BR-02).
- JWT tokens carry the role claim (**CUSTOMER | GARAGE_OWNER | MECHANIC | ADMIN**) used for RBAC enforcement at the Express middleware layer.
- All API endpoints except `/auth/register` and `/auth/login` require a valid JWT. Missing or expired tokens return HTTP 401 (NFR-S1).
- Admin role access is enforced via middleware verifying `role == 'ADMIN'` before any `/admin/*` route is processed.

---

## 7.2 Security Design Summary

### Table 23: Security Design Summary

| Security Aspect | Mechanism | Details |
|----------------|----------|---------|
| Authentication | JWT (HS256) + Twilio OTP | Access tokens expire after 15 minutes (NFR-S1); refresh tokens stored in HttpOnly cookie (7-day expiry) |
| Authorisation | RBAC via Express middleware | Roles: CUSTOMER, GARAGE_OWNER, MECHANIC, ADMIN. Middleware enforces role per route group |
| Password Storage | BCrypt (cost factor 12) | Passwords never stored in plain text; BCrypt applied on registration and reset (NFR-S2) |
| Data in Transit | TLS 1.2+ + WSS | All REST APIs via HTTPS; Socket.io uses WSS (NFR-S3) |
| Data at Rest | AES-256 + BCrypt | NIC encrypted; passwords hashed with BCrypt (NFR-S2) |
| Input Validation | Express-validator + Joi | Schema-based validation; unknown fields rejected |
| SQL Injection | Parameterised queries (pg/Sequelize) | No raw SQL string concatenation |
| XSS Prevention | React/React Native escaping + CSP | Admin dashboard secured via CSP headers |
| Race Conditions | SELECT FOR UPDATE (PostgreSQL lock) | Prevents duplicate emergency assignments (FR-06) |
| File Upload Security | MIME + size validation | JPEG/PNG (10MB max), PDF (5MB max), server-side enforcement |
| Audit Logging | Morgan + event logger | Logs auth events, admin actions, emergency assignments |

---

## 7.3 Input Validation and Sanitisation Strategy

All input data is validated at two independent layers:

- **Client-side validation (React Native / React.js):**
  - Required field enforcement
  - Phone format validation (E.164)
  - Email format validation
  - NIC pattern validation
  - GPS coordinate range validation

- **Server-side validation (Express + Joi):**
  - All request bodies validated against strict schemas
  - Length constraints enforced
  - Unknown fields rejected with HTTP 400
  - String trimming applied before processing

Additional protections:
- Review text filtered for offensive patterns before storage (UC-09 Alt Flow 2)
- File uploads restricted by MIME type and size validation
- GPS and numeric inputs validated against safe bounds

---

## 7.4 Encryption Standards

### Table 24: Encryption Standards

| Data Category | Standard | Implementation |
|--------------|----------|---------------|
| Data in Transit (REST) | TLS 1.2+ | Enforced via Nginx/load balancer; HTTP redirected to HTTPS |
| Data in Transit (WebSocket) | WSS (TLS 1.2+) | Socket.io over HTTPS server only |
| Passwords at Rest | BCrypt (cost 12) | Applied before DB storage (NFR-S2) |
| NIC Numbers at Rest | AES-256-GCM | Application-level encryption using secure key storage |
| JWT Signature | HS256 | Secret stored in environment variables |
| File Storage | Provider encryption | Cloudinary/S3 server-side encryption enabled |

---

## 7.5 OWASP Top 10 Considerations

### Table 25: OWASP Top 10 – GarageGo Controls

| # | OWASP Risk | GarageGo Control |
|--|------------|------------------|
| A01 | Broken Access Control | RBAC middleware enforces role-based access on all routes |
| A02 | Cryptographic Failures | TLS 1.2+, BCrypt, AES-256-GCM, no sensitive JWT payload data |
| A03 | Injection | Parameterised queries + Joi validation |
| A04 | Insecure Design | Threat modelling + race condition locking (SELECT FOR UPDATE) |
| A05 | Security Misconfiguration | Secrets in env vars; restricted CORS; API docs disabled in production |
| A06 | Vulnerable Components | CI/CD `npm audit`; dependency locking via package-lock.json |
| A07 | Authentication Failures | OTP required; JWT expiry 15 min; (future: rate limiting planned) |
| A08 | Data Integrity Failures | JWT verification + strict state machines + file validation |
| A09 | Logging Failures | Full audit logging (auth, admin, emergency events) |
| A10 | SSRF | No user-controlled URL fetching; fixed Google Maps API endpoints |

---


# Chapter 8: Interface Design

This chapter specifies all external interfaces (third-party APIs and services) and internal module interfaces for the GarageGo platform.

---

## 8.1 External Interfaces

### Table 26: External Interface Specifications

| Interface | Provider | Purpose | Key Endpoints / Integration |
|----------|----------|--------|------------------------------|
| Google Maps Geocoding API | Google Maps Platform | Convert garage addresses to latitude/longitude during registration (FR-10) | `GET /geocode/json?address=&key=` → returns `lat`, `lng`, `formatted_address` |
| Google Maps Distance Matrix API | Google Maps Platform | Calculate distance and ETA between mechanic and customer during emergency (FR-07) | `GET /distancematrix/json?origins=&destinations=&key=` → returns `distance_value`, `duration_value` |
| Google Maps Directions API | Google Maps Platform | Provide turn-by-turn navigation for mechanic app (FR-15) | `GET /directions/json?origin=&destination=&key=` → returns encoded polyline |
| Twilio SMS API | Twilio Inc. | Send OTP codes for phone verification (FR-01; BR-01) | `POST /2010-04-01/Accounts/{SID}/Messages` → sends 6-digit OTP |
| Firebase Cloud Messaging (FCM) | Google Firebase | Push notifications for Customer, Garage Owner, Mechanic apps (FR-14) | `POST https://fcm.googleapis.com/fcm/send` → delivers notification payload |
| Cloudinary / Amazon S3 | Cloudinary / AWS | Store images, documents, and complaint evidence | `POST /auto/upload` → returns `secure_url` for database storage |

---

## 8.2 Internal Module Interfaces

### Table 27: Internal Module Interface Specifications

| From Module | To Module | Interface Type | Key Interactions |
|-------------|----------|---------------|------------------|
| Express REST API | PostgreSQL 15 | TCP/IP via `node-postgres (pg)` | Parameterised SQL queries using `pg.Pool`; max 20 connections per instance; all queries use prepared statements |
| Express REST API | Socket.io Server | In-process EventEmitter (same Node.js runtime) | `EmergencyService` emits `emergency_alert`; `BookingService` emits `booking_status` |
| Express REST API | Redis 7 | TCP/IP via `ioredis` | OTP storage `SET otp:{customerId} EX 300`; caching `garages:nearby:{geohash}`; session/state caching |
| Socket.io Server | Redis 7 (Pub/Sub) | socket.io-redis adapter | Enables cross-instance message broadcasting for scalability (NFR-SC1) |
| Express REST API | NotificationService | In-process service call | `sendPush(token, type, payload)` → FCM; `sendSMS(phone, message)` → Twilio |
| Express REST API | Google Maps Client | HTTPS via `@googlemaps/google-maps-services-js` | Geocoding, nearby garage search, ETA calculation |
| Mobile Apps (Customer / Garage / Mechanic) | Express REST API | HTTPS REST (JSON) | All authentication, CRUD operations, booking, emergency handling via `/api/v1/*` |
| Mechanic App | Socket.io Server | WSS WebSocket | Emits `location_update` every 5 seconds; receives `job_assigned`, `navigation_update` |
| Customer App | Socket.io Server | WSS WebSocket | Receives real-time `mechanic_location`, `booking_status`, `emergency_assigned` |

---



# Chapter 9: Deployment Design

This chapter describes the hosting plan, environment specifications, deployment network architecture, and CI/CD pipeline for the GarageGo platform. All components are containerised using Docker to ensure environment parity between development, staging, and production.

---

## 9.1 Hosting Plan

### Table 28: GarageGo Deployment Components

| Component | Technology | Specification |
|----------|------------|--------------|
| Customer Mobile App | React Native (Expo) | Published to Apple App Store and Google Play Store; OTA updates via Expo EAS |
| Garage Owner Mobile App | React Native (Expo) | Published to App Store and Play Store alongside Customer App |
| Mechanic Mobile App | React Native (Expo) | Published to App Store and Play Store; GPS always-on during active dispatch |
| Admin Web Dashboard | React.js (Vite) | Served as static build from Nginx container; HTTPS with valid TLS certificate |
| REST API Server | Node.js 20 LTS / Express | Docker container behind load balancer; min 1 instance, scales to 4 (NFR-SC3) |
| Socket.io Server | Node.js 20 LTS / Socket.io 4 | Co-located with REST API; Redis adapter enables multi-instance scaling (NFR-SC1) |
| Database | PostgreSQL 15 | Managed cloud service; daily backups retained 30 days (NFR-R3); partitioned booking tables (NFR-SC2) |
| Cache / Session | Redis 7 | Managed service used for OTP TTL, session handling, and caching |
| File Storage | Cloudinary / Amazon S3 | Stores images, invoices, and complaint evidence (SRS §1.2.1) |
| Reverse Proxy | Nginx | Routes `/api/*` to backend; serves Admin Dashboard; enforces HTTPS redirect |

---

## 9.2 Environment Specification

### Table 29: Environment Specification

| Attribute | Development | Staging | Production |
|-----------|-------------|---------|------------|
| Infrastructure | Local Docker Compose | Single cloud VM | Multi-container cluster behind load balancer |
| Database | Local PostgreSQL 15 | Managed PostgreSQL (single instance) | Managed PostgreSQL (HA + backups) |
| Redis | Local Redis 7 container | Managed Redis (single node) | Managed Redis (persistent) |
| Branch Strategy | feature/* | develop | main |
| Logging Level | DEBUG + full traces | INFO level | ERROR + audit logs only |
| HTTPS | Self-signed certificate | Valid TLS (staging domain) | Valid TLS (production domain) |
| External APIs | Mock services | Test credentials (Twilio/FCM) | Live production keys |
| Unit Test Coverage | ≥70% (NFR-M1) | ≥70% enforced | ≥70% enforced via CI/CD gate |

---

## 9.3 Network Architecture

The deployment architecture follows a secure, segmented network model:

- All backend services are deployed inside a **private network**
- The **load balancer is the only public entry point**
- PostgreSQL and Redis are fully isolated in a **private subnet**
- No direct external access to database or cache layers
- Outbound API calls (Google Maps, Twilio, FCM) are routed via **NAT gateway**

This architecture enforces strong isolation between application layers and reduces the attack surface.

---

### Figure 12: Deployment Network Diagram  
`[see documents/diagrams/architecture/deployment-diagram.png]`

---



# Chapter 10: Non-Functional Design Decisions

This chapter documents how each of the fourteen non-functional requirements defined in the GarageGo SRS v0.1 (Sections 5.2–5.7) is addressed by the system design. Each entry states the NFR, the design decision that satisfies it, and the responsible component or configuration.

---

## 10.1 Performance

### Table 30: Performance NFR Design Decisions

| NFR ID | Requirement (SRS §5.2) | Design Decision | Responsible Component |
|--------|------------------------|----------------|----------------------|
| NFR-P1 | 95% of REST API requests respond within 2 seconds under 50 concurrent users | Stateless Express API with connection pooling (`pg.Pool` max 20); Redis caching for garage list results; indexed PostgreSQL queries on `garageId`, `customerId`, `status` | Express API, pg.Pool, Redis, PostgreSQL indices |
| NFR-P2 | Emergency SOS broadcast delivered within 2 seconds of trigger | Socket.io emits directly to garage-specific rooms after `findNearbyGarages()`; Redis pub/sub ensures cross-instance delivery | Socket.io Server, EmergencyService, Redis |
| NFR-P3 | Mechanic GPS relay within 1 second; max 6-second end-to-end latency | Mechanic app emits location every 5 seconds; Socket.io forwards instantly to customer room without persistence delay | Socket.io Server, Mechanic App, Customer App |

---

## 10.2 Security

### Table 31: Security NFR Design Decisions

| NFR ID | Requirement (SRS §5.3) | Design Decision | Responsible Component |
|--------|------------------------|----------------|----------------------|
| NFR-S1 | Valid JWT required for all protected endpoints; 15-min expiry; 7-day refresh | Express middleware validates JWT on `/api/v1/*`; HS256 signing; refresh token in HttpOnly cookie | Express JWT Middleware, AuthService |
| NFR-S2 | BCrypt hashing ≥ cost 12; no plain-text storage | `bcryptjs` with `saltRounds=12` in `AuthService.hashPassword()` before DB write | AuthService |
| NFR-S3 | TLS 1.2+ and WSS required for all communication | Nginx enforces HTTPS; Socket.io runs over HTTPS server instance only | Nginx, Socket.io Server |

---

## 10.3 Usability

### Table 32: Usability NFR Design Decisions

| NFR ID | Requirement (SRS §5.4) | Design Decision | Responsible Component |
|--------|------------------------|----------------|----------------------|
| NFR-U1 | SOS button visible without scrolling; ≥48×48 dp | Full-width SOS button placed above fold with height 80 and minimum touch target compliance | Customer App – HomeScreen |
| NFR-U2 | Registration completes within 3 minutes on 4G | Single-page form; SMS OTP auto-detection; optimized keyboard types per field | RegisterScreen |
| NFR-U3 | Error feedback within 3 seconds; no raw error codes | NetInfo listener + Redux state triggers `ErrorBanner` with mapped user-friendly messages | NetworkMonitor, ErrorBanner |

---

## 10.4 Reliability

### Table 33: Reliability NFR Design Decisions

| NFR ID | Requirement (SRS §5.5) | Design Decision | Responsible Component |
|--------|------------------------|----------------|----------------------|
| NFR-R1 | 99.5% uptime during peak hours | Auto-restart containers; `/health` endpoint monitored by load balancer | Docker, Load Balancer |
| NFR-R2 | Webhook failure recovery within 10 minutes | Bull queue reconciliation job reprocesses failed events every 10 minutes | Bull Queue, Redis |
| NFR-R3 | Daily backups with 30-day retention | Managed PostgreSQL snapshots with restore testing in runbooks | PostgreSQL Managed Service |

---

## 10.5 Scalability

### Table 34: Scalability NFR Design Decisions

| NFR ID | Requirement (SRS §5.6) | Design Decision | Responsible Component |
|--------|------------------------|----------------|----------------------|
| NFR-SC1 | ≥500 concurrent Socket.io connections | Redis pub/sub + socket.io-redis adapter; horizontal scaling to 3 instances (~600 capacity) | Socket.io, Redis |
| NFR-SC2 | Partitioned bookings table supports 100k+ records | PostgreSQL RANGE partitioning by `scheduledAt` with pg-cron automation | PostgreSQL |
| NFR-SC3 | Adding API instance increases throughput ≥80% | Stateless architecture with JWT + load balancer round-robin distribution | Express API, Load Balancer |

---

## 10.6 Maintainability

### Table 35: Maintainability NFR Design Decisions

| NFR ID | Requirement (SRS §5.7) | Design Decision | Responsible Component |
|--------|------------------------|----------------|----------------------|
| NFR-M1 | ≥70% test coverage enforced | Jest coverage thresholds enforced in CI via GitHub Actions | Jest, GitHub Actions |
| NFR-M2 | Independent redeployment ≤5 minutes, zero downtime | Dockerized services with rolling updates and health checks | Docker, CI/CD Pipeline |
| NFR-M3 | OpenAPI 3.0 documentation available | Swagger generated from JSDoc; served at `/api/docs` in dev mode | swagger-jsdoc, swagger-ui-express |

---

## References

### Table 36: References

| # | Reference |
|--|-----------|
| [1] | IEEE Std 1016-2009 – Software Design Descriptions |
| [2] | IEEE Std 830-1998 – Software Requirements Specifications |
| [3] | ISO/IEC 25010: System and Software Quality Models |
| [4] | Pressman, *Software Engineering: A Practitioner’s Approach*, 8th ed. |
| [5] | Sommerville, *Software Engineering*, 10th ed. |
| [6] | Node.js 20 LTS Documentation https://nodejs.org/en/docs/ |
| [7] | Socket.IO Documentation https://socket.io/docs/v4/ |
| [8] | Expo Documentation https://docs.expo.dev/ |
| [9] | PostgreSQL 15 Documentation https://www.postgresql.org/docs/15/ |
| [10] | Redis 7 Documentation https://redis.io/docs/ |
| [11] | Google Maps Platform Documentation https://developers.google.com/maps/documentation |
| [12] | Twilio Messaging API https://www.twilio.com/docs/messaging |
| [13] | Firebase Cloud Messaging https://firebase.google.com/docs/cloud-messaging |
| [14] | OWASP Top Ten https://owasp.org/www-project-top-ten/ |
| [15] | WCAG 2.1 Guidelines https://www.w3.org/TR/WCAG21/ |
| [16] | GarageGo SRS v0.1 – University of Kelaniya |
| [17] | SENG 31242 System Design Project Guidelines – 2026 |

---








=======
>>>>>>> main

