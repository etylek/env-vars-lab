# Laboratory Work #4: Environment Variables & Configuration Management

## Overview
This laboratory work demonstrates best practices for managing environment-specific configurations in containerized microservices. Using a Flask API connected to PostgreSQL, this project highlights secure credential handling, runtime variable validation, strict secrets isolation via `.gitignore`, and environment segregation using Docker networks.

---

## Project Structure

```text
lab4-env-vars/
├── app.py              # Flask API with endpoints and error handling
├── config.py           # Configuration parser and startup validator
├── requirements.txt    # Application dependencies
├── Dockerfile          # Multi-stage/production-ready container image spec
├── .env.development    # Development environment settings
├── .env.production     # Production environment settings
├── .env.example        # Version-controlled configuration template
├── .gitignore          # Rules ensuring secrets are never committed
└── README.md           # Project documentation and guide
```
---

## Instructions & Execution Guide
Part 1: Build the Application Image
```
docker build -t config-app .
```
Part 2: Running in Development Mode

Start Development Database:

```
docker run -d \
  --name dev-postgres \
  -e POSTGRES_USER=devuser \
  -e POSTGRES_PASSWORD=devpassword123 \
  -e POSTGRES_DB=devdb \
  -p 5432:5432 \
  postgres:15-alpine
```

Start Dev Application Container (with Linux host mapping):

```
docker run -d \
  --name config-app-dev \
  --add-host=host.docker.internal:host-gateway \
  --env-file .env.development \
  -p 5000:5000 \
  config-app
```

Verify Development Endpoint:

```
curl http://localhost:5000/
curl http://localhost:5000/db-test
```

Part 3: Running in Production Mode (Isolated Network)
Create Isolated Bridge Network:

```
docker network create app-network
```

Launch Production PostgreSQL Database:

```
docker run -d \
  --name prod-postgres \
  --network app-network \
  -e POSTGRES_USER=produser \
  -e POSTGRES_PASSWORD=prodpass123 \
  -e POSTGRES_DB=proddb \
  postgres:15-alpine
```

Launch Production Application Container:

```
docker run -d \
  --name config-app-prod \
  --network app-network \
  --env-file .env.production \
  -p 5001:5000 \
  config-app
```

Verify Production Endpoint:

```
curl http://localhost:5001/
curl http://localhost:5001/db-test
```
