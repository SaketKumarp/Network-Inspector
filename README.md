# NetSentinel — Network Inspector

> A network intelligence and security monitoring platform for capturing, analyzing, and understanding network traffic.

NetSentinel is a full-stack computer networking project focused on **packet inspection, network flow analytics, DNS intelligence, and rule-based threat detection**. It is designed to combine a high-performance C++ packet-processing engine with a Python API and a real-time Next.js dashboard.

**Project status:** Planning / initial development. The architecture and feature set below describe the intended implementation; features should be marked complete as they are built and tested.

---

## Overview

NetSentinel aims to provide a single interface for observing network activity:

- Capture traffic from an authorized network interface or analyze a PCAP file.
- Decode common network protocols and extract useful metadata.
- Group packets into bidirectional network flows.
- Monitor bandwidth, connection activity, and protocol distribution.
- Inspect DNS queries and visible HTTP/TLS metadata.
- Detect suspicious patterns using configurable, rule-based heuristics.
- Explore packets, flows, and alerts through a live web dashboard.

The project is intended for learning, experimentation, and monitoring networks you own or are authorized to inspect. It is not intended to decrypt encrypted traffic or replace a production-grade IDS.

## Planned Features

### 1. Packet Explorer
- Live packet capture and offline PCAP analysis.
- Ethernet, IPv4, IPv6, TCP, UDP, and ICMP parsing.
- Packet search and protocol filters.
- Packet metadata and protocol-layer inspection.
- Export of analyzed capture data.

### 2. Deep Packet Inspection
- Application-protocol identification.
- DNS query metadata extraction.
- Basic HTTP metadata parsing where traffic is unencrypted.
- TLS ClientHello metadata, including SNI when visible.
- TLS version, ALPN, and cipher-suite metadata where available.

> DPI is metadata-oriented. NetSentinel will not attempt to decrypt TLS payloads. Encrypted ClientHello (ECH), packet loss, fragmentation, and incomplete captures can limit the metadata available to passive inspection.

### 3. Flow Analyzer
- Bidirectional five-tuple flow tracking.
- Source/destination IP addresses and ports.
- Transport and detected application protocols.
- Packet and byte counters in both directions.
- Flow start time, last-seen time, and duration.
- Active-flow tracking and idle-flow expiration.
- Top talkers and protocol distribution.

### 4. DNS Intelligence
- DNS query and response metadata.
- Domain and query-type statistics.
- NXDOMAIN monitoring.
- Detection heuristics for unusually long or high-volume DNS queries.
- Suspicious-domain indicators.

### 5. Threat Detection
Initial rule-based detections are planned for:
- Port-scan-like connection patterns.
- Unusual TCP SYN rates.
- DNS activity anomalies.

Alerts will include a severity, source information, timestamp, description, and supporting metadata. These heuristics are indicators for investigation, not proof that an attack has occurred.

### 6. Dashboard and Reports
- Live network overview.
- Packet Explorer.
- Flow Analyzer.
- DNS Intelligence.
- Threat and alert management.
- Detection-rule configuration.
- Historical traffic statistics and reports.

---

## Architecture

The application is split into independent components:

```text
                 Network Interface / PCAP
                           |
                           v
                  C++ Packet Engine
             libpcap · Parsing · DPI · Flows
                           |
                    JSON Lines / IPC
                           |
                           v
                    Python Backend
             FastAPI · Event Processing
             Flow Aggregation · Detection
                    /             \
                   v               v
             Supabase          WebSocket
             PostgreSQL            |
                   |               |
                   +-------+-------+
                           |
                           v
                    Next.js Dashboard
             Overview · Packets · Flows
                DNS · Threats · Reports
```

### Component responsibilities

| Component | Responsibility |
|---|---|
| C++ network engine | Packet capture, protocol parsing, metadata extraction, and flow tracking |
| Python backend | Engine lifecycle, event validation, aggregation, detection rules, and API endpoints |
| Supabase PostgreSQL | Persistent sessions, flow summaries, DNS metadata, alerts, rules, and traffic aggregates |
| WebSocket layer | Push selected live events and metric updates to connected clients |
| Next.js frontend | Dashboard, filters, packet/flow inspection, and alert management |

The C++ engine should remain independently executable. The backend communicates with it through structured JSON Lines over a managed subprocess initially; this keeps the first implementation simple and easy to debug.

**Data-volume strategy:** Raw packets should not be inserted individually into PostgreSQL. The engine/backend should aggregate traffic and persist flow summaries, DNS metadata, alerts, and periodic traffic statistics. PCAP files and a bounded recent-packet buffer are better suited to packet-level inspection.

---

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js, React, TypeScript |
| UI | Tailwind CSS, shadcn/ui |
| Charts | Recharts |
| Backend | Python, FastAPI, Pydantic |
| Packet engine | C++17 |
| Packet capture | libpcap |
| Database | Supabase PostgreSQL |
| Authentication | Supabase Auth |
| Live updates | WebSockets |
| Build system | CMake |
| Testing | GoogleTest, Pytest, Playwright |
| Local deployment | Docker Compose |

---

## Planned Dashboard Pages

| Page | Purpose |
|---|---|
| Overview | Packet volume, bandwidth, active connections, alerts, traffic charts, and top talkers |
| Packets | Searchable packet table with selected-packet details and protocol hierarchy |
| Flows | Connection table, byte/packet counts, duration, top talkers, and protocol distribution |
| DNS | Query history, popular domains, response codes, and anomaly indicators |
| Threats | Alert list, severity, evidence, and alert status |
| Rules | Enable, disable, and configure supported detection thresholds |
| Reports | Historical traffic and security summaries |
| Settings | Capture preferences and application configuration |

