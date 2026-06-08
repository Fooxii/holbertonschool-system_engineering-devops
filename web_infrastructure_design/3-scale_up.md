```mermaid
flowchart TD
    User["User browser\nwww.foobar.com"]

    subgraph LB_CLUSTER ["HAProxy cluster"]
        LB1["HAProxy - primary"]
        LB2["HAProxy - secondary"]
        LB1 <-->|"Keepalived sync"| LB2
    end

    User -->|"HTTPS request"| LB_CLUSTER

    subgraph WEB_SERVER ["Web server"]
        Nginx["Nginx\nServes static files and proxies"]
    end

    subgraph APP_SERVER ["Application server"]
        App["Application server\nRuns business logic"]
    end

    subgraph DB_SERVER ["Database server"]
        DB["MySQL\nStores persistent data"]
    end

    LB_CLUSTER --> WEB_SERVER
    WEB_SERVER --> APP_SERVER
    APP_SERVER --> DB_SERVER
```

## Why each element was added

**1 additional server:**
Each component (web server, application server, database) now lives on its own dedicated server. Separating them eliminates resource contention — a traffic spike on the web server no longer competes with database queries for CPU and RAM. It also allows each layer to be scaled, maintained, and secured independently.

**HAProxy configured as a cluster with the other load balancer:**
Adding a second HAProxy instance configured as a cluster with the first eliminates the load balancer as a single point of failure. The two instances use Keepalived to share a virtual IP address. If the primary HAProxy goes down, the secondary takes over the virtual IP automatically with no manual intervention and no downtime for users.

**Split components (web server, application server, database) on their own servers:**
Separating components allows each one to be sized and scaled to its own workload. If the application logic becomes the bottleneck, only the application server needs to be scaled up or out. If database read load increases, a replica can be added to the database tier without touching the other servers. This architecture is the foundation for horizontal scaling.
