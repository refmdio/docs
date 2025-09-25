# Quick Start

## 1. Deploy with docker-compose

1. Create a working directory:
   ```bash
   mkdir ~/refmd && cd ~/refmd
   ```

2. Create `docker-compose.yml`:

   ```yaml
   services:
     postgres:
       image: postgres:15
       environment:
         POSTGRES_DB: refmd
         POSTGRES_USER: refmd
         POSTGRES_PASSWORD: refmd
       ports:
         - "5432:5432"
       volumes:
         - postgres_data:/var/lib/postgresql/data
       healthcheck:
         test: ["CMD-SHELL", "pg_isready -U refmd"]
         interval: 5s
         timeout: 5s
         retries: 5

     api:
       image: ghcr.io/refmdio/refmd/api:latest
       environment:
         RUST_ENV: production
         API_PORT: 8888
         DATABASE_URL: postgresql://refmd:refmd@postgres:5432/refmd
         FRONTEND_URL: https://refmd.example.com
         JWT_SECRET: change-me-please
         ENCRYPTION_KEY: change-me-please
         UPLOAD_MAX_BYTES: 26214400
         RUST_LOG: api=info,axum=info,tower_http=info
         PUBLIC_BASE_URL: https://refmd.example.com
         PLUGINS_DIR: /app/plugins
       ports:
         - "8888:8888"
       depends_on:
         postgres:
           condition: service_healthy
       volumes:
         - refmd_data:/data
         - refmd_plugins:/app/plugins

     app:
       image: ghcr.io/refmdio/refmd/app:latest
       environment:
         VITE_API_BASE_URL: https://refmd.example.com/api
       ports:
         - "3000:80"
       depends_on:
         api:
           condition: service_started

   volumes:
     postgres_data:
     refmd_data:
     refmd_plugins:
   ```

   Update the environment variables before starting containers:
   - `JWT_SECRET`, `ENCRYPTION_KEY`: replace with strong, random values.
   - `FRONTEND_URL`, `PUBLIC_BASE_URL`: set to the public domain for the app (`http://localhost:3000` locally, `https://refmd.example.com` in production).
   - `VITE_API_BASE_URL`: match the API origin exposed to users (e.g. `http://localhost:8888` locally, `https://refmd.example.com/api` behind a proxy).

3. Start the stack:
   ```bash
   docker compose up -d
   ```

4. Check container status:
   ```bash
   docker compose ps
   docker compose logs -f
   ```

5. Open `http://localhost:3000` (or your configured domain) and create the first account.

## 2. Publish behind Nginx reverse proxy

Nginx configuration forwarding traffic to the containers launched above:

```nginx
server {
    listen 80;
    server_name refmd.example.com;

    client_max_body_size 25M;

    location /api/ {
        proxy_pass         http://127.0.0.1:8888/api/;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection "upgrade";
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_buffering    off;
    }

    location / {
        proxy_pass         http://127.0.0.1:3000/;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

## 3. Publish behind Caddy reverse proxy

Example configuration for Caddy:

```caddyfile
refmd.example.com {
    encode zstd gzip

    @api route /api* {
        reverse_proxy 127.0.0.1:8888
    }

    route {
        reverse_proxy 127.0.0.1:3000
    }
}
```
