### Task 1: Pick Your App
I have picked an app with frontend and db https://github.com/hamzaavvan/library-management-system

### Task 2: Write the Dockerfile
```FROM python:3.11-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```
### Task 3: Add Docker Compose
```
services:
  db:
    image: mysql:8.0
    container_name: flask_db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-root}
      MYSQL_DATABASE: ${MYSQL_DATABASE:-lms}
      MYSQL_USER: ${MYSQL_USER:-lmsuser}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD:-lmspassword}
    volumes:
      - mysql_data:/var/lib/mysql
      - ./db/lms.sql:/docker-entrypoint-initdb.d/lms.sql:ro
    networks:
      - lms_network
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h 127.0.0.1 -u root -p$$MYSQL_ROOT_PASSWORD --silent"]
      interval: 5s
      timeout: 5s
      retries: 20
      start_period: 10s

  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: rahulsingh2k/library-management-system:latest
    container_name: flask_app
    restart: unless-stopped
    environment:
      MYSQL_HOST: db
      MYSQL_PORT: 3306
      MYSQL_USER: ${MYSQL_USER:-lmsuser}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD:-lmspassword}
      MYSQL_DATABASE: ${MYSQL_DATABASE:-lms}
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "5000:5000"
    networks:
      - lms_network

volumes:
  mysql_data:

networks:
  lms_network:
    driver: bridge
```

### Task 4: Ship It
https://hub.docker.com/repository/docker/rahulsingh2k/library-management-system/general

### Task 5: Test the Whole Flow
the flow worked fine after pulling the image from docker hub and running it using docker compose up command.
