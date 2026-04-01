# Design a Load Balancer

A Load Balancer is a critical infrastructure component that distributes incoming network traffic across multiple servers to ensure no single server becomes overwhelmed.

> Problem: 
>   - instead of routing all traffic to one server (which would eventually crash under heavy load),
>   - a load balancer acts as a traffic cop, intelligently spreading requests across a pool of healthy servers. 
>   - This improves application availability, responsiveness, and overall system reliability.


## Requirements

> `Candidate`: "What type of traffic should the load balancer handle? Are we focusing on HTTP/HTTPS traffic, or should it support any TCP/UDP traffic?"
> 
> `Interviewer`: "Let's design a general-purpose load balancer that supports both Layer 4 (TCP/UDP) and Layer 7 (HTTP/HTTPS) load balancing."

> `Candidate`: "What is the expected scale? How many requests per second should the system handle?"
> 
> `Interviewer`: "The load balancer should handle up to 1 million requests per second at peak traffic."

> `Candidate`: "How should we handle server failures? Should the load balancer automatically detect and route around unhealthy servers?"
> 
> `Interviewer`: "Yes, health checking is critical. The system should detect failures within seconds and stop routing traffic to unhealthy servers."

> `Candidate`: "Do we need to support session persistence, where requests from the same client go to the same backend server?"
> 
> `Interviewer`: "Yes, sticky sessions should be supported for stateful applications, but it should be configurable."

> `Candidate`: "What about SSL/TLS termination? Should the load balancer handle encryption?"
> 
> `Interviewer`: "Yes, SSL termination at the load balancer is required to offload encryption work from backend servers."

> `Candidate`: "What availability target should we aim for?"
> 
> `Interviewer`: "The load balancer itself must be highly available with 99.99% uptime, since it's on the critical path for all traffic."


### Functional Requirements

- **Traffic Distribution**: Distribute incoming requests across multiple backend servers using configurable algorithms.
- **Health Checking**: Continuously monitor backend servers and automatically remove unhealthy ones from the pool.
- **Session Persistence**: Support sticky sessions to route requests from the same client to the same server.
- **SSL Termination**: Handle SSL/TLS encryption and decryption to offload work from backend servers.
- **Layer 4 and Layer 7 Support**: Support both transport-level (TCP/UDP) and application-level (HTTP/HTTPS) load balancing.

### Non Functional Requirements
- **High Availability**: The load balancer must be highly available (99.99% uptime) with no single point of failure.
- **Low Latency**: Should add minimal latency to requests (< 1ms overhead).
- **High Throughput**: Handle up to 1 million requests per second at peak.
- **Scalability**: Should scale horizontally to handle increasing traffic.
- **Fault Tolerance**: Continue operating even when individual components fail.

---

## Estimations

### 1. Traffic 
| Category | Metric                 | Value     | Notes                  |
|----------|------------------------|-----------|------------------------|
| Traffic  | Peak RPS               | 1,000,000 | Maximum load           |
| Traffic  | Average RPS            | ~300,000  | ~1/3 of peak           |
| Traffic  | Concurrent Connections | ~500,000  | Derived using duration |


### 2. Concurrent Connections Calculation

| Parameter            | Value                                   |
|----------------------|-----------------------------------------|
| Avg Request Duration | 0.5 sec (500 ms)                        |
| Formula              | Concurrent Connections = RPS × Duration |
| Calculation          | 1,000,000 × 0.5                         |
| Result               | 500,000 connections                     |

### 3. Bandwidth Estimates

| Type              | Formula     | Value              |
|-------------------|-------------|--------------------|
| Request Size      | —           | 2 KB               |
| Response Size     | —           | 10 KB              |
| Ingress Bandwidth | 1M × 2 KB   | 2 GB/s             |
| Egress Bandwidth  | 1M × 10 KB  | 10 GB/s            |
| Total Bandwidth   | 2 + 10 GB/s | 12 GB/s (~96 Gbps) |


### 4. Health Check Overhead

| Parameter       | Value              |
|-----------------|--------------------|
| Backend Servers | 1000               |
| Check Interval  | 5 sec              |
| Formula         | Servers / Interval |
| Result          | 200 checks/sec     |
| Impact          | Negligible         |

### 5. Memory Requirements (Per Connection)

| Component                | Size           |
|--------------------------|----------------|
| Source IP + Port         | 6 bytes        |
| Destination IP + Port    | 6 bytes        |
| Connection State         | 4 bytes        |
| Timestamps               | 16 bytes       |
| Protocol Data            | ~200 bytes     |
| Buffer Space             | ~250 bytes     |
| **Total per Connection** | **~500 bytes** |


### Summary

| Area          | Observation                 | Impact                         |
|---------------|-----------------------------|--------------------------------|
| Network       | ~12 GB/s (~96 Gbps)         | 🚨 Primary bottleneck          |
| Memory        | ~250 MB                     | ✅ Not a concern                |
| Health Checks | 200 RPS                     | ✅ Negligible                   |
| Scalability   | Single machine insufficient | 🚀 Requires horizontal scaling |

---

## Core APIs

| Plane             | Purpose                            | Characteristics              | API requirement        |
|-------------------|------------------------------------|------------------------------|------------------------|
| **Data Plane**    | Handles actual request traffic     | High throughput, low latency | No                     |
| **Control Plane** | Handles configuration & management | Low traffic, high importance | Explicit APIs required |

![img.png](images/plane-architecture.png)

## APIs for Control Plane

### 1. Register Backend Server

Endpoint: `POST /backends`

- When you spin up a new application server, you need to tell the load balancer about it. 
- This endpoint adds a backend to the pool and starts health checking it.

![img.png](images/backend-server-register-api.png)

> The initial status is "unknown" because we have not run a health check yet. Within a few seconds, it will transition to "healthy" or "unhealthy."


### 2. Remove Backend Server

Endpoint: `DELETE /backends/{backend_id}`

- When you want to decommission a server (for maintenance, scaling down, or replacement), this endpoint removes it from the pool. 
- The load balancer will stop sending new traffic immediately, but existing connections are allowed to complete gracefully.
- path params: `backend_id` - the ID returned when the backend was registered

![img.png](images/backend-server-remove-api.png)

> The `drained_connections` field tells you how many connections were in progress when the backend was removed.

### 3. Get Backend Health Status

Endpoint: `GET /backends/{backend_id}/health`

- Returns detailed health information about a specific backend, useful for debugging and monitoring dashboards.

![img.png](images/health-check-api.png)

> The response includes both current state (status, active connections) and historical metrics (total requests, failures) to help operators understand backend performance over time.


### 4. Configure Load Balancing Algorithm

Endpoint: `PUT /config/algorithm`

- Changes how traffic is distributed across backends. 
- This takes effect immediately for new connections (existing connections are not affected).

![img.png](images/load-balancing-configure-algo-api.png)


---

## High Level Design

- Fundamental operations
  - **Distribute Traffic**: Accept incoming requests and forward them to healthy backend servers.
  - **Monitor Health**: Continuously check backends and route around failures.
  - **Stay Available**: The load balancer itself cannot be a single point of failure.

- The architecture will split into two parts
  - Data plane - handles actual traffic at high speed
  - Control plane - manages configuration and health checking at a slower pace.

