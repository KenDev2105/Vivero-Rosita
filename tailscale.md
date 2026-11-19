## Despliegue en VM con Tailscale (Vía SSH)

Cuando te conectes por SSH a la máquina virtual y quieras exponer el sitio directamente en `https://100.109.236.53`:

### 1. Regenerar el certificado SSL incluyendo la IP de Tailscale

Desde la carpeta raíz `Vivero-Rosita/`:

```bash
mkcert -cert-file web/certs/server.crt -key-file web/certs/server.key localhost 127.0.0.1 100.109.236.53
```

### 2. Activar las líneas de Tailscale automáticamente (o editar manual)

Ejecuta estos comandos para descomentar `server_name` y cambiar el puerto a `100.109.236.53:443:443`:

```bash
sed -i 's/# server_name 100.109.236.53;/server_name 100.109.236.53;/' nginx/nginx.conf
sed -i 's/- "8088:443"/# - "8088:443"/' docker-compose.yml
sed -i 's/# - "100.109.236.53:443:443"/- "100.109.236.53:443:443"/' docker-compose.yml
```

*(Si prefieres editarlos manualmente, verifica que queden así)*:

- **En `nginx/nginx.conf`:**

```nginx
server_name 100.109.236.53;
```

- **En `docker-compose.yml`:**

```yaml
    ports:
      # - "8088:443"
      - "100.109.236.53:443:443"
```

### 3. Reconstruir la imagen y recrear el clúster

Es necesario incluir `--build` para que los 3 nodos Apache empaqueten los certificados nuevos dentro de la imagen `vivero-rosita`:

```bash
docker compose up -d --build --force-recreate
```

---

## Verificación y Pruebas

### 1. Estado y salud de los contenedores

```bash
docker compose ps
```

- **Resultado esperado:** `nodo1`, `nodo2` y `nodo3` deben aparecer como `Up ... (healthy)` y `lb` como `Up ...`.

### 2. Prueba de balanceo Round Robin por HTTPS

En la VM (con Tailscale activo):

```bash
for i in {1..6}; do curl -sIk https://100.109.236.53 | grep -E "HTTP/|X-Nodo"; done
```

En local (antes de activar Tailscale, usando el puerto `8088`):

```bash
for i in {1..6}; do curl -sIk https://localhost:8088 | grep -E "HTTP/|X-Nodo"; done
```

- **Resultado esperado:** Respuestas `HTTP/1.1 200 OK` alternando `X-Nodo` entre `172.29.0.11:443`, `172.29.0.12:443` y `172.29.0.13:443`.

### 3. Verificar que HTTP plano (puerto 80) está cerrado

```bash
curl -I http://100.109.236.53
```

- **Resultado esperado:** Error de conexión rechazada (`Connection refused`).

### 4. Prueba de tolerancia a fallos (Failover)

```bash
docker compose stop nodo2
for i in {1..6}; do curl -sIk https://100.109.236.53 | grep -E "HTTP/|X-Nodo"; done
docker compose start nodo2
```

### 5. Depuración directa de nodos individuales (solo desde la VM/Host)

```bash
curl -Ik https://127.0.0.1:9005
curl -Ik https://127.0.0.1:9006
curl -Ik https://127.0.0.1:9007
```

---

## Solución de Problemas Comunes

- **Error `Pool overlaps with other one on this address space`:** Otro proyecto dejó ocupada la subred `172.29.0.0/24`. Limpia redes sin uso con `docker network prune` o cambia el tercer octeto en `docker-compose.yml` (ej. `172.30.0.0/24`).
- **Error de sintaxis YAML al descomentar `ports` en `lb`:** Asegúrate de que `- "100.109.236.53:443:443"` tenga exactamente la misma alineación vertical (6 espacios) que `# - "8088:443"`.