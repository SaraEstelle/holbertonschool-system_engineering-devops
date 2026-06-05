## Objective
Design a distributed 3‑server web infrastructure to remove the most critical SPOFs and allow horizontal scaling.

Understand the role of the Load Balancer, MySQL replication, and Active‑Active vs Active‑Passive modes.

## Key concepts
HAProxy: widely used open‑source load balancer

Round Robin algorithm: distributes requests in a circular fashion

Active‑Active vs Active‑Passive: two redundancy strategies

Primary‑Replica (Master‑Slave): MySQL replication architecture

Primary vs Replica: Primary = read + write, Replica = read‑only

Remaining SPOFs: the Load Balancer itself is a SPOF in this design

## Whiteboard reasoning
text
Problems from Task 0 to solve:
1. Server SPOF → Solution: add a second server + Load Balancer
2. No scalability → Solution: Load Balancer distributes the load
3. Non‑redundant DB → Solution: MySQL Primary + Replica

New problems introduced:
1. The Load Balancer becomes a SPOF → solved in Task 3
2. No encryption (HTTP only) → solved in Task 2
3. No monitoring → solved in Task 2
4. No firewall → solved in Task 2

Architecture decisions:
- Load Balancer: HAProxy with Round Robin
- Mode: Active‑Active (both web/app servers receive traffic)
- Database: Primary‑Replica (Server 1 = Primary, Server 2 = Replica)
- Application: same code base deployed on Server 1 and Server 2

## Imgur link : https://imgur.com/a/LXcOMVK 

## Mermaid diagram
````mermaid
graph TD
    User["👤 User<br>www.foobar.com"]
    DNS["🌐 DNS<br>www.foobar.com → LB IP"]

    subgraph LB_Zone["Load Balancer Zone"]
        HAProxy["⚖️ HAProxy<br>Load Balancer<br>Round Robin<br>Active-Active"]
    end

    subgraph S1["🖥️ SERVER 1 (Primary)"]
        Nginx1["🔵 Nginx<br>Web Server"]
        App1["🟢 Application Server<br>+ Code Base"]
        MySQL1["🗄️ MySQL<br>PRIMARY<br>Read + Write"]
        Nginx1 --> App1
        App1 --> MySQL1
    end

    subgraph S2["🖥️ SERVER 2 (Replica)"]
        Nginx2["🔵 Nginx<br>Web Server"]
        App2["🟢 Application Server<br>+ Code Base"]
        MySQL2["🗄️ MySQL<br>REPLICA<br>Read Only"]
        Nginx2 --> App2
        App2 --> MySQL2
    end

    MySQL1 -->|"Replication<br>binlog"| MySQL2

    User -->|"1. DNS Lookup"| DNS
    DNS -->|"2. LB IP"| User
    User -->|"3. HTTP Request"| HAProxy
    HAProxy -->|"Req 1,3,5…<br>(Round Robin)"| Nginx1
    HAProxy -->|"Req 2,4,6…<br>(Round Robin)"| Nginx2

    style LB_Zone fill:#fff9c4,stroke:#f9a825
    style S1 fill:#e8f5e9,stroke:#2e7d32
    style S2 fill:#e3f2fd,stroke:#1565C0
    style MySQL1 fill:#ffe0b2,stroke:#e65100
    style MySQL2 fill:#f3e5f5,stroke:#6a1b9a
````
## Diagram explanation
### Why each component is added:

#### HAProxy (Load Balancer):

- Receives all HTTP traffic on a single public IP.

- Distributes requests between Server 1 and Server 2 using Round Robin:

  - Request 1 → S1, Request 2 → S2, Request 3 → S1, etc.

- Can perform health checks: if S1 is down, it stops sending traffic to S1.

#### Server 1 and Server 2:

- Each has Nginx + Application Server + the same code base.

- This doubles the processing capacity.

- Active‑Active mode: both servers receive traffic at the same time.

  - Advantage: better load distribution and higher availability.

  - Compared to Active‑Passive: in Active‑Passive, S2 only takes traffic if S1 fails.

#### MySQL Primary‑Replica:

- Primary (on S1): handles both reads and writes (SELECT, INSERT, UPDATE, DELETE).

- Replica (on S2): handles read‑only queries (SELECT).

- Replication: every write on the Primary is propagated to the Replica via the binary log.

- Application impact: the app can send writes to the Primary and distribute reads across Primary + Replica.

#### Remaining problems in this architecture:

- SPOF: HAProxy :	If the Load Balancer fails, the whole infrastructure becomes unreachable.

- No firewall: All services are potentially exposed; no network‑level filtering.

- No HTTPS : Traffic is unencrypted; sensitive data can be intercepted.

- No monitoring	: No visibility on server health, load, or failures.
