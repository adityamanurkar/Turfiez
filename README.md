# Turfiez - Turf Booking System

Turfiez is a full-stack Turf Booking System built with a React frontend and a Java Spring Boot backend. It provides turf discovery/booking workflows, secure authentication, booking management, notifications/integrations, and a production-oriented Docker setup.

## Architecture

```text
                    Internet / Browser
                           |
                           v
                 React + Vite Frontend
                           |
                        REST API
                           |
                           v
                Spring Boot Backend
                 |       |       |
                 |       |       +--> Email / Twilio
                 |       |
                 |       +----------> Cloudinary
                 |
                 v
             PostgreSQL
```

## Tech Stack

### Frontend
- React
- Vite
- Tailwind CSS
- Framer Motion
- Axios
- Zustand
- React Query

### Backend
- Java 17
- Spring Boot 3.2.x
- Spring Web
- Spring Data JPA / Hibernate
- Spring Security
- JWT
- Bean Validation
- PostgreSQL
- Spring Mail
- Twilio
- Cloudinary
- Swagger / OpenAPI

### Deployment
- Docker
- Docker Compose
- AWS EC2 / RDS
- AWS ECR
- AWS EKS / Kubernetes

## Repository Structure

```text
Turfiez/
├── turf-booking-backend/
│   ├── src/
│   ├── pom.xml
│   └── Readme.md
├── turf-booking-frontend/
│   ├── src/
│   ├── package.json
│   └── Readme.md
├── docker-compose.yml
├── eks.md
└── README.md
```

## Prerequisites

For local development:

- JDK 17+
- Maven
- Node.js 20+
- npm
- PostgreSQL 14+
- Git

For Docker deployment:

- Docker
- Docker Compose

For AWS deployment:

- AWS account
- AWS CLI
- ECR
- EC2 or EKS
- PostgreSQL RDS if using managed database

## 1. Clone the Project

```bash
git clone https://github.com/adityamanurkar/Turfiez.git
cd Turfiez
```

## 2. PostgreSQL Database Setup

Create the database:

```sql
CREATE DATABASE turf_db;
```

Create a dedicated application user:

```sql
CREATE USER turf_user WITH PASSWORD 'CHANGE_ME';
GRANT ALL PRIVILEGES ON DATABASE turf_db TO turf_user;
```

Connect to the database and grant schema permissions if required:

```sql
\c turf_db

GRANT USAGE, CREATE ON SCHEMA public TO turf_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO turf_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO turf_user;
```

The application uses Hibernate `ddl-auto: update`, so the required JPA tables are created/updated when the backend starts.

## 3. Backend Configuration

The backend reads database and service credentials from environment variables.

Example:

```bash
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=turf_db
export DB_USERNAME=turf_user
export DB_PASSWORD=CHANGE_ME

export JWT_SECRET=CHANGE_ME

export MAIL_USERNAME=your-email@gmail.com
export MAIL_PASSWORD=your-gmail-app-password

export TWILIO_ACCOUNT_SID=your-twilio-sid
export TWILIO_AUTH_TOKEN=your-twilio-auth-token
export TWILIO_PHONE_NUMBER=your-twilio-phone-number

export CLOUDINARY_CLOUD_NAME=your-cloud-name
export CLOUDINARY_API_KEY=your-api-key
export CLOUDINARY_API_SECRET=your-api-secret
```

Do not commit real passwords, JWT secrets, API keys, Twilio credentials, mail passwords, or Cloudinary secrets to Git.

## 4. Run Backend Locally

```bash
cd turf-booking-backend
mvn clean package
mvn spring-boot:run
```

Backend:

```text
http://localhost:8080
```

Swagger UI:

```text
http://localhost:8080/swagger-ui.html
```

OpenAPI JSON:

```text
http://localhost:8080/v3/api-docs
```

## 5. Run Frontend Locally

Open another terminal:

```bash
cd turf-booking-frontend
npm install
```

Create the frontend environment file from the project's `.env.example` when available and configure the backend API URL.

Then:

```bash
npm run dev
```

Frontend:

```text
http://localhost:5173
```

## 6. Run Full Application with Docker Compose

From the repository root:

```bash
docker compose up --build
```

Check running containers:

```bash
docker ps
```

Stop the application:

```bash
docker compose down
```

