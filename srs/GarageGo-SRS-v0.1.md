# GarageGo — Software Requirements Specification

**Document Version:** v0.1  
**Date:** 2026-05-22  
**Authors:** P. Kasturi  
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
4. Functional Requirements *[To be completed ]*  
5. Non-Functional Requirements *[To be completed ]*  
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
