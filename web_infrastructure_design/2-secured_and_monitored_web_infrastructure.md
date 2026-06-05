## Objective
Add security and observability to the distributed infrastructure:

3 firewalls to segment and protect the network

SSL certificate to encrypt traffic (HTTPS)

3 monitoring agents to watch each server (LB, S1, S2)

## Key concepts
Role of firewalls: filter traffic and only allow what is strictly necessary

HTTPS: TLS encryption between client and server

Monitoring agent: program that collects metrics/logs and sends them to a monitoring platform

SSL termination: where HTTPS is decrypted (at the Load Balancer vs on each server)

MySQL single Primary limitation: if the Primary fails, no more writes are possible

Uniform stack issue: all components on the same server are tightly coupled

##  Whiteboard reasoning
text
Security:
- FW 1: in front of the Load Balancer → only allows incoming 80/443
- FW 2: in front of Server 1 → only allows traffic from LB + MySQL replication
- FW 3: in front of Server 2 → same as Server 1

Encryption:
- SSL certificate installed on the Load Balancer (e.g. Let's Encrypt)
- LB terminates SSL and forwards HTTP internally (risk) OR re-encrypts (ideal)
- Alternative: SSL on each server (end-to-end encryption)

Monitoring:
- One monitoring agent on each server (LB, S1, S2)
- Collects: CPU, RAM, disk, network, Nginx QPS, app logs, MySQL slow queries
- Sends data to a cloud monitoring platform
- Alerts configured (email, Slack, SMS)

Problems to explain:
1. SSL termination at the LB = internal traffic is not encrypted
2. Single MySQL Primary = SPOF for writes (no automatic failover)
3. Uniform stack = same software everywhere, one bug can affect all servers

## Imgur link : https://imgur.com/a/rGAXIqd

## Mermaid diagram :
```` mermaid
graph TD
    User["👤 User"]
    Internet["🌍 Internet<br>(HTTPS)"]

    subgraph FW1_Zone["🛡️ FIREWALL 1<br>Allows: IN 443, IN 80 (redirect)<br>Blocks: everything else"]
        HAProxy["⚖️ HAProxy<br>Load Balancer<br>+ SSL Termination<br>🔒 SSL Certificate<br>www.foobar.com"]
        Monitor_LB["📊 Monitoring Agent<br>(LB)"]
    end

    subgraph FW2_Zone["🛡️ FIREWALL 2<br>Allows: IN 80 (from LB)<br>IN 3306 (replication)<br>Blocks: everything else"]
        subgraph S1["🖥️ SERVER 1"]
            Nginx1["🔵 Nginx"]
            App1["🟢 Application Server"]
            MySQL1["🗄️ MySQL PRIMARY"]
            Monitor1["📊 Monitoring Agent<br>(Server 1)"]
        end
    end

    subgraph FW3_Zone["🛡️ FIREWALL 3<br>Allows: IN 80 (from LB)<br>IN 3306 (replication)<br>Blocks: everything else"]
        subgraph S2["🖥️ SERVER 2"]
            Nginx2["🔵 Nginx"]
            App2["🟢 Application Server"]
            MySQL2["🗄️ MySQL REPLICA"]
            Monitor2["📊 Monitoring Agent<br>(Server 2)"]
        end
    end

    SumoLogic["☁️ Monitoring Platform<br>Dashboards + Alerts"]

    User -->|"HTTPS 443"| Internet
    Internet --> FW1_Zone
    HAProxy -->|"HTTP (internal)"| Nginx1
    HAProxy -->|"HTTP (internal)"| Nginx2
    Nginx1 --> App1 --> MySQL1
    Nginx2 --> App2 --> MySQL2
    MySQL1 -->|"Replication (binlog)"| MySQL2

    Monitor_LB -->|"LB metrics/logs"| SumoLogic
    Monitor1 -->|"Server 1 metrics/logs"| SumoLogic
    Monitor2 -->|"Server 2 metrics/logs"| SumoLogic

    style FW1_Zone fill:#ffebee,stroke:#c62828,stroke-dasharray: 5 5
    style FW2_Zone fill:#ffebee,stroke:#c62828,stroke-dasharray: 5 5
    style FW3_Zone fill:#ffebee,stroke:#c62828,stroke-dasharray: 5 5
    style SumoLogic fill:#e8eaf6,stroke:#283593
````

## Diagram explanation
 #### Why firewalls:

Firewalls reduce the attack surface. Without them, if an attacker knows a server’s IP, they can try to connect to any port (SSH, MySQL, etc.).

With firewalls:

- FW1 only exposes ports 80 and 443 to the Internet.

- FW2 and FW3 only accept traffic from the Load Balancer and MySQL replication.

- MySQL (port 3306) is never directly reachable from the Internet.

#### Why HTTPS:

Without HTTPS, all data (passwords, personal info, session cookies) travels in clear text.

Anyone on the same network (e.g. public Wi‑Fi) can sniff and read it (Man‑in‑the‑Middle attack).

HTTPS encrypts all communication between the user and the Load Balancer.

#### Why monitoring:

Monitoring allows you to:

- Detect failures proactively (before users complain)

- Identify bottlenecks (CPU at 95%, disk full, DB too slow)

- Track performance trends over time

- Have evidence for incident analysis and post‑mortems

#### How monitoring agents work:

A small agent runs on each server and continuously reads:

- System metrics: CPU, RAM, disk, network

- Nginx logs (access/error logs)

- Application logs

- MySQL metrics and slow queries

It sends all this data securely (often via HTTPS) to the monitoring platform, which builds dashboards and triggers alerts.


### Typical issues to highlight:
---

- SSL termination at the LB	:

HTTPS is decrypted at the Load Balancer, then HTTP is used internally. If someone gains access to the internal network, they can see traffic in clear text. A more secure option is to use HTTPS between LB and backend servers as well.


- Single MySQL Primary :

If the Primary database fails, no more writes are possible (user registration, orders, updates…). Reads may still work from the Replica, but the application must handle this. Proper failover requires extra tooling and orchestration.

- Uniform stack :

All servers run the same stack (Nginx + App + MySQL). A critical bug or misconfiguration in one component can affect all servers at once. Also, resource contention on one service (e.g. MySQL using too much RAM) impacts the others on the same machine.
