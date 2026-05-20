# 🔌 Solución: Error "Connection lost" en n8n

## 📌 Problema detectado
- Al hacer clic en **Save** en un workflow de n8n, aparecía el mensaje `Connection lost` y no se guardaban los cambios.
- Los logs de n8n mostraban repetidamente:
Origin header does NOT match the expected origin.
(Origin: "https://n8n.unisasalud.com" → Expected: "n8n_demo")

text

- La causa raíz era una **discrepancia en el header `Origin`** entre el proxy inverso (Nginx) y n8n. n8n esperaba el origen interno `n8n_demo` pero recibía `https://n8n.unisasalud.com`, lo que bloqueaba las conexiones WebSocket.

## 🛠️ Solución aplicada

### 1. Verificar variables de entorno de n8n
Comprobamos que las variables necesarias estuvieran correctas:

```bash
docker exec n8n-container env | grep -E "N8N_HOST|N8N_EDITOR_BASE_URL|N8N_PROTOCOL"
Salida esperada:

text
N8N_HOST=n8n.unisasalud.com
N8N_EDITOR_BASE_URL=https://n8n.unisasalud.com/
N8N_PROTOCOL=https
2. Editar la configuración de Nginx
Archivo: /etc/nginx/sites-available/unisasalud.conf

En el bloque server_name n8n.unisasalud.com, dentro de location /, se forzaron los headers Host y Origin al dominio real:

nginx
location / {
    proxy_pass http://n8n_demo;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';

    # 🔧 Corrección clave
    proxy_set_header Host n8n.unisasalud.com;
    proxy_set_header Origin https://n8n.unisasalud.com;

    proxy_cache_bypass $http_upgrade;
    proxy_redirect off;
}
3. Validar y recargar Nginx
bash
sudo nginx -t          # Verificar sintaxis
sudo systemctl reload nginx
4. (Opcional) Cambiar backend de WebSocket a SSE
En caso de que el error persista, se puede cambiar el sistema de comunicación en tiempo real de n8n de WebSockets a Server‑Sent Events (SSE).

Esto se logra añadiendo en el archivo docker-compose.n8n.yml:

yaml
environment:
  - N8N_PUSH_BACKEND=sse
Luego recrear el contenedor:

bash
docker compose up -d --force-recreate n8n
5. Verificación final
Acceder a n8n (https://n8n.unisasalud.com).

Crear o editar un workflow y presionar Save.

El error Connection lost ya no aparece; el workflow se guarda correctamente.

📁 Archivos modificados
/etc/nginx/sites-available/unisasalud.conf

(Opcional) docker-compose.n8n.yml

✅ Resultado
Las conexiones WebSocket entre el navegador y n8n ahora son estables.

El header Origin coincide con lo que n8n espera.

Se eliminaron los errores de validación de origen en los logs.