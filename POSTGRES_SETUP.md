cat << 'EOF' > POSTGRES_SETUP.md
# 🐘 PostgreSQL Setup & Deployment Guide

This document details the PostgreSQL configuration and database structure for the `ApexTheDestroyer/n8n-email-summarizer` project.

---

## 1. Overview & Connection Info

PostgreSQL runs as a dedicated service alongside n8n inside Docker Compose. It serves as the primary system database for storing workflow states, execution data, and user credentials.

* **Container Name:** `n8n_postgres`
* **Host:** `postgres` *(internal Docker network)*
* **Port:** `5432`
* **Database Name:** `n8n_db`
* **Database User:** `n8n_user`
* **Database Password:** `n8n_secure_password_123`

---

## 2. Docker Compose Configuration

The following configuration is taken directly from `docker-compose.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: n8n_postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: n8n_user
      POSTGRES_PASSWORD: n8n_secure_password_123
      POSTGRES_DB: n8n_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U n8n_user -d n8n_db"]
      interval: 5s
      timeout: 5s
      retries: 5

  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_PORT: 5432
      DB_POSTGRESDB_DATABASE: n8n_db
      DB_POSTGRESDB_USER: n8n_user
      DB_POSTGRESDB_PASSWORD: n8n_secure_password_123
