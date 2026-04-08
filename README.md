🚀 Traefik Reverse Proxy with Docker / Podman

Setup sederhana Traefik sebagai reverse proxy menggunakan Docker atau Podman.
Cukup jalankan satu perintah, dan semua service langsung aktif.

📦 Fitur
Auto reverse proxy via Docker labels
HTTPS ready (Let's Encrypt optional)
Dashboard Traefik
Support multi-service
Bisa pakai Docker atau Podman
🧱 Struktur Project
.
├── docker-compose.yml
├── traefik/
│   ├── traefik.yml
│   └── acme.json
└── services/
    └── app/
        └── Dockerfile
⚙️ Konfigurasi
1. docker-compose.yml
version: "3.9"

services:
  traefik:
    image: traefik:v3.0
    container_name: traefik
    restart: unless-stopped
    command:
      - "--api.dashboard=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
    ports:
      - "80:80"
      - "8080:8080" # dashboard
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik/traefik.yml:/traefik.yml
      - ./traefik/acme.json:/acme.json

  app:
    build: ./services/app
    container_name: myapp
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`localhost`)"
      - "traefik.http.routers.myapp.entrypoints=web"
    restart: unless-stopped
2. traefik/traefik.yml
api:
  dashboard: true

entryPoints:
  web:
    address: ":80"

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
3. traefik/acme.json
touch traefik/acme.json
chmod 600 traefik/acme.json
4. Contoh App (services/app/Dockerfile)
FROM nginx:alpine
COPY . /usr/share/nginx/html
▶️ Cara Menjalankan
Docker
docker-compose up -d --build
Podman
podman-compose up -d --build
🌐 Akses
App: http://localhost
Traefik Dashboard: http://localhost:8080
🔐 (Opsional) HTTPS dengan Let's Encrypt

Tambahkan ke traefik.yml:

certificatesResolvers:
  letsencrypt:
    acme:
      email: your-email@example.com
      storage: acme.json
      httpChallenge:
        entryPoint: web

Dan update label service:

- "traefik.http.routers.myapp.entrypoints=websecure"
- "traefik.http.routers.myapp.tls.certresolver=letsencrypt"
🧪 Tips
Gunakan domain real (bukan localhost) untuk HTTPS
Pastikan port 80 & 443 terbuka

Untuk Podman, gunakan socket yang sesuai:

/run/podman/podman.sock
🧹 Stop & Cleanup
docker-compose down

atau

podman-compose down
📌 Catatan
Traefik membaca service otomatis dari Docker labels
