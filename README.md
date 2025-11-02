# 🚀 Blue/Green Deployment with Nginx Upstreams  
DevOps Intern – Stage 2 Task  

This project sets up a **Blue/Green deployment** using **Nginx** and **Docker Compose**, where:
- **Blue** is the active (primary) service.
- **Green** is the standby (backup) service.
- **Nginx** automatically switches to Green if Blue fails.
- You can manually trigger Blue’s failure using the provided endpoints.

---

## 🧩 What’s Included
- Two Node.js services: **app_blue** and **app_green**
- **Nginx** reverse proxy handling routing and failover
- **Docker Compose** to orchestrate everything
- **.env** file to configure environment variables

---

## ⚙️ Requirements
Make sure you have the following installed:
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/)
- `curl` (for testing)

---

## 📁 Project Structure
```
.
├── .env.example             # Environment configuration
├── docker-compose.yml       # Service definitions
├── nginx/
│   ├── docker-entrypoint.sh # Nginx configuration 
├── alert_watcher/
│   ├── Dockerfile          # Watcher container setup
│   ├── watcher.py          # Alert logic
│   └── requirements.txt    # Python dependencies
├── tests/
│   ├── smoke.sh            # Basic health checks
│   ├── failover_test.sh    # Failover testing
│   ├── induce_chaos.sh
│   └── test_all_alerts.sh  # Complete test suite
└── runbook.md              # Operational guide
```

---

## 🧾 .env Example
Create a `.env` file in the root directory:

```env
# .env - set by CI or locally for testing
BLUE_IMAGE=yourrepo/app-blue:latest
GREEN_IMAGE=yourrepo/app-green:latest

# active pool = "blue" or "green"
ACTIVE_POOL=blue

# release ids to be injected into the app containers
RELEASE_ID_BLUE=blue-1.2.3
RELEASE_ID_GREEN=green-1.2.3

# container internal port the app listens on (apps should honor this)
PORT=3000

# NGINX external port
NGINX_PORT=8080

BLUE_HOST_PORT=8081
GREEN_HOST_PORT=8082

# Alerting Configuration
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/your/webhook/url
ERROR_RATE_THRESHOLD=2
WINDOW_SIZE=200
ALERT_COOLDOWN_SEC=300
MAINTENANCE_MODE=false

# Logging
NGINX_LOG_LEVEL=info
LOG_RETENTION_DAYS=7
```

## ▶️ How to Run
1. Clone the repository
```
git clone <repo-link>
cd <repo-folder>
```

2. Create the .env file
Copy the example above and update the values if needed.

3. Start the containers
```
docker-compose up -d
```

5. Check running containers
```
docker ps
```

You should see:
- app_blue
- app_green
- nginx


## ✅ Test the Setup
Check the active version
```
curl -i http://localhost:8080/version
```

You should see:
```
X-App-Pool: blue
```

Simulate a Blue failure
```
curl -X POST http://localhost:8081/chaos/start?mode=error
```

Now, test again:
```
curl -i http://localhost:8080/version
```

You should see:
```
X-App-Pool: green
```

Stop the failure
```
curl -X POST http://localhost:8081/chaos/stop
```
Blue will recover and become active again.

## 🧪 Optional Quick Test Script
Run:
```
bash tests/failover_test.sh
```
This automatically simulates a failure and verifies the system switches from Blue → Green with no downtime.

🧹 Stop Everything
```
docker-compose down
```

💡 Notes
- All traffic goes through Nginx (http://localhost:8080)
- Blue and Green are available directly on:
  - Blue → http://localhost:8081
  - Green → http://localhost:8082
- Failover happens automatically within the same request if Blue fails.
