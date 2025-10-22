🧾 **Informe de Estado Técnico: `self_hosted_n8n_starter_kit`**

**Fecha:** 17 de octubre de 2025  
**Ubicación del proyecto:**  
`C:\dev_space\docker_projects\self_hosted_n8n_starter_kit`

---

## 🧱 1️⃣ Infraestructura base

**Tipo de despliegue:**  
Docker Compose (no DevContainer, no Visual Studio Code activo)

**Archivos principales:**
- `docker-compose.yml` → versión estable final (confirmada)
- `.env` → variables coherentes y funcionales
- Red Docker: `self_hosted_n8n_starter_kit_demo`
- Volúmenes persistentes activos

---

## ⚙️ 2️⃣ Contenedores activos (`docker ps`)

| Nombre | Imagen | Puerto Host | Estado | Descripción |
|--------|--------|-------------|---------|--------------|
| **n8n** | `n8nio/n8n:1.106.3` | 5678 | 🟢 Up | Núcleo principal de n8n (Editor & API) |
| **postgres** | `postgres:16-alpine` | interno (5432) | 🟢 Healthy | Base de datos persistente |
| **qdrant** | `qdrant/qdrant` | 6333 | 🟢 Up | Base vectorial (IA embeddings) |
| **ollama** | `ollama/ollama:latest` | 11434 | 🟢 Up | Servidor local de modelos (Llama, DeepSeek, etc.) |

**Puertos activos:**
- `http://localhost:5678` → n8n  
- `http://localhost:6333` → Qdrant  
- `http://localhost:11434` → Ollama  

---

## 💾 3️⃣ Volúmenes de datos

| Volumen | Uso | Persistencia |
|----------|------|---------------|
| `self_hosted_n8n_starter_kit_n8n_storage` | Workflows y credenciales | ✅ Persistente |
| `self_hosted_n8n_starter_kit_postgres_storage` | Base de datos Postgres | ✅ Persistente |
| `self_hosted_n8n_starter_kit_ollama_storage` | Modelos descargados | ✅ Persistente |
| `self_hosted_n8n_starter_kit_qdrant_storage` | Datos vectoriales | ✅ Persistente |

---

## ⚙️ 4️⃣ Variables del entorno (.env)

```env
POSTGRES_USER=n8n
POSTGRES_PASSWORD=n8n
POSTGRES_DB=n8n
N8N_ENCRYPTION_KEY=superclave123456789xyz
N8N_USER_MANAGEMENT_JWT_SECRET=jwtsecret123
N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
OLLAMA_HOST=ollama:11434
PULSE_OIDC_CLIENT_ID=
PULSE_OIDC_CLIENT_SECRET=
PULSE_OIDC_ISSUER_URL=
PULSE_OIDC_SCOPES=openid email profile


🧩 5️⃣ Configuración Docker Compose (resumen)

Versión de n8n: 1.106.3

Base de datos: Postgres 16

Servicios auxiliares: Qdrant + Ollama

Red compartida: demo

Persistencia total activada

Puerto principal: 5678

Puerto alternativo (5679): eliminado

🧼 6️⃣ Limpieza previa realizada

Eliminado el proyecto self_hosted_ai_starter_kit

Eliminado el contenedor n8n-dev

Desactivado DevContainer de VS Code

Removido conflicto con puerto 5679

✅ 7️⃣ Estado final del sistema
Área	Estado	Comentario
N8N (5678)	🟢 Activo	Funcional y estable
Workflows previos	⚪ Reinicializados	Nuevo entorno limpio
Persistencia	🟢 OK	Volúmenes funcionales
Configuración	🟢 OK	Variables coherentes
Puertos duplicados	🔴 Eliminados	Sin conflictos
Visual Studio DevContainer	🔴 Desactivado	No interfiere más
🧠 8️⃣ Recomendaciones finales

Trabajar solo con Docker Compose desde PowerShell o terminal.

No abrir el proyecto con DevContainer (VS Code).

Para reiniciar n8n en caso de error:

docker compose down
docker compose up -d

🧰 9️⃣ Script opcional: reset_n8n.ps1
cd "C:\dev_space\docker_projects\self_hosted_n8n_starter_kit"
docker compose down
docker compose up -d
Start-Process "http://localhost:5678"


Doble clic en el .ps1 → limpia y reinicia todo automáticamente ⚡

Resumen corto:

Sistema n8n estable y funcional.
Puerto activo: 5678.
Persistencia confirmada.
Entorno dev (5679) eliminado.
Visual Studio DevContainer deshabilitado.
Ollama, Qdrant y Postgres funcionando en red “demo”.