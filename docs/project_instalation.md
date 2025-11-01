````markdown
# Informe técnico – Restauración y despliegue n8n (via Docker + Cloudflare Tunnel)

## 1. Contexto
- Punto de partida: n8n **no arrancaba** (“Start not found / pensando”).
- Causa raíz inicial: archivo **`.env` mal formado** (líneas con `:`, duplicados, `...`) → n8n no leía bien las variables.
- Objetivo: volver a levantar n8n **limpio**, con **Docker Compose**, **Postgres**, **Qdrant**, **túnel Cloudflare** y **recuperar datos viejos** (workflows + credenciales cifradas).

---

## 2. Descarga del proyecto
1. Se clonó el repo sin carpeta madre:
   ```bash
   git clone https://github.com/fede1085/Self_Hosted_N8N_Starter_Kit.git .
````

(clonado directo en la carpeta de trabajo).

2. Se creó / normalizó el `.env` (modo local primero):

   ```env
   POSTGRES_USER=n8n
   POSTGRES_PASSWORD=n8n
   POSTGRES_DB=n8n
   N8N_ENCRYPTION_KEY=rFXdsilK7CYEMIqZq2fwR3aJbHwzRIUoRy8yLVGBPlQ=
   N8N_USER_MANAGEMENT_JWT_SECRET=jwtsecret123
   N8N_SECURE_COOKIE=false
   N8N_TRUST_PROXY=false
   N8N_PROTOCOL=http
   N8N_HOST=localhost
   WEBHOOK_URL=http://localhost:5678/
   N8N_EDITOR_BASE_URL=http://localhost:5678
   N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=false
   ```

---

## 3. Docker Compose – versión base

Se dejó el `docker-compose.yml` así (estructura clave):

```yaml
name: self_hosted_n8n_starter_kit

services:
  postgres:
    image: postgres:16-alpine
    ...
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}

  n8n:
    image: n8nio/n8n:1.106.3   # luego se actualizó a latest
    platform: linux/amd64
    env_file: .env
    depends_on:
      postgres:
        condition: service_healthy
    ports:
      - "5678:5678"
    environment:
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_PORT: 5432
      DB_POSTGRESDB_DATABASE: ${POSTGRES_DB}
      DB_POSTGRESDB_USER: ${POSTGRES_USER}
      DB_POSTGRESDB_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - self_hosted_n8n_starter_kit_n8n_storage:/home/node/.n8n
      - ./shared:/data/shared

  qdrant:
    image: qdrant/qdrant
    ...
    ports:
      - "6333:6333"

volumes:
  self_hosted_n8n_starter_kit_postgres_storage:
  self_hosted_n8n_starter_kit_n8n_storage:
  self_hosted_n8n_starter_kit_qdrant_storage:

networks:
  demo:
```

Se levantó con:

```bash
docker compose up -d
docker compose logs -f n8n
```

Resultado: **n8n arrancó y mostró el panel vacío** → ✅ base OK.

---

## 4. Paso a HTTPS con Cloudflare Tunnel

### 4.1 Cambio de `.env` a modo túnel

Se pasó el `.env` a HTTPS/dominio:

```env
POSTGRES_USER=n8n
POSTGRES_PASSWORD=n8n
POSTGRES_DB=n8n

N8N_ENCRYPTION_KEY=rFXdsilK7CYEMIqZq2fwR3aJbHwzRIUoRy8yLVGBPlQ=
N8N_USER_MANAGEMENT_JWT_SECRET=jwtsecret123

N8N_PROTOCOL=https
N8N_HOST=n8ntunnel.federicomosqueira.site
WEBHOOK_URL=https://n8ntunnel.federicomosqueira.site/
N8N_OAUTH2_CALLBACK_URL=https://n8ntunnel.federicomosqueira.site/rest/oauth2-credential/callback

N8N_SECURE_COOKIE=true
N8N_PROXY_HOPS=1
N8N_TRUST_PROXY=false
```

Reinicio:

```bash
docker compose down
docker compose up -d
```

### 4.2 Ejecución manual del túnel (Windows)

Se detectó que el túnel ya existía en Cloudflare:

```bash
cloudflared.exe tunnel run 7bfbfd83-0866-4cef-9bc3-6650f2b55977
```

→ Funcionó, se accedió a `https://n8ntunnel.federicomosqueira.site`.

---

## 5. Problema: Cloudflare dentro de Docker se apagaba

**Síntoma:** `tunnel credentials file ... doesn't exist`
**Causa:** el contenedor no veía las credenciales de Windows (`C:\Users\fede\.cloudflared`).

### 5.1 Solución

