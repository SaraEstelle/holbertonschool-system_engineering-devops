# Simple web stack
## Objective :

Design a minimalist web infrastructure capable of hosting www.foobar.com.

Understand the role of each component in a LAMP‑style stack.

Identify the limitations (SPOF, maintenance downtime, scalability issues) of a single‑server architecture.

## Key Concepts :

* A DNS A record: maps www.foobar.com → 8.8.8.8

* IP 8.8.8.8: public IPv4 address of our single server

* Nginx: the web server receiving HTTP/HTTPS requests

* Application server: executes the application logic (Python, PHP, Node.js…)

* MySQL: persistent data storage

* SPOF: Single Point of Failure — if the server dies, everything dies

* HTTP/TCP protocol: how the browser communicates with the server

## Whiteboard Reasoning:

What happens when a user types www.foobar.com?

1. The user types www.foobar.com in Chrome
2. Chrome asks the DNS: “What is the IP of www.foobar.com?”
3. DNS looks up the A record → returns 8.8.8.8
4. Chrome opens a TCP connection to 8.8.8.8:80 (or 443)
5. The HTTP request reaches Nginx (port 80/443)
6. Nginx analyzes the request:
   - Static file → served directly
   - Dynamic content → forwarded to the application server
7. The application server executes the business logic
8. If data is needed → SQL query sent to MySQL
9. MySQL returns the data
10. The application server generates the HTML response
11. Nginx sends the HTTP response back to the browser
12. Chrome renders the page

- Identify the issues:

  - If the server goes down → total SPOF

  - Deploying new code → requires restarting Nginx → downtime

  - High traffic → the server becomes overloaded → no scalability


## Diagramm :
```mermaid
graph TD
    User["👤 User<br>www.foobar.com"]
    DNS["🌐 DNS<br>A Record<br>www → 8.8.8.8"]

    User -->|"1. DNS Lookup"| DNS
    DNS -->|"2. Returns IP 8.8.8.8"| User

    User -->|"3. HTTP/TCP Request<br>GET /"| Nginx

    subgraph Server["🖥️ Single Server<br>IP: 8.8.8.8"]
        Nginx["🔵 Nginx<br>Web Server<br>Port 80/443"]
        AppServer["🟢 Application Server<br>Runs Code Base"]
        MySQL["🗄️ MySQL<br>Database<br>Port 3306"]

        Nginx -->|"Dynamic request"| AppServer
        AppServer -->|"SQL Query"| MySQL
        MySQL -->|"Query Result"| AppServer
        AppServer -->|"Generated HTML"| Nginx
    end

    Nginx -->|"4. HTTP Response<br>200 OK + HTML"| User

    style Server fill:#eef7ff,stroke:#1e88e5,stroke-width:2px
    style Nginx fill:#e3f2fd,stroke:#1565C0
    style AppServer fill:#e8f5e9,stroke:#2e7d32
    style MySQL fill:#fff3e0,stroke:#ef6c00
    style DNS fill:#fce4ec,stroke:#c62828
````

## Explanation of the Diagram

Reading the diagram (top to bottom):

 1 - User → DNS :

The browser performs a DNS lookup for www.foobar.com.

The DNS server returns an A record pointing to 8.8.8.8.

 2 - User → Nginx :

The browser opens a TCP connection to the server and sends an HTTP request.

 3 - Nginx → Application Server :

Nginx handles the request.

  - Static files → served directly

  - Dynamic content → forwarded to the application server

 4 - Application Server → MySQL :

The application server queries the database when needed.

 5 - MySQL → Application Server → Nginx → User :

Data flows back up the chain until the final HTML response is sent to the user.


## Required Project Answers :


  - What is a server?

A physical or virtual machine that provides services and responds to network requests.

  - Role of the domain name?

Provides a human‑readable alias (www.foobar.com) for the server’s IP address.

  - Type of DNS record for www?

A record (maps a hostname to an IPv4 address).

  - Role of the web server (Nginx)?

Handles HTTP requests, serves static files, forwards dynamic requests to the application server.

  - Role of the application server?

Runs the application code and generates dynamic content.

  - Role of the database (MySQL)?

Stores and organizes persistent data.

  - How does the server communicate with the user?

Through the HTTP protocol over TCP (ports 80/443).

## Issues with This Infrastructure :

   - SPOF (Single Point of Failure) : 	One server → if it crashes, the entire website goes offline.

  - Maintenance downtime: Deploying new code or restarting Nginx causes temporary unavailability.

  - No scalability	: A single machine cannot handle large traffic spikes.
