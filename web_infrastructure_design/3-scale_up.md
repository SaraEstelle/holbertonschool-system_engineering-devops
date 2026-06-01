## Objective :

Industrialize the infrastructure by:

Splitting each component (Web Server, App Server, DB) onto dedicated servers

Putting the Load Balancers in a high‑availability cluster to remove that SPOF

Understanding why separation of responsibilities is a best practice

## Key concepts :

Separation of responsibilities: each server does one thing and does it well

HAProxy cluster: 2 Load Balancers in Active‑Passive mode using Keepalived/VRRP

Virtual IP (VIP): floating IP address managed by the LB cluster

VRRP (Virtual Router Redundancy Protocol): failover protocol for the VIP

Horizontal vs vertical scaling: adding more servers vs making one server bigger

## Whiteboard reasoning :
```` text
Why separate components?

BEFORE (uniform stack on 1 server):
┌────────────────────────┐
│  Nginx + App + MySQL   │  ← All share CPU/RAM/Disk
│  If MySQL eats RAM     │  → Nginx becomes slow
│  If App crashes        │  → Nginx is impacted too
│  Scaling?              │  → You scale EVERYTHING at once (expensive)
└────────────────────────┘

AFTER (separated components):
┌──────────┐  ┌──────────┐  ┌──────────┐
│  Nginx   │  │   App    │  │  MySQL   │
│ (Web)    │  │ Server   │  │  (DB)    │
│          │  │          │  │          │
│ Scale if │  │ Scale if │  │ Scale if │
│ traffic↑ │  │ CPU↑     │  │ data↑    │
└──────────┘  └──────────┘  └──────────┘

→ Each layer can be scaled INDEPENDENTLY
→ A MySQL crash does not consume Nginx memory
→ Hardware profiles can be tuned per role (I/O for DB, CPU for App, network for Web)

Why 2 Load Balancers in a cluster?

- In Task 2, the LB was still the final SPOF
- Solution: 2 HAProxy instances + Keepalived
- 1 VIP (Virtual IP) shared between both LBs
- If LB1 fails, LB2 takes over the VIP in < 2 seconds
- For users: no visible interruption
````
## Mermaid diagram :
````mermaid
graph TD
    User["👤 User<br>www.foobar.com"]
    DNS["🌐 DNS<br>www → VIP<br>(Virtual IP)"]

    subgraph LB_Cluster["⚖️ HAProxy CLUSTER<br>(Keepalived + VRRP)"]
        VIP["🔄 Virtual IP (VIP)<br>Public floating IP"]
        LB1["HAProxy 1<br>🟢 ACTIVE<br(holds VIP)"]
        LB2["HAProxy 2<br>🟡 PASSIVE<br>(standby)"]
        VIP -->|"Normal traffic"| LB1
        LB1 <-->|"VRRP heartbeats<br>state + health"| LB2
    end

    subgraph Web_Layer["🔵 WEB LAYER (Web Servers)"]
        Nginx1["Nginx 1<br>Static files + Proxy"]
        Nginx2["Nginx 2<br>Static files + Proxy"]
    end

    subgraph App_Layer["🟢 APPLICATION LAYER (App Servers)"]
        App1["App Server 1<br>Business logic<br>Code base"]
        App2["App Server 2<br>Business logic<br>Code base"]
    end

    subgraph DB_Layer["🗄️ DATABASE LAYER"]
        MySQL1["MySQL PRIMARY<br>Read + Write"]
        MySQL2["MySQL REPLICA<br>Read Only"]
        MySQL1 -->|"Replication"| MySQL2
    end

    User --> DNS
    DNS --> VIP
    LB1 --> Nginx1
    LB1 --> Nginx2
    Nginx1 --> App1
    Nginx2 --> App2
    App1 --> MySQL1
    App2 --> MySQL1
    App1 -.->|"Read-only queries"| MySQL2
    App2 -.->|"Read-only queries"| MySQL2

    style LB_Cluster fill:#fff9c4,stroke:#f9a825,stroke-width:2px
    style Web_Layer fill:#e3f2fd,stroke:#1565C0,stroke-width:2px
    style App_Layer fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style DB_Layer fill:#fce4ec,stroke:#c62828,stroke-width:2px
````

##  Diagram explanation

#### Dedicated Web Servers (Nginx):

  - Nginx is optimized for handling many concurrent HTTP connections.

  - On its own tier, it can use hardware tuned for network throughput and concurrency.

  - We can scale the web layer independently (e.g. add Nginx3 during traffic spikes).

#### Dedicated App Servers:

  - The application layer is CPU/RAM intensive (business logic, template rendering, etc.).

  - Isolating it from Nginx means app bugs (memory leaks, CPU spikes) don’t directly impact the web tier.

  - We can deploy new application versions without touching Nginx or the database.

#### Dedicated Database Servers (MySQL):

  - MySQL is I/O and memory heavy (buffer pool, indexes, caches).

  - On a dedicated server, you can choose hardware optimized for storage (fast SSDs, lots of RAM).

  - Failures or heavy load on the DB do not directly affect web or app processes.

#### HAProxy cluster (2 Load Balancers):

  - Both LBs share a Virtual IP (VIP) managed by Keepalived using VRRP.

  - LB1 (ACTIVE) owns the VIP and handles all traffic in normal conditions.

  - LB2 (PASSIVE) monitors LB1 via VRRP heartbeats and is ready to take over.

  - If LB1 fails, LB2 grabs the VIP in under a couple of seconds → failover is transparent to users.

  - DNS always points to the VIP, not to LB1 or LB2 individually, so no DNS change is needed during failover.

#### Future scalability:

This modular architecture makes it easy to:

  - Add a 3rd Nginx behind the LB → more web capacity

  - Add a 3rd App Server → more compute capacity

  - Add another MySQL Replica → more read capacity