# MEAN Stack Application - Dockerized

This repository contains a **MEAN (MongoDB, Express, Angular, Node.js) stack application** containerized with **Docker** and managed using **Docker Compose**.

## Prerequisites
Ensure you have the following installed before proceeding:
- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/install/)
- Node.js (if running locally without Docker)

---

## **Backend - Docker Setup**

### **1. Create a Dockerfile in the Backend Folder**
Before building the Dockerfile, ensure MongoDB is running and create a `.env` file inside the backend folder with:
```
PORT=5000
```

**Dockerfile (backend):**
```dockerfile
FROM node:18

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5000

CMD ["node", "server.js"]
```

### **2. Build the Docker Image**
Run the following command to build the backend image:
```bash
docker build -t mean-backend:latest .
```

### **3. Run the Docker Container**
```bash
docker run -d -p 5000:5000 mean-backend:latest
```

---

## **Frontend - Docker Setup**

### **1. Create a Dockerfile in the Frontend Folder**
Ensure MongoDB is running and update `tutorial.service.ts` to change the base URL port from `8080` to `5000`.

**Dockerfile (frontend):**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build --prod

FROM nginx:alpine
COPY --from=builder /app/dist/angular-15-crud /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### **2. Build the Docker Image**
```bash
docker build -t mean-frontend:latest .
```

### **3. Run the Docker Container**
```bash
docker run -d -p 8081:80 mean-frontend:latest
```

---

## **Docker Compose Setup**

### **1. Create `docker-compose.yml`**
This file defines the backend, frontend, and MongoDB services.

```yaml
version: "3"
services:
  backend:
    image: mean-backend:latest
    container_name: mean-backend
    ports:
      - "5000:5000"
    depends_on:
      - mongo
    environment:
      - MONGO_URL=mongodb://mongo:27017/dd_db
  
  frontend:
    image: mean-frontend:latest
    container_name: mean-frontend
    ports:
      - "8081:80"
    depends_on:
      - backend
      
  mongo:
    image: mongo:latest
    container_name: mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

### **2. Deploy the Application using Docker Compose**
```bash
docker-compose up -d
```

### **3. Stop and Remove Containers**
```bash
docker-compose down
```

## **Nginx Reverse Proxy Setup**
Create an Nginx config file `/etc/nginx/sites-available/mean-app:`
```nginx
server {
    listen 80;
    
    location / {
        proxy_pass http://localhost:80;
    }

    location /api/ {
        proxy_pass http://localhost:5000/;
    }
}
```

Enable and restart:
```bash
sudo ln -s /etc/nginx/sites-available/mean-app /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

## **CI/CD Pipeline Setup**
**GitHub Actions (`.github/workflows/deploy.yml`)**

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Build and Push Backend
        run: |
          cd backend
          docker build -t surendergupta/mean-backend:latest .
          docker push surendergupta/mean-backend:latest

      - name: Build and Push Frontend
        run: |
          cd frontend
          docker build -t surendergupta/mean-frontend:latest .
          docker push surendergupta/mean-frontend:latest

      - name: Deploy on VM
        uses: appleboy/ssh-action@v0.1.4
        with:
          host: ${{ secrets.VM_HOST }}
          username: ${{ secrets.VM_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            docker pull surendergupta/mean-backend:latest
            docker pull surendergupta/mean-frontend:latest
            cd /home/ubuntu/mean-app/
            docker-compose down
            docker-compose up -d

```
---

## **Notes & Troubleshooting**

### **MongoDB Connection Issues**
- **For Docker Compose Deployment:** Use `mongodb://mongo:27017/dd_db`
- **For Local MongoDB Deployment:** Use `mongodb://localhost:27017/dd_db`
- **For MongoDB Atlas Deployment:** Use the provided Atlas connection string

### **CORS Configuration (Backend - `server.js`)**
Ensure CORS is enabled to allow frontend access:
```js
const cors = require("cors");
app.use(cors());
```

### **Fix for `strictQuery` Warning in Mongoose**
Set `strictQuery` to `false` in `server.js` to avoid issues with MongoDB queries:
```js
mongoose.set("strictQuery", false);
```

---

## **Access the Application**
- **Backend API:** http://localhost:5000/api/tutorials
- **Frontend UI:** http://localhost:8081

---

## **Author**
- **Surender Gupta** 🚀
- Contact: [Github](https://github.com/surendergupta) | [LinkedIn](https://www.linkedin.com/in/surender-gupta/)

---

## **License**
This project is licensed under the MIT License. Feel free to modify and use it as needed.