To remove volumes as well:

```bash
docker compose down -v
```

## 7. AWS RDS PostgreSQL Deployment

Create an Amazon RDS PostgreSQL database and note:

- RDS endpoint
- Port: `5432`
- Database name
- Username
- Password

Make sure the RDS security group permits PostgreSQL traffic only from the backend security group or private network.

Set backend variables:

```bash
export DB_HOST=<RDS_ENDPOINT>
export DB_PORT=5432
export DB_NAME=turf_db
export DB_USERNAME=<RDS_USERNAME>
export DB_PASSWORD=<RDS_PASSWORD>
```

Test connectivity from the backend server:

```bash
psql -h <RDS_ENDPOINT> -p 5432 -U <RDS_USERNAME> -d turf_db
```

## 8. Docker Image Build

Backend:

```bash
cd turf-booking-backend
docker build -t turfiez-backend:v1 .
```

Frontend:

```bash
cd ../turf-booking-frontend
docker build -t turfiez-frontend:v1 .
```

Run examples:

```bash
docker run -d --name turfiez-backend -p 8080:8080   -e DB_HOST=<DB_HOST>   -e DB_PORT=5432   -e DB_NAME=turf_db   -e DB_USERNAME=<DB_USERNAME>   -e DB_PASSWORD=<DB_PASSWORD>   turfiez-backend:v1
```

```bash
docker run -d --name turfiez-frontend -p 80:80 turfiez-frontend:v1
```

## 9. AWS ECR

Authenticate Docker to ECR:

```bash
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com
```

Create repositories:

```bash
aws ecr create-repository --repository-name turfiez-backend --region ap-south-1
aws ecr create-repository --repository-name turfiez-frontend --region ap-south-1
```

Tag images:

```bash
docker tag turfiez-backend:v1 <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/turfiez-backend:v1
docker tag turfiez-frontend:v1 <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/turfiez-frontend:v1
```

Push:

```bash
docker push <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/turfiez-backend:v1
docker push <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/turfiez-frontend:v1
```

## 10. Kubernetes / EKS

The `eks.md` file contains a Kubernetes deployment template covering:

- Namespace
- PostgreSQL configuration
- Backend Deployment
- Backend Service
- Frontend Deployment
- Frontend LoadBalancer Service
- Secrets
- ConfigMap
- Deployment and verification commands

Apply manifests:

```bash
kubectl create namespace turfiez

kubectl apply -f postgres-secret.yaml
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```

Verify:

```bash
kubectl get pods -n turfiez
kubectl get svc -n turfiez
kubectl get deployments -n turfiez
```

Get the frontend LoadBalancer:

```bash
kubectl get svc frontend-service -n turfiez
```

## Environment Variables

| Variable | Purpose |
|---|---|
| `DB_HOST` | PostgreSQL hostname |
| `DB_PORT` | PostgreSQL port |
| `DB_NAME` | Database name |
| `DB_USERNAME` | Database user |
| `DB_PASSWORD` | Database password |
| `JWT_SECRET` | JWT signing secret |
| `MAIL_USERNAME` | SMTP email account |
| `MAIL_PASSWORD` | SMTP app password |
| `TWILIO_ACCOUNT_SID` | Twilio account |
| `TWILIO_AUTH_TOKEN` | Twilio authentication |
| `TWILIO_PHONE_NUMBER` | Twilio sender number |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary cloud |
| `CLOUDINARY_API_KEY` | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret |

## Useful Commands

```bash
git status
git pull
git add .
git commit -m "Update Turfiez documentation"
git push
```

Docker:

```bash
docker ps
docker logs <container>
docker exec -it <container> sh
docker stop <container>
docker rm <container>
```

Kubernetes:

```bash
kubectl get pods -n turfiez
kubectl logs <pod-name> -n turfiez
kubectl describe pod <pod-name> -n turfiez
kubectl get svc -n turfiez
```

## Security Notes

- Never commit `.env` files containing real credentials.
- Use AWS Secrets Manager, Kubernetes Secrets, or another secret-management solution for production credentials.
- Restrict PostgreSQL access to the backend network/security group.
- Use a strong random JWT secret in production.
- Use HTTPS/TLS for production traffic.
- Do not expose PostgreSQL directly to the public internet.

## License

This project is intended for learning and development purposes.
