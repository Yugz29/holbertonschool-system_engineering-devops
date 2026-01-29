## **0-simple_web_stack**

This infrastructure represents a simple, centralised web architecture that allows a user to access a site via a browser. It is based on a single server that hosts the web server, the application and the database, and communicates with the user via the Internet.

---

### **🔹 What is a server**

A server is a machine (physical or virtual) that provides services to other machines called clients.

Here, it hosts the website, executes the application logic, and stores the data.

### **🔹 What is the role of the domain name**

A domain name allows an IP address to be translated into a name that is readable by humans (ex: foobar.com).

It serves as an entry point for accessing the server.

### **🔹 What type of DNS record www is in www.foobar.com**

www is usually an A record (or sometimes a CNAME record) in the DNS.

*   **A record:** points directly to an IP address
    
*   **CNAME:** alias to another domain name

### **🔹 What is the role of the web server**

The web server (Apache, Nginx, etc.):

*   receives HTTP/HTTPS requests
    
*   serves static files (HTML, CSS, JS)
    
*   forwards dynamic requests to the application
    

It acts as an intermediary between the browser and the application.

### **🔹 What is the role of the application server**

The application server:

*   contains the business logic
    
*   processes data
    
*   communicates with the database
    
*   generates dynamic responses
    

Examples: Django, Node.js, Flask, Rails, etc.

### **🔹 What is the role of the database**

The database:

*   stores persistent data (users, content, etc.)
    
*   ensures their consistency and durability
    
*   is queried by the application only

### **🔹 What is the server using to communicate with the computer of the user**

The server communicates with the browser via:

*   the HTTP or HTTPS protocol
    
*   on port 80 (HTTP) or 443 (HTTPS)
    
*   using the client–server model

---

**Problems with this infrastructure**
-------------------------------------

### **⚠️ SPOF (Single Point Of Failure)**

Everything is hosted on a single server.

If it falls:

*   the site becomes inaccessible
    
*   application, database and web server are affected
    

Zero tolerance for breakdowns.

### **⚠️ Downtime during maintenance**

To deploy a new version:

*   the web or application server often needs to be restarted
    
*   the site is temporarily unavailable
    

No rolling updates, no high availability.

---

This architecture is simple and functional for a small project or test environment, but it is neither scalable nor resilient, and presents significant risks to availability in production.

---
## **1-distributed-web-infrastructure**

This infrastructure introduces high availability and improved scalability by adding a load balancer, multiple application servers, and a Primary-Replica replication database.

The objective is to distribute the load, reduce service interruptions, and improve resilience compared to a monolithic architecture.

---

**Added elements and justification**
------------------------------------

### **🔹 Load Balancer**

**Why add it:**

*   distribute traffic across multiple web/application servers
    
*   preventing a single server from becoming overloaded
    
*   improve service availability
    

It becomes the single point of entry for customers.

### **🔹 Multiple Web / Application Servers**

**Why add them:**

*   absorb more traffic
    
*   enable horizontal scaling
    
*   prevent a server failure from making the site unavailable
    

Each server is assumed to be stateless.

### **🔹 Database Primary-Replica (Master-Slave)**

**Why add it:**

*   improve reading performance
    
*   provide data redundancy
    
*   reduce the load on the main node
    
---

**Load Balancer – operation**
-----------------------------

### **🔹 Distribution algorithm**

The load balancer is configured in Round Robin mode (the most common case).

**Operation:**

*   each request is sent to the next server in the list
    
*   equitable distribution of the load
    
*   simple, effective, but without taking into account the actual load
    

### **🔹 Active-Active vs Active-Passive**

**Active-Active (case here)**

*   all backend servers process requests
    
*   better use of resources
    
*   actual horizontal scaling
    

**Active-Passive**

*   one active server, one or more standby servers
    
*   switches over only in the event of a failure
    
*   less efficient but simpler
    
---

**Database Primary-Replica**
----------------------------

### **🔹 How it works**

*   The Primary receives all writes (INSERT, UPDATE, DELETE).
    
*   Replicas replicate data from the Primary.
    
*   the application can read from replicas
    

### **🔹 Difference between Primary and Replica for the application**

| Role | Primary | Replica |
|:--------:|:--------:|:--------:|
| Ecriture| Yes | No |
| Lectures | Yes | Yes |
| Source de vérité | Yes | No |

Entries must go through the Primary.

---

**Problems with this infrastructure**
-------------------------------------

### **⚠️ SPOF (Single Point Of Failure)**

*   **Single Load Balancer → if it goes down, no more access**
    
