# Web Infrastructure Design

This project focuses on designing and explaining different web infrastructures used to host a website. The objective is to understand how web servers, application servers, databases, load balancers, firewalls, monitoring systems, and scaling strategies work together to provide availability, security, and performance.

All diagrams were designed using Draw.io.

---

## Learning Objectives

At the end of this project, I should be able to explain without the help of Google:

- What is a server
- What is the role of a domain name
- What DNS is and how it works
- What a web server does
- What an application server does
- What a database does
- What a load balancer does
- What HTTPS is and why it is important
- What a firewall is
- What monitoring is used for
- What are SPOFs (Single Points of Failure)
- How scaling improves availability and performance

---

# Tasks

## 0. Simple Web Stack

### Infrastructure

A simple web infrastructure hosting **www.foobar.com** on a single server.

### Components

- Domain name: `foobar.com`
- DNS A Record:
  - `www -> 8.8.8.8`
- One server containing:
  - Nginx Web Server
  - Application Server
  - Application Files (Codebase)
  - MySQL Database

### Request Flow

1. User enters `www.foobar.com`
2. DNS resolves the domain name to `8.8.8.8`
3. The request reaches the server through TCP/IP
4. Nginx receives the request
5. The application server processes business logic
6. MySQL provides or stores data
7. The response is returned to the user

### Issues

- Single Point of Failure (SPOF)
- Downtime during maintenance or deployment
- Cannot scale under high traffic

---

## 1. Distributed Web Infrastructure

### Infrastructure

A distributed architecture using multiple servers and a load balancer.

### Components

- Domain name: `foobar.com`
- DNS A Record
- HAProxy Load Balancer
- Two application servers:
  - Nginx
  - Application Server
  - Application Files
- MySQL Primary-Replica replication

### Load Balancer Configuration

- HAProxy
- Round Robin algorithm
- Active-Active setup

### Request Flow

1. User accesses `www.foobar.com`
2. DNS resolves the domain
3. Request reaches HAProxy
4. HAProxy distributes traffic between backend servers
5. Servers process requests
6. Database replication synchronizes data

### Primary vs Replica

#### Primary

- Read operations
- Write operations

#### Replica

- Read operations only
- Receives replicated data from Primary

### Remaining Issues

#### SPOF

- DNS
- Load Balancer
- Primary Database

#### Security Issues

- No firewall
- No HTTPS

#### Monitoring Issues

- No monitoring

---

## 2. Secured and Monitored Web Infrastructure

### Infrastructure

A secured and monitored version of the distributed architecture.

### Additional Components

- 3 Firewalls
- SSL Certificate
- HTTPS
- 3 Monitoring Agents
- Monitoring Service (Sumologic or equivalent)

### Security Improvements

#### Firewalls

Protect servers by filtering incoming and outgoing traffic.

#### HTTPS

Encrypts communication between clients and infrastructure.

#### SSL Certificate

Used to authenticate the website and establish encrypted connections.

### Monitoring

Monitoring agents collect:

- CPU usage
- Memory usage
- Disk usage
- Application logs
- Nginx logs
- MySQL metrics

Collected data is sent to a monitoring platform.

### Monitoring QPS

To monitor web server Queries Per Second (QPS):

- Configure monitoring agent on Nginx
- Collect request metrics from Nginx logs or status module
- Send metrics to monitoring platform
- Create dashboards and alerts

### Issues

#### SSL Termination at Load Balancer

Traffic between HAProxy and backend servers is not encrypted.

#### Single Write Database

Only the Primary database accepts writes.

If the Primary fails:

- Writes stop working
- Application becomes partially unavailable

#### Mixed Server Responsibilities

Each backend server contains:

- Web Server
- Application Server
- Database

This makes:

- Scaling harder
- Maintenance more complex
- Resource isolation difficult

---

## 3. Scale Up

### Infrastructure

A highly available architecture with separated responsibilities.

### Components

#### Load Balancer Layer

- HAProxy #1
- HAProxy #2
- HA Cluster

#### Web Layer

- Dedicated Nginx Server

#### Application Layer

- Dedicated Application Server

#### Database Layer

- Dedicated MySQL Server

### Improvements

#### High Availability

Two HAProxy servers eliminate the load balancer SPOF.

#### Separation of Concerns

Each server has a dedicated role:

##### Web Server

Responsible for:

- Serving static files
- Reverse proxying requests

##### Application Server

Responsible for:

- Executing business logic
- Running application code

##### Database Server

Responsible for:

- Data persistence
- Query processing

#### Better Scalability

Each layer can be scaled independently:

- More Web Servers
- More Application Servers
- Database Replicas

#### Better Resource Isolation

CPU, memory, and storage resources are dedicated to specific services.

---

# Repository Structure

```text
web_infrastructure_design/
├── 0-simple_web_stack
├── 1-distributed_web_infrastructure
├── 2-secured_and_monitored_web_infrastructure
├── 3-scale_up
└── README.md
```

---

# Author

Project completed as part of the Holberton School System Engineering & DevOps curriculum.
