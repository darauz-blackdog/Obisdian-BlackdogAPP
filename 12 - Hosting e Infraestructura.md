---
tags: [blackdog-app, hosting, infrastructure]
created: 2026-02-15
status: planning
---

# Hosting e Infraestructura

## Arquitectura de servidores

### Fase inicial (MVP)

Usar el **mismo VPS** que PeopleBD para minimizar costos:

```
VPS actual
├── Puerto 3000 → PeopleBD Production (people-backend.service)
├── Puerto 3001 → PeopleBD Staging (people-prueba-backend.service)
└── Puerto 3002 → BlackDog API (blackdog-api.service)    ← NUEVO
```

### Servicio systemd

```ini
# /etc/systemd/system/blackdog-api.service
[Unit]
Description=BlackDog App API
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/blackdog-api
ExecStart=/usr/bin/npx tsx src/server.ts
Restart=on-failure
RestartSec=10
Environment=NODE_ENV=production
Environment=PORT=3002
EnvironmentFile=/opt/blackdog-api/.env

[Install]
WantedBy=multi-user.target
```

### Deploy script

```bash
# /opt/update-blackdog-api.sh
#!/bin/bash
cd /opt/blackdog-api
git pull origin main
npm install
sudo systemctl restart blackdog-api
echo "BlackDog API deployed!"
```

## Dominio y SSL

### Temporal (sin dominio)
- API accesible vía IP directa: `http://IP:3002`
- Solo para desarrollo/testing

### Futuro (con dominio)
- Comprar dominio: `blackdogapp.com` o `app.blackdogpanama.com`
- Subdominio API: `api.blackdogapp.com`
- SSL con Let's Encrypt (certbot)
- Nginx como reverse proxy:

```nginx
# /etc/nginx/sites-available/blackdog-api
server {
    listen 443 ssl;
    server_name api.blackdogapp.com;

    ssl_certificate /etc/letsencrypt/live/api.blackdogapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.blackdogapp.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3002;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## Servicios externos

| Servicio | Plan | Costo estimado |
|----------|------|----------------|
| **Supabase** | Free → Pro ($25/mes) | $0-25/mes |
| **Firebase** | Spark (free) | $0/mes |
| **Google Maps API** | Free tier (28K loads/mes) | $0/mes |
| **Apple Developer** | Individual | $99/año |
| **Google Play** | Developer | $25 una vez |
| **Dominio** | .com | ~$12/año |
| **VPS adicional** (cuando escale) | 4GB RAM | ~$20-40/mes |

### Costo total estimado MVP: ~$25-50/mes + $136/año (stores + dominio)

## Migración futura (cuando escale)

Cuando la app tenga tráfico considerable:

1. **Mover backend a VPS dedicado** (4GB+ RAM)
2. **Agregar Redis** para caching en memoria (sesiones, productos hot)
3. **CDN para imágenes** (Cloudflare o Supabase Storage CDN)
4. **Load balancer** si se necesitan múltiples instancias
5. **Monitoring**: Grafana + Prometheus o servicio como Datadog

## Backups

- **Supabase**: Backups automáticos incluidos en plan Pro
- **Backend code**: Git (repositorio)
- **Env vars**: Documentados en password manager (no en Git)
- **Odoo**: Ya tiene sus propios backups (responsabilidad del hosting de Odoo)

## Monitoreo

| Qué | Herramienta |
|-----|------------|
| App crashes | Firebase Crashlytics |
| API uptime | UptimeRobot (free) o similar |
| API logs | Pino → archivos + rotación |
| Sync jobs | Tabla sync_logs en Supabase |
| Performance | Firebase Performance Monitoring |
