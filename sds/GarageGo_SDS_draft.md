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
