🐳 Voting App – Docker Compose Deployment
📌 Overview

This project is a complete voting application composed of multiple microservices working together through Docker Compose:

vote → Voting frontend

result → Results UI

worker → Background processor between Redis and PostgreSQL

redis → Cache + queue

db (PostgreSQL) → Main database

seed → Optional service to initialize seed data

The setup includes healthchecks, networks, and seed profiles for controlled startup.

📁 Project Structure
.
├── docker-compose.yml
├── healthchecks/
│   ├── redis.sh
│   └── postgres.sh
└── README.md

🚀 How to Run the System
1️⃣ Start all services
docker compose up -d

2️⃣ Verify running containers
docker ps

3️⃣ Access the application
Service	URL
Vote App	http://localhost:8080

Results	http://localhost:8081
🌱 Running the Seed Service (Optional)

The seed service is disabled by default because it belongs to the seed profile.

To run it manually:

docker compose --profile seed up seed


or:

docker compose --profile seed up -d


This initializes the database with seed data.

🧩 Detailed Service Explanation
1️⃣ vote (Frontend)
image: 01061875164/vote-app-1:v2
ports:
  - "8080:80"
depends_on:
  redis:
    condition: service_healthy
environment:
  - OPTION_A=Cats
  - OPTION_B=Dogs


✔ Provides the voting frontend
✔ Waits for Redis healthcheck before starting
✔ Uses environment variables for vote options

2️⃣ result (Results UI)
image: 01061875164/result:v3
ports:
  - "8081:4000"
depends_on:
  db:
    condition: service_healthy


✔ Displays voting results
✔ Depends on PostgreSQL
✔ Runs internally on port 4000

3️⃣ worker
image: 01061875164/worker:v2
depends_on:
  redis:
    condition: service_healthy
  db:
    condition: service_healthy


✔ Processes background jobs
✔ Moves data from Redis → PostgreSQL
✔ No exposed ports (backend-only service)

4️⃣ redis
image: 01061875164/redis:v1
healthcheck:
  test: ["CMD", "sh", "/healthcheck.sh"]


✔ Includes custom healthcheck
✔ Doesn’t start dependent services until healthy

5️⃣ db (PostgreSQL)
environment:
  - POSTGRES_USER=postgres
  - POSTGRES_PASSWORD=postgres
  - POSTGRES_DB=postgres
volumes:
  - postgres-data:/var/lib/postgresql/data


✔ Initializes PostgreSQL
✔ Custom healthcheck via postgres.sh
✔ Data is persisted using a named volume

6️⃣ seed (Optional)
profiles:
  - seed


✔ Runs once when manually triggered
✔ Seeds PostgreSQL with initial records

🧪 Healthchecks

Custom healthcheck scripts ensure services start in correct order.

Service	Healthcheck Command
redis	sh /healthcheck.sh
db	sh /healthcheck.sh

Docker Compose will not start vote/result/worker until Redis and PostgreSQL are healthy.

🔗 Networks

Two networks are defined:

frontend-tier → vote, result, seed

backend-tier → redis, db, worker, seed

This separation improves security and isolation.

💾 Persistent Storage

PostgreSQL uses a persistent volume:

volumes:
  postgres-data:


This prevents data loss when restarting or recreating containers.

🛑 Stop the Application

Stop containers:

docker compose down


Stop containers and delete volumes:

docker compose down -v


