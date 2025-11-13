# Flash Sale E-Commerce System — README

## 📦 Microservices Repositories (Highly Visible)
- **User Management:** https://github.com/Bayan-aboalrob/UserManagement  
- **Inventory Service:** https://github.com/duaa-braik/InventoryManagement  
- **Reservation Service:** https://github.com/Bayan-aboalrob/ReservationServices  
- **Order Service:** https://github.com/Bayan-aboalrob/OrderService  
- **Payment Service:** https://github.com/Bayan-aboalrob/PaymentService  
- **Load Tests:** https://github.com/duaa-braik/FlashSale.LoadTest  

🟦 **Nginx Configuration:**  
Included inside the **InventoryManagement** repository.

---

## 🧩 Architecture Overview  
This system follows a **microservices-based architecture**, where each service is isolated in its own repository and communicates through either synchronous HTTP calls or asynchronous message-driven flows.

### 🔵 Synchronous Architecture  
Flow:  
**Nginx → Reservation API → Order API → Payment API → Inventory API**

Characteristics:  
- Each service waits for the next.  
- Strong consistency.  
- Higher latency under load.  
- Risk of cascading failures.  
- Simple debugging and flow tracing.

### 🟢 Asynchronous Architecture  
Flow:  
**Reservation publishes events → RabbitMQ → Order → Payment → Inventory**

Characteristics:  
- Fully non-blocking communication.  
- High scalability and fault isolation.  
- Supports very high concurrency.  
- Requires idempotency, retry rules, DLQs.

---

## 🚀 Running the System  
From the root (where docker-compose exists):

```bash
docker-compose up --build
```

---

## 🔌 Ports (Important)
| Component | Port |
|----------|------|
| **API Gateway (Nginx)** | `9090` |
| **Grafana** | `3000` |
| **InfluxDB** | `8086` |
| **RabbitMQ Management UI** | `15672` |
| **Redis** | `6379` |

---

## 📊 InfluxDB Setup  
Install InfluxDB using Docker:

```bash
docker run -d --name influxdb -p 8086:8086 influxdb:2.7
```

Then inside the UI:
- Create **organization**
- Create **bucket**
- Generate **token**

Use these in each service’s background metrics collector.

---

## 📈 Grafana Setup  
Download Grafana (Windows):  
https://grafana.com/grafana/download?platform=windows&edition=oss

Add data source:
- **URL:** `http://localhost:8086`
- **Bucket:** your InfluxDB bucket  
- **Organization:** your org  
- **Token:** your generated token  

Build dashboards for:
- CPU per microservice  
- Memory per microservice  
- Summed CPU usage (system-wide)  
- Summed memory usage  
- Throughput  
- Latency metrics  

---

## 🧪 Load Testing  
### Run test normally:
```bash
k6 run tests/flashsale-endtoend.js
```

### Run with metrics pushed to Influx:
```bash
k6 run --out influxdb=http://localhost:8086/k6 tests/flashsale-endtoend.js
```

Load tests repo:  
https://github.com/duaa-braik/FlashSale.LoadTest

---

## 🗂 Repository Structure  
```
/src
  /ReservationService
  /OrderService
  /PaymentService
  /InventoryService
  /UserService
```

Each service follows:
- Clean Architecture  
- Domain layer + Application layer + Infrastructure  
- Database per microservice  
- Messaging integrated using RabbitMQ  
- Redis for reservation + inventory caching  
