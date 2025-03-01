# Fault-Tolerant-DB-As-a-Service

This project demonstrates a fault-tolerant database service built using Docker, Flask, Zookeeper, RabbitMQ, and Docker Compose. The system is designed to handle read and write operations through an orchestrator that coordinates multiple worker nodes (slaves and a master) to ensure high availability and consistency.

## Project Structure

- **Orchestrator & Workers**  
    - [orch/orchestrator.py](orch/orchestrator.py) – Main orchestrator service responsible for managing workers, handling master/slave election via Zookeeper, and forwarding DB read/write requests.
    - [orch/worker.py](orch/worker.py) – Worker service that executes database operations and synchronizes with other workers.
    - [orch/Dockerfile](orch/Dockerfile) – Dockerfile for building the orchestrator container.
    - [orch/docker-compose.yml](orch/docker-compose.yml) – Docker Compose file for running Zookeeper, RabbitMQ, and the orchestrator.

- **User Microservice**  
    - [user/user_man/app/main.py](user/user_man/app/main.py) – Main Flask application providing user-related REST APIs.
    - [user/user_man/app/CC_0094_0155_0260_1509_users.py](user/user_man/app/CC_0094_0155_0260_1509_users.py) – API endpoints for creating and deleting users.
    - [user/user_man/app/test.py](user/user_man/app/test.py) – Test script for user service.
    - [user/user_man/dockerfile](user/user_man/dockerfile) – Dockerfile for building the user service.
    - [user/docker-compose.yml](user/docker-compose.yml) – Docker Compose file for the user microservice.

- **Ride Microservice**  
    - [ride/ride_man/app/main.py](ride/ride_man/app/main.py) – Main Flask app for ride-related operations such as creating rides and joining rides.
    - [ride/ride_man/app/CC_0094_0155_0260_1509_rides.py](ride/ride_man/app/CC_0094_0155_0260_1509_rides.py) – Ride service endpoints.
    - [ride/ride_man/dockerfile](ride/ride_man/dockerfile) – Dockerfile for the ride service.
    - [ride/docker-compose.yml](ride/docker-compose.yml) – Docker Compose file for the ride microservice.

- **Configuration & Keys**  
    - [demo.pem](demo.pem) – RSA private key.
    - [AreaNameEnum.csv](user/user_man/app/AreaNameEnum.csv) and [ride/ride_man/app/AreaNameEnum.csv](ride/ride_man/app/AreaNameEnum.csv) – CSV mappings used by services.

- **Documentation**  
    - [README.md](README.md) – This readme file.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed on your system.
- Docker Compose installed.

## How to Build and Run

### Orchestrator and Workers

1. **Build and Run the Orchestrator**  
     Navigate to the project root and run:
     ```sh
     docker-compose -f orch/docker-compose.yml build
     docker-compose -f orch/docker-compose.yml up
     ```
     This command builds and starts Zookeeper, RabbitMQ, and the orchestrator (which in turn manages worker nodes).

### Ride and User Services

1. **Run User Service**  
     In a separate terminal, run:
     ```sh
     docker-compose -f user/docker-compose.yml build
     docker-compose -f user/docker-compose.yml up
     ```

2. **Run Ride Service**  
     Similarly, run:
     ```sh
     docker-compose -f ride/docker-compose.yml build
     docker-compose -f ride/docker-compose.yml up
     ```

## API Endpoints

### User Service

- **Create User**  
    `PUT /api/v1/users`  
    Uses CC_0094_0155_0260_1509_users.py to add a new user.

- **Delete User**  
    `DELETE /api/v1/users/<username>`

- **List Users**  
    `GET /api/v1/users`

### Ride Service

- **Create Ride**  
    `POST /api/v1/rides`  
    See main.py for implementation details.

- **Join Ride**  
    `POST /api/v1/rides/<rideId>`

- **List Ride Details**  
    `GET /api/v1/rides/<rideId>`  
    `GET /api/v1/rides?source=<source>&destination=<destination>`

### Database and Orchestration

- **DB Write**  
    `POST /api/v1/db/write` – For write operations. Handled by the orchestrator.

- **DB Read**  
    `POST /api/v1/db/read` – For read operations.

- **Clear DB**  
    `POST /api/v1/db/clear`

### Worker Management APIs

- Create a new slave: `POST /api/v1/create/slave`
- Crash master/slave: `POST /api/v1/crash/master` and `POST /api/v1/crash/slave`

## Architecture Overview

- **Orchestrator:**  
    Uses Zookeeper to monitor and elect a master amongst worker nodes (orchestrator.py). It forwards DB queries to the master/slave workers via RabbitMQ queues.

- **Workers:**  
    Each worker starts as a slave (orch/worker.py) and may be promoted to master if needed. Workers synchronize their local SQLite databases to ensure consistency.

- **Message Queuing:**  
    RabbitMQ is used to distribute read/write requests among workers.

- **Microservices:**  
    The ride and user microservices expose REST APIs to interact with the underlying fault-tolerant DB service.

## Testing

The user service test script is available at [test.py](user/user_man/app/test.py).

## License

Distributed under the MIT License.