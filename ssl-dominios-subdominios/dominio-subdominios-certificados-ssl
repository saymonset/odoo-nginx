

# 1. Para crear el subdominio lead.jumpjibe.com que apunte a tu segunda instancia de Odoo (puertos 28069 y 28072), debes agregar un nuevo bloque server en ## Nginx. Aquí está la configuración:
```bash
# Upstream principal Odoo
upstream odoo {
    server 127.0.0.1:18069;
}

# Upstream longpolling/evented
upstream odoo_longpolling {
    server 127.0.0.1:8072;
}

# Nuevos upstreams para el subdominio leads - CORREGIDO
upstream odoo_leads {
    server 127.0.0.1:28069;
}

upstream odoo_leads_longpolling {
    server 127.0.0.1:28072;
}

# DOMINIO PRINCIPAL - CONFIGURACIÓN MEJORADA
server {
    listen 80;
    server_name jumpjibe.com www.jumpjibe.com;
    include snippets/letsencrypt.conf;
    return 301 https://jumpjibe.com$request_uri;
}

server {
    listen 443 ssl http2;
    server_name jumpjibe.com www.jumpjibe.com;

    ssl_certificate /etc/letsencrypt/live/jumpjibe.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/jumpjibe.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/jumpjibe.com/chain.pem;
    include snippets/ssl.conf;
    include snippets/letsencrypt.conf;

    access_log /var/log/nginx/odoo.access.log;
    error_log /var/log/nginx/odoo.error.log;

    # Cabeceras proxy mejoradas
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $host;

    proxy_read_timeout 720s;
    proxy_connect_timeout 720s;
    proxy_send_timeout 720s;

    # Archivos estáticos
    location ~* /web/static/ {
        proxy_pass http://odoo;
        proxy_cache_valid 200 90m;
        proxy_buffering on;
        expires 864000;
        add_header Cache-Control "public, immutable";
    }

    # HTTP normal
    location / {
        proxy_pass http://odoo;
        proxy_redirect off;
    }

    # Longpolling / Websockets
    location /longpolling/ {
        proxy_pass http://odoo_longpolling;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 900s;
        proxy_connect_timeout 900s;
        proxy_send_timeout 900s;
    }
}

```

# ####################################################################
# Nueva configuración para el subdominio lead.jumpjibe.com
# Nuevos upstreams para el subdominio leads
```bash

# SUBDOMINIO LEAD - CONFIGURACIÓN CORREGIDA
server {
    listen 80;
    server_name lead.jumpjibe.com;
    return 301 https://lead.jumpjibe.com$request_uri;
}

```


# Certificado SSL:
# Si el certificado actual no incluye el subdominio, genera uno nuevo con:
```bash
sudo certbot certonly --agree-tos --email lead@jumpjibe.com --webroot -w /var/lib/letsencrypt/ -d lead.jumpjibe.com
```
# 3 Colocar el 443 con su certificado 
```bash

server {
    listen 443 ssl http2;
    server_name lead.jumpjibe.com;

    # Certificados SSL para el subdominio
    ssl_certificate /etc/letsencrypt/live/lead.jumpjibe.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lead.jumpjibe.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/lead.jumpjibe.com/chain.pem;
    include snippets/ssl.conf;
    include snippets/letsencrypt.conf;

    access_log /var/log/nginx/odoo_leads.access.log;
    error_log /var/log/nginx/odoo_leads.error.log;

    # Cabeceras proxy mejoradas
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $host;

    proxy_read_timeout 720s;
    proxy_connect_timeout 720s;
    proxy_send_timeout 720s;

    # Bye-bye cache
    proxy_buffering off;

    # Archivos estáticos para leads
    location ~* /web/static/ {
        proxy_pass http://odoo_leads;
        proxy_cache_valid 200 90m;
        proxy_buffering on;
        expires 864000;
        add_header Cache-Control "public, immutable";
    }

    # Configuración principal MEJORADA
    location / {
        proxy_pass http://odoo_leads;
        proxy_redirect http://odoo_leads/ https://lead.jumpjibe.com/;
        proxy_redirect https://odoo_leads/ https://lead.jumpjibe.com/;
        proxy_redirect /web/ /web/;
    }

    # Longpolling corregido
    location /longpolling/ {
        proxy_pass http://odoo_leads_longpolling;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 900s;
        proxy_connect_timeout 900s;
        proxy_send_timeout 900s;
    }
}

```

# Verificar certificados
```bash
sudo certbot certificates

```
# 1. Ver fecha de expiración del certificado:
```bash
# Ver todos los certificados y sus fechas
sudo certbot certificates

# Ver específicamente el certificado de lead.jumpjibe.com
sudo certbot certificates | grep -A 10 "lead.jumpjibe.com"

# O usar openssl para ver detalles
sudo openssl x509 -in /etc/letsencrypt/live/lead.jumpjibe.com/cert.pem -noout -dates
```
# 2. Renovar el certificado:
```bash
# Renovar solo este certificado
sudo certbot renew --cert-name lead.jumpjibe.com

# O renovar todos los certificados que estén próximos a expirar
```bash
# Renovar solo este certificado
sudo certbot renew --cert-name lead.jumpjibe.com

# O renovar todos los certificados que estén próximos a expirar
sudo certbot renew
# Forzar renovación (incluso si no está próximo a expirar):
sudo certbot renew --cert-name lead.jumpjibe.com --force-renewal
```

# 3. Verificar renovación:
```bash
# Verificar que se renovó correctamente
sudo certbot certificates | grep -A 5 "lead.jumpjibe.com"

# Ver la nueva fecha de expiración
sudo openssl x509 -in /etc/letsencrypt/live/lead.jumpjibe.com/cert.pem -noout -enddate
```
# 4. Renovación automática:
```bash
# Verificar el servicio de renovación automática
sudo systemctl status certbot.timer

# Ver cuándo se ejecutará la próxima renovación automática
sudo systemctl list-timers | grep certbot

# Ejecutar una prueba de renovación (sin guardar)
sudo certbot renew --dry-run
```
# 5. Recargar Nginx después de renovar:
```bash
# Si usas --renew-hook o quieres recargar manualmente
sudo systemctl reload nginx

# Verificar que Nginx use el nuevo certificado
sudo nginx -t
```