*   **Single primary database → critical failure if not automatically promoted**
    

### **⚠️ Security issues**

*   **No firewall → direct exposure of servers**
    
*   **No HTTPS → unencrypted data (credentials, sessions)**
    
*   unnecessarily large attack surface
    

### **⚠️ No monitoring**

*   no visibility on:
    
    *   server status
        
    *   the load
        
    *   errors
        
*   fault detection only via users
    
---

This architecture significantly improves availability and performance, but remains fragile without security, monitoring, and load balancer redundancy. It is a good foundation, but not yet a production-ready infrastructure.

---

## **2-secured_and_monitored_web_infrastructure**

This infrastructure introduces:

*   firewalls to secure access
    
*   HTTPS to secure communications
    
*   a monitoring tool to track server health and performance
    

The objective is to make the system more resilient, secure and observable, while maintaining a distributed architecture with load balancer and Primary-Replica distribution for the database.

---

**Added elements and justification**
------------------------------------

### **🔹 Firewalls**

**Why add them:**

*   filter incoming and outgoing traffic
    
*   restrict access to necessary ports (ex: 80, 443)
    
*   protect against network attacks (DDoS, scans, etc.)
    

### **🔹 HTTPS**

**Why traffic is served over HTTPS:**

*   encrypts exchanges between the client and the server
    
*   protects credentials, sessions, and sensitive data
    
*   improves user confidence and SEO
    

### **🔹 Monitoring**

**Rôle :**

*   collect metrics and logs (CPU, memory, QPS, errors, response time)
    
*   detect anomalies and faults
    
*   enable proactive alerts
    

**How data is collected:**

*   agents installed on each server
    
*   collection of system and application metrics
    
*   sending to a central monitoring server (ex: Prometheus, Grafana)
    

**Example for monitoring QPS (queries per second) of a web server:**

*   configure the web server to export HTTP metrics
    
*   configure monitoring to collect the requests per second counter
    
*   create a dashboard or alerts if the QPS exceeds a critical threshold

---

**Problems with this infrastructure**
-------------------------------------

### **⚠️ Terminating SSL to load balancer**

*   if the LB decrypts traffic, communications between the LB and the backend are unencrypted
    
*   exposes internal servers to network attacks
    
*   solution: end-to-end SSL (encryption up to the final server)
    

### **⚠️ Only one MySQL database for writing**

*   Single Point of Failure for writes
    
*   if it falls, no more writing possible
    
*   solution: Primary-Replica with automatic failover or multi-primary cluster
    

### **⚠️ Identical servers (DB + Web + App) on all nodes**

*   mixing responsibilities = complexity, unnecessary redundancy
    
*   difficulty in scaling components independently
    
    *   ex: DB load increase = you unnecessarily duplicate Web/App
        
*   solution: separate roles by server type (Web/App vs DB)
    
---

This architecture is more secure and observable, but it remains fragile in terms of:

*   SSL and DB redundancy
    
*   component separation for scalability
    
*   fault tolerance if certain nodes fail

---

## **3-scale_up**

This architecture is designed for high availability and scalability. Each key component (web server, application server, database) has its own dedicated server, and the load balancer is now clustered to avoid any SPOF.

The objective is to enable:

*   heavier traffic
    
*   total resilience at critical points
    
*   an independent ramp-up for each component
    
---

**Added elements and justification**
------------------------------------

### **🔹 Additional server**

**Why add it:**

*   increase processing capacity
    
*   reduce the load on existing servers
    
*   ensure redundancy in the event of failure
    

### **🔹 Load Balancer HA (HAproxy)**

**Why add it:**

*   prevent the single load balancer from being a SPOF
    
*   distribute traffic evenly between web/app servers
    
*   Clustered, it enables automatic failover: if one LB goes down, the other takes over.
    

### **🔹 Separation of components**

**Why each component has its own server:**

| Component | Role | Benefits |
|:--------:|:--------:|:--------:|
| Web Server| Serves static files, receives requests | Independent scalability, less backend overhead |
| Application server |Business logic, query processing| Separates CPU and memory load from the DB|
| Database | Stores data, manages consistency | Insulation, safety and optimised performance |

*   This separation allows each component to be scaled independently as required.
    
*   Reduces the risk of conflict or overload if a component becomes heavily used.
    
---

**Conclusion**
--------------

This architecture is production-ready for high traffic:

*   full redundancy on the load balancer
    
*   complete insulation of components
    
*   horizontal and vertical scalability possible
    
*   fault tolerance and maintenance without major downtime