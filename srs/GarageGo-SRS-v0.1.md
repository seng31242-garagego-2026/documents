# GarageGo — Software Requirements Specification

**Document Version:** v0.1  
**Date:** 2026-05-22  
**Authors:** P.Kasturi, K.Kajaluxmy, S.Krishnapiriyan, V.Vanushan  
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
| **PostgreSQL** | Open-source relational database management system |
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
