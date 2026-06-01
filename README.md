# Web Infrastructure Design

## Description

This project explores the design of web infrastructures, from the simplest (a single server) to more robust architectures (distributed, secured, monitored, and highly available).
The goal is to understand the fundamental components of a modern web infrastructure and how they interact.

##
Author
Sara Rebati — Holberton School, 2026

## Concepts Covered

Network and web architecture

Web servers (Nginx) and application servers

DNS and record types

Load balancing (HAProxy) — Round Robin, Active‑Active/Passive

MySQL replication (Primary‑Replica)

Firewalls and network security

SSL/HTTPS and encryption

Monitoring and observability (Sumologic)

High availability and SPOF elimination

Horizontal scalability

##
Key Acronyms
Acronym	Meaning
LAMP	Linux, Apache/Nginx, MySQL, PHP/Python/Perl
SPOF	Single Point Of Failure
QPS	Queries Per Second
DNS	Domain Name System
LB	Load Balancer
VIP	Virtual IP
VRRP	Virtual Router Redundancy Protocol
SSL/TLS	Secure Sockets Layer / Transport Layer Security


## Project Structure
```` Code
web_infrastructure_design/
├── README.md
├── 0-simple_web_stack
├── 1-distributed_web_infrastructure
├── 2-secured_and_monitored_web_infrastructure
└── 3-scale_up
````
## Tasks

### Task 0 — Simple Web Stack
Single‑server infrastructure using a LAMP‑style stack (Nginx + App Server + MySQL).
Domain: www.foobar.com → A record → 8.8.8.8

Limitations: full SPOF, maintenance downtime, no scalability.


### Task 1 — Distributed Web Infrastructure
Three‑server infrastructure with HAProxy (Round Robin), two Nginx+App servers, and MySQL Primary‑Replica.

Mode: Active‑Active
Database: Primary (read+write) + Replica (read‑only, binlog replication)

Remaining limitations: LB is still a SPOF, no HTTPS, no firewall, no monitoring.


### Task 2 — Secured & Monitored Web Infrastructure
Added 3 firewalls, 1 SSL certificate (HTTPS), and 3 Sumologic monitoring agents.

Security: Firewalls isolate layers, HTTPS encrypts public traffic.
Monitoring: Agents collect CPU/RAM/Disk/QPS and send data to Sumologic.

Remaining limitations:

SSL termination at LB → internal traffic unencrypted

Single MySQL Primary → SPOF for writes

Uniform stack → poor component isolation


### Task 3 — Scale Up
Industrialized infrastructure with:

HAProxy cluster (2 LBs in Active‑Passive with VIP)

Fully separated layers: dedicated Web Servers, App Servers, and Database Servers

Advantages: independent scalability per layer, fault isolation, hardware optimized per role.



## Resources

HAProxy Documentation

MySQL Replication

Nginx Documentation

Sumologic

Let’s Encrypt SSL

Keepalived / VRRP

## Whiteboarding Procedure

For each task:

Read the requirements

List the necessary components and their roles

Draw the user flow (from URL to response)

Identify SPOFs and limitations

Draw the final diagram

Take a screenshot and upload it to Imgur

Insert the link in the task file

## License

Holberton School — Educational Project