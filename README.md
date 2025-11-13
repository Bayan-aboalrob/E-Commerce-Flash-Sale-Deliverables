# Flash Sale E-Commerce System — README

## 📦 Microservices Repositories (Highly Visible)
Each microservice is implemented in its own standalone repository:

- **User Management:** https://github.com/Bayan-aboalrob/UserManagement
- **Reservation Service:** https://github.com/Bayan-aboalrob/ReservationServices
- **Order Service:** https://github.com/Bayan-aboalrob/OrderService
- **Payment Service:** https://github.com/Bayan-aboalrob/PaymentService
- **Inventory Service:** https://github.com/duaa-braik/InventoryManagement
- **Load Tests:** https://github.com/duaa-braik/FlashSale.LoadTest

🟦 **Nginx Configuration:** Included inside the **InventoryManagement** repository.

---

# 1. ⭐ Scenario Chosen and Why
This project simulates a **flash-sale system** where thousands of users compete for a limited set of products simultaneously. The system must handle:

- Very high concurrency  
- Inventory race conditions  
- Payment reliability  
- Overselling prevention  
- Real-time inventory updates  

The architecture was designed to compare two real-world approaches: **Synchronous HTTP chaining** vs **Asynchronous event-driven messaging**, analyzing their behavior under heavy load.

---

# 2. 🧰 Tech Stack Used

### **Backend**
- .NET 8 — REST APIs  
- Clean Architecture  
- SQL Server — Separate database for each microservice (DB-per-service pattern)  

### **Messaging**
- RabbitMQ — Event-driven communication  

### **Caching**
- Redis — Inventory & reservation caching  

### **Monitoring**
- InfluxDB 2.7 — Time-series database  
- Grafana — Dashboards for CPU, memory, latency, throughput  

### **Load Testing**
- k6 — Full workflow load tests  

### **Gateway**
- Nginx — Reverse proxy + Load balancer  

---

# 3. 🛠 Setup Instructions (No Docker for Microservices)
You **do not** run the services via Docker.  
Only monitoring & message broker tools use Docker.

### 3.1 Run supporting services via Docker  
```bash
docker run -d --name redis -p 6379:6379 redis
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
docker run -d --name influxdb -p 8086:8086 influxdb:2.7
docker run -d --name grafana -p 3000:3000 grafana/grafana-oss
```

---

# 4. 🗄 SQL Server Database Setup

Each microservice has its own **SQL Server database** following DB-per-service architecture.

### Example connection string:
```txt
Server=localhost,1433;
Database=InventoryDb;
User Id=sa;
Password=YourStrong!Passw0rd;
TrustServerCertificate=True;
```

### Databases:
| Service | Database |
|---------|----------|
| User Management | UserDb |
| Reservation Service | ReservationDb |
| Order Service | OrderDb |
| Payment Service | PaymentDb |
| Inventory Service | InventoryDb |

You must update each service's `appsettings.json`:

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=localhost,1433;Database=OrderDb;User Id=sa;Password=YourStrong!Passw0rd;TrustServerCertificate=True;"
}
```

Run migrations in each service:
```bash
dotnet ef database update
```

---

# 5. 🧪 How to Run Load Tests

### Run normally:
```bash
k6 run tests/flashsale-endtoend.js
```

### Run with InfluxDB metrics:
```bash
k6 run --out influxdb=http://localhost:8086/k6 tests/flashsale-endtoend.js
```

---

# 6. 🏗️ Brief Architecture Overview

## 6.1 User Management (First in Flow)
- Handles login, registration, identity validation  
- Issues JWT tokens  
- All requests pass through **User Management → Reservation Service**  

---

## 🔵 6.2 Synchronous Architecture (HTTP)
Flow:

**Nginx → User Management → Reservation → Order → Payment → Inventory**

Characteristics:
- Direct HTTP calls  
- Strong consistency  
- Higher latency on peak load  
- Thread blocking  
- Cascading failures possible  

---

## 🟢 6.3 Asynchronous Architecture (Events)
Flow:

**Reservation → RabbitMQ → Order → Payment → Inventory**

Characteristics:
- Non-blocking  
- High scalability  
- Better concurrency handling  
- Requires retries, DLQ, idempotency  

---

# 7. 📊 Grafana + InfluxDB Setup

### Add InfluxDB as a data source:
- URL: `http://localhost:8086`
- Token: (generated from Influx)
- Bucket: flashsale-bucket  

### Recommended dashboards:
- CPU per service  
- Memory per service  
- Aggregated CPU (sum of all services)  
- Aggregated memory  
- Throughput (requests/sec) from k6  
- Latency charts  

---

# 8. 🔌 Ports Overview

| Component | Port |
|----------|------|
| Nginx | 9090 |
| Grafana | 3000 |
| InfluxDB | 8086 |
| RabbitMQ UI | 15672 |
| Redis | 6379 |
| SQL Server | 1433 |

---

This README is complete and aligned with your architecture.  
