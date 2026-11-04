# 🔐 Secure File Transfer for GM Vehicle Ordering System (VOMS) using FTPS

## 🎯 Project Overview

| Item | Description |
|---|---|
| **Client** | General Motors (GM) |
| **System** | VOMS — Vehicle Ordering Management System (dealer vehicle orders, order changes, allocation and status) |
| **Problem** | Vehicle order, allocation, order-status and invoice files were exchanged between dealers, GM order systems, plants and downstream partners over plain FTP, exposing credentials and business-sensitive order and pricing data in clear text |
| **Solution** | Implemented **FTPS (FTP over TLS)** secure server and client applications with certificate-based authentication, encrypted control and data channels, and full transfer auditing |
| **Scope** | VOMS (GM data center), dealer systems (DMS), FTPS gateway, manufacturing and production scheduling, logistics/shipping and finance systems |
| **My Role** | <Your role — e.g., Software Engineer: design, development of FTPS client/server components, certificate setup, testing and production rollout> |

Secured file-based integration for GM's Vehicle Ordering Management System by replacing plain FTP with TLS-secured FTP (FTPS). Dealer orders, vehicle configurations, allocations, order-status updates and invoice files are now encrypted in transit, authenticated with digital certificates and fully audited. Existing batch schedules and order-to-delivery workflows kept running without interruption.

---

## 💼 Business Objectives

- **Protect order & pricing data:** encrypt dealer orders, vehicle configurations (VIN, options), allocation and pricing information in transit.
- **Secure credentials:** remove clear-text FTP usernames and passwords from GM, dealer and partner networks.
- **Trusted endpoints:** authenticate dealers, plants and partner systems with X.509 certificates (mutual TLS where supported).
- **Reliable order-to-delivery flow:** keep order, schedule and status files complete and unaltered, so dealers, plants and logistics stay in sync.
- **Compliance:** meet GM information-security standards, corporate audit and data-protection requirements for data in transit.
- **Full traceability:** log every session and file transfer for audits and order-dispute resolution.
- **Zero business disruption:** move dealers and partners to FTPS with configuration changes only, without affecting daily order processing.

---

## 🏗️ Architecture

​```mermaid
flowchart LR
    subgraph Dealers[GM Dealer Network]
        DMS[Dealer Management Systems]
    end
    subgraph GMDC[GM Data Center]
        VOMS[VOMS<br/>Vehicle Ordering Management]
        CA[(Certificate Authority)]
        LOG[(Audit Logs / SIEM)]
    end
    subgraph DMZ[DMZ]
        FTPS[FTPS Gateway<br/>TLS 1.2+ · Explicit AUTH TLS]
    end
    subgraph Downstream[Manufacturing & Partners]
        PLANT[Plant Production Scheduling]
        LOGI[Logistics / Vehicle Shipping]
        FIN[Finance · Invoicing · Floorplan]
    end

    DMS <-->|Orders · Changes · Status<br/>FTPS| FTPS
    FTPS <-->|Order & status files| VOMS
    VOMS <-->|Build schedule files| PLANT
    VOMS <-->|Shipment / ETA files| LOGI
    VOMS <-->|Invoice files| FIN
    CA -.->|Certificates| FTPS
    CA -.-> Dealers
    FTPS --> LOG
​```

### Components

| Component | Responsibility |
|---|---|
| **Dealer Management Systems** | Submit vehicle orders and order changes; receive allocation, order status and invoices |
| **VOMS** | Validates and consolidates orders, manages allocation, sends build requests to plants and tracks order-to-delivery status |
| **FTPS Gateway** | Single secure entry point for file exchange; enforces TLS, authentication and directory isolation |
| **FTPS Clients** | Dealer, VOMS and partner applications that upload and download files over TLS |
| **Plant / Logistics / Finance** | Production scheduling, vehicle shipment tracking and invoicing |
| **Certificate Authority** | Issues, renews and revokes server and client certificates |
| **Audit Logs / SIEM** | Records sessions, transfers, failures and security alerts |

### Security Design

| Area | Implementation |
|---|---|
| **Protocol** | FTPS — Explicit mode (`AUTH TLS`, port 21); Implicit mode (port 990) where required |
| **TLS Version** | TLS 1.2 / 1.3 only; SSLv3, TLS 1.0 and 1.1 disabled |
| **Cipher Suites** | Strong AES-GCM suites; weak, NULL and export ciphers disabled |
| **Channel Protection** | Control and data channels both encrypted (`PROT P`) |
| **Authentication** | Server certificate validation, plus account credentials and/or client certificates (mutual TLS) |
| **Authorization** | One account per dealer or partner, jailed home directories, least-privilege read/write |
| **Network** | Gateway in DMZ; firewall allows port 21/990 and a fixed passive port range from whitelisted IPs only |
| **File Integrity** | SHA-256 checksums, header/trailer record counts and naming conventions to confirm complete files |
| **Monitoring** | Transfer logs, failed-login alerts, certificate-expiry alerts |

### Vehicle Order File Flow

​```mermaid
sequenceDiagram
    autonumber
    participant D as Dealer (DMS)
    participant G as FTPS Gateway
    participant V as VOMS
    participant P as Plant Scheduling
    D->>G: Connect · AUTH TLS · certificate check
    D->>G: Upload vehicle order file (PROT P, encrypted)
    G->>V: Hand-off order file (checksum verified)
    V->>V: Validate order · allocate
    V->>P: Build request / schedule file
    P-->>V: Production & VIN assignment
    V->>G: Order status / acknowledgement file
    D->>G: Download status & invoice files
    G->>G: Audit log for every transfer
​```