1. Se copió la carpeta de credenciales al proyecto:

   ```text
   C:\dev_space\docker_projects\self_hosted_n8n_starter_kit\.cloudflared
   ```

2. Se actualizó el servicio en `docker-compose.yml`:

```yaml
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    command: tunnel --credentials-file /etc/cloudflared/7bfbfd83-0866-4cef-9bc3-6650f2b55977.json run 7bfbfd83-0866-4cef-9bc3-6650f2b55977
    volumes:
      - ./.cloudflared:/etc/cloudflared
    networks: [demo]
```

3. Reinicio:

```bash
docker compose down
docker compose up -d
docker compose logs -f cloudflared
```

4. Resultado:

   * Cloudflare registró 4 conexiones QUIC a mrs/bru
   * Túnel **estable** dentro de Docker → ✅

---

## 6. Actualización de n8n a última versión

Se cambió:

```yaml
image: n8nio/n8n:1.106.3
```

por

```yaml
image: n8nio/n8n:latest
```

y se ejecutó:

```bash
docker compose pull n8n
docker compose up -d
docker compose logs -f n8n
```

Resultado en logs:

```text
Version: 1.117.3
Editor is now accessible via:
https://n8ntunnel.federicomosqueira.site
```

→ ✅ última versión corriendo detrás del túnel.

---

## 7. Restauración de datos viejos (workflows + credenciales)

**Importante:** el backup viejo (`n8n_pg_20251022.dump`) tenía la **misma** `N8N_ENCRYPTION_KEY` → se pueden leer credenciales.

### 7.1 Copiar backup al contenedor

(ya estando el dump en la carpeta del proyecto)

```bash
docker cp n8n_pg_20251022.dump postgres:/tmp/n8n_pg_20251022.dump
```

### 7.2 Intento 1 (fallido pero esperado)

```bash
psql -U n8n -d n8n -f /tmp/n8n_pg_20251022.dump
```

→ Postgres respondió: “custom-format dump” → hay que usar `pg_restore`.

### 7.3 Restauración real

Entrar al contenedor:

```bash
docker compose exec postgres bash
```

Restaurar:

```bash
pg_restore -U n8n -d n8n /tmp/n8n_pg_20251022.dump
```

Esto dio **muchos** mensajes de:

> `constraint ... already exists`

🔎 Interpretación:

* La DB actual ya tenía el esquema más nuevo (por las migraciones de la 1.117.3).
* El dump viejo quería recrear las FKs.
* **Pero** los datos se importaron.

Salir:

```bash
exit
```

### 7.4 Recargar n8n para leer la DB restaurada

```bash
docker compose restart n8n
```

Resultado: al entrar al panel → **aparecen los workflows y credenciales del n8n viejo**. ✅

---

## 8. Estado final

**Servicios activos:**

* ✅ `postgres` → base existente, sin reinicios
* ✅ `n8n` → v1.117.3, accesible por HTTPS (Cloudflare)
* ✅ `qdrant` → listo en 6333
* ✅ `cloudflared` → túnel estable con credenciales montadas desde `.cloudflared/`
* ✅ credenciales de n8n legibles (misma `N8N_ENCRYPTION_KEY`)

**Acceso:**

* Interno: `http://localhost:5678`
* Público / HTTPS: `https://n8ntunnel.federicomosqueira.site`

---

## 9. Recomendaciones pendientes

1. **Quitar warnings de n8n** agregando al `.env`:

   ```env
   N8N_RUNNERS_ENABLED=true
   N8N_BLOCK_ENV_ACCESS_IN_NODE=false
   N8N_GIT_NODE_DISABLE_BARE_REPOS=true
   ```

2. **Backup periódico** del volumen de n8n:

   ```bash
   docker run --rm -v self_hosted_n8n_starter_kit_n8n_storage:/data -v %cd%:/backup alpine tar czf /backup/n8n_backup.tar.gz -C /data .
   ```

3. **Backup de Postgres** (por si volvemos a restaurar):

   ```bash
   docker compose exec postgres pg_dump -U n8n -d n8n -Fc -f /tmp/n8n_latest.dump
   docker cp postgres:/tmp/n8n_latest.dump .
   ```

---

## 10. Resumen ultra corto (para vos)

* Clonaste repo ✅
* Arreglamos `.env` ✅
* Compose con Postgres + n8n + Qdrant ✅
* Metimos Cloudflare **dentro** del Compose (no solo manual) ✅
* Montamos credenciales `.cloudflared` ✅
* Actualizamos n8n a `latest` ✅
* Restauramos tu backup viejo `.dump` ✅
* Mismo `N8N_ENCRYPTION_KEY` → credenciales vuelven ✅
* Ahora n8n funciona por **HTTPS público** ✅

```
```