---

## Repository Structure

The intended repository layout is:

```text
Network-Inspector/
├── frontend/                 # Next.js dashboard
│   ├── app/
│   ├── components/
│   ├── hooks/
│   ├── lib/
│   └── types/
│
├── backend/                  # FastAPI application
│   ├── app/
│   │   ├── api/
│   │   ├── database/
│   │   ├── detection/
│   │   ├── engine/
│   │   ├── schemas/
│   │   ├── services/
│   │   └── workers/
│   └── tests/
│
├── network-engine/           # C++ packet processing
│   ├── include/
│   │   ├── capture/
│   │   ├── flow/
│   │   ├── models/
│   │   └── parser/
│   ├── src/
│   │   ├── capture/
│   │   ├── flow/
│   │   ├── output/
│   │   └── parser/
│   └── tests/
│
├── database/
│   ├── migrations/
│   └── seed.sql
│
├── test-data/
│   ├── pcaps/
│   └── generate_traffic.py
│
├── docker-compose.yml
├── .env.example
└── README.md
```

This is the target structure; directories will be added as the corresponding modules are implemented.

---

## Getting Started

The complete setup instructions will be added as the components become available. The planned development environment requires:

- C++17 compiler
- CMake
- libpcap
- Python 3.11+
- Node.js LTS
- Supabase project and database credentials
- Git

### Environment configuration

A root `.env.example` will document the required configuration. Expected values include:

```env
# Backend
API_HOST=127.0.0.1
API_PORT=8000

# Supabase — keep service-role credentials on the backend only
SUPABASE_URL=
SUPABASE_SERVICE_ROLE_KEY=

# Frontend
NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
```

Never commit real credentials or capture data containing private information. The Supabase service-role key must never use a `NEXT_PUBLIC_` prefix or be exposed to browser code.

---

## API Plan

The backend API will use the base path `/api/v1`.

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/capture/interfaces` | List capture interfaces |
| POST | `/capture/start` | Start an authorized live capture |
| POST | `/capture/stop` | Stop the active capture |
| GET | `/capture/status` | Get capture status |
| POST | `/capture/upload` | Submit a PCAP for offline analysis |
| GET | `/capture/sessions` | List capture sessions |
| GET | `/analytics/overview` | Dashboard summary |
| GET | `/analytics/traffic` | Traffic time series |
| GET | `/analytics/top-talkers` | Highest-volume endpoints |
| GET | `/flows` | Search and list flows |
| GET | `/flows/{id}` | Flow details |
| GET | `/dns/queries` | DNS query history |
| GET | `/dns/top-domains` | Most requested domains |
| GET | `/alerts` | List security alerts |
| PATCH | `/alerts/{id}` | Update alert status |
| GET/POST | `/rules` | List or create detection rules |
| PATCH/DELETE | `/rules/{id}` | Update or delete a rule |

A WebSocket endpoint at `/ws/live` is planned for traffic updates, new flows, capture status, and alert notifications. All user-specific API and WebSocket data must be authenticated and authorized.

---

## Development Roadmap

The project will be developed incrementally:

- [ ] **Phase 1 — Packet capture:** CMake setup, libpcap integration, interface listing, and Ethernet/IP/TCP/UDP/ICMP parsing.
- [ ] **Phase 2 — DPI and flows:** DNS, HTTP, TLS metadata extraction, bidirectional flow tracking, and PCAP input.
- [ ] **Phase 3 — Backend and database:** FastAPI, C++ process management, event ingestion, Supabase schema, and REST APIs.
- [ ] **Phase 4 — Detection:** Port-scan heuristics, SYN-rate monitoring, DNS anomaly rules, and alert management.
- [ ] **Phase 5 — Frontend:** Authentication, dashboard layout, packet explorer, flow analyzer, DNS, and threats pages.
- [ ] **Phase 6 — Live updates and release:** WebSockets, integration tests, Docker Compose, documentation, and demo.

---

## Testing

Testing will use generated and controlled traffic rather than relying only on live network activity.

- **C++:** Unit tests for protocol parsers, flow keys, counters, and expiration.
- **PCAP fixtures:** Normal traffic and controlled test scenarios.
- **Python:** Unit and integration tests for event processing and detection rules.
- **API:** FastAPI endpoint and authorization tests.
- **Frontend:** Component and end-to-end tests.
- **WebSockets:** Connection authentication, event delivery, and disconnect handling.

Malformed, truncated, fragmented, and unexpected packets should be treated as untrusted input. Parsers must validate captured lengths before reading headers or payloads.

---

## Security and Privacy

- Capture only traffic on systems and networks you own or are authorized to monitor.
- Run packet capture with the minimum privileges required by the operating system.
- Restrict capture interfaces and validate capture filters.
- Treat packet contents, DNS queries, and IP addresses as potentially sensitive data.
- Authenticate API and WebSocket clients.
- Enable Row Level Security on user-owned Supabase tables.
- Keep service-role credentials on the backend.
- Apply retention limits to stored network metadata and uploaded PCAP files.

---

## Learning Goals

Building NetSentinel is intended to provide practical experience with:

- Network layers, Ethernet, IP, TCP, UDP, DNS, and TLS.
- Packet capture and protocol parsing.
- Five-tuple flow tracking and traffic aggregation.
- C++ systems programming and memory-safe parsing practices.
- Python asynchronous processing and API design.
- Real-time communication with WebSockets.
- PostgreSQL schema design and Row Level Security.
- Full-stack application architecture, testing, and deployment.

---

## License

A license has not yet been selected. Until one is added, all rights are reserved by the repository owner.

---

**NetSentinel — Understand your network, one packet at a time.**
