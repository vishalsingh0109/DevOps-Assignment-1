# 🚀 DevOps Assignment – FastAPI + PostgreSQL (Dockerized)

## 👨‍💻 Student Details
- **Name:** Vishal Singh  
- **SAP ID:** 500125297  
- **Course:** B.Tech CSE CCVT B4  

---

## 📌 Project Overview
This project demonstrates a containerized web application using **FastAPI** as backend and **PostgreSQL** as database.  
Docker and Docker Compose are used to manage and run multiple services efficiently.

---

## ⚙️ Tech Stack
- FastAPI (Backend)
- PostgreSQL (Database)
- Docker (Containerization)
- Docker Compose (Orchestration)

---

## 🧠 Concepts Used
- Containerization
- Multi-container architecture
- Docker networking
- Persistent storage using volumes

---
## Network Design Diagram

Below is an ASCII representation of the architecture, illustrating how the isolated Docker containers communicate via the Macvlan network bridge directly mapping to the host's physical network interface.

```text
+-------------------------------------------------------------+
|               Host Machine / Physical Switch                |
|               (eth0 / Physical Interface)                   |
+-----------------------------+-------------------------------+
                              |
+-----------------------------v-------------------------------+
|               Macvlan Network (802.1q Bridge)               |
+--------------+-------------------------------+--------------+
               |                               |
     +---------v---------+           +---------v---------+
     |   FastAPI Backend |           |   PostgreSQL DB   |
     |    (Container)    |           |    (Container)    |
     | Unique MAC & IP   |           | Unique MAC & IP   |
     +---------+---------+           +---------+---------+
               |                               ^
               |           TCP / 5432          |
               +-------------------------------+
```

---

## ⚙️ Project Structure
DevOps-Assignment/
```
│
├── backend/
├── database/
├── docker-compose.yml
├── index.html
├── screenshots.html
├── report.pdf
└── README.md
```
 Image Size Comparison

When packaging Python applications that require database connectivity (like `psycopg2`), the difference in image size between a single-stage and multi-stage build is significant.

- **Standard Single-Stage Build:** A standard Dockerfile copying all requirements into a `python:3.11-slim` image alongside build-essential tools (`gcc`, `g++`, `make`) results in an image size typically ranging between **350MB to 500MB**. This image is bloated because it carries tools that a container does not need to execute the runtime application.
- **Optimized Multi-Stage Build:** By dropping the build environment layout in the first stage and utilizing only the pre-compiled `.whl` distributions in the second stage, the final image size dramatically shrinks down to approximately **120MB to 150MB**.

**Conclusion:** The multi-stage build decreases the storage footprint by over 60%. This significantly reduces network bandwidth costs during image pulls, speeds up deployment times in CI/CD pipelines, and minimizes the attack surface by ensuring fewer vulnerable system binaries are present.

---

## Macvlan vs. Ipvlan Comparison

Both **Macvlan** and **Ipvlan** are advanced Docker networking drivers utilized when containers must bridge directly to an underlying physical network segment, bypassing traditional Docker NAT (Network Address Translation) and port-forwarding.

### Macvlan
Macvlan operates at Layer 2 (Data Link Layer) and relies on the 802.1q trunking protocol. Sub-interfaces are spawned and attached to the parent adapter.
- **Mechanism:** Each container is assigned a **unique IP Address** *and* a **unique MAC Address**.
- **Perception:** To the external network setup, physical switches, and routers, the containers look like completely separate, physical network devices plugged uniquely into the switch.
- **Primary Use Case:** Best suited for legacy applications that demand direct broadcast domains, strict physical-like network presence, or environments where network hardware mandates standard 1:1 MAC-to-IP mappings.

### Ipvlan
Ipvlan abstracts the MAC address away and allows finer routing control. It can run in either Layer 2 (L2) or Layer 3 (L3) modes.
- **Mechanism:** All containers on the Ipvlan network share the single, underlying **parent host MAC address**, but they receive **unique IP Addresses**.
- **Perception:** To the external network, all container traffic appears to originate from the single host machine, despite bearing different IP headers.
- **Primary Use Case:** Mandatory in modern cloud environments (such as AWS VPC, Google Cloud, or strict VMware ESXi infrastructures). These cloud providers implement rigid security mechanisms (MAC spoofing protection or port-security) that aggressively drop traffic originating from unrecognized or multiple MAC addresses on a single switch port. Ipvlan seamlessly bypasses these limitations.

**Summary for Production:**
In an on-premise datacenter with full switch control and promiscuous mode active, **Macvlan** is preferred for hardware-level transparency. However, in AWS, Azure, or strict hypervisor environments where MAC filtering is enforced, **Ipvlan** is strictly necessary.
