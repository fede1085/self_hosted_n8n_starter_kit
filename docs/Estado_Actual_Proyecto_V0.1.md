🧾 **Informe de Estado Técnico: `self_hosted_n8n_starter_kit`**

**Fecha:** 24 de julio de 2024  
**Proyecto Analizado:** `self_hosted_n8n_starter_kit`

---

## 🧱 1. Infraestructura Base

El proyecto se despliega como un stack de microservicios autocontenidos, orquestados por Docker Compose. La arquitectura está diseñada para ser funcional y privada desde el primer momento, alineada con los principios del proyecto.

*   **Método de Despliegue:** Docker Compose.
*   **Archivos de Configuración:**
    *   `docker-compose.yml`: Define los servicios, redes y volúmenes.
    *   `.env`: Almacena las variables de entorno y secretos (generado a partir de `.env.example`).
*   **Red Docker:** Todos los servicios se comunican a través de una red interna aislada llamada `demo`, permitiendo la resolución de nombres entre contenedores (ej. `postgres`, `ollama`).

---

## ⚙️ 2. Contenedores y Puertos Activos

El entorno está configurado para levantar cuatro servicios principales, cada uno encapsulado en su propio contenedor Docker, formando un ecosistema de IA local completo.

| Nombre del Servicio | Imagen Docker | Puerto Expuesto (Host) | Estado Esperado | Descripción |
| :------------------ | :-------------------- | :----------------------- | :---------------- | :---------------------------------------------------------------- |
| **n8n**             | `n8nio/n8n:latest`    | `5678`                   | 🟢 **Up**         | Plataforma de automatización y orquestación de workflows.         |
| **postgres**        | `postgres:16-alpine`  | `(interno)`              | 🟢 **Healthy**    | Base de datos relacional para la persistencia de datos de n8n.    |
| **qdrant**          | `qdrant/qdrant`       | `6333`                   | 🟢 **Up**         | Base de datos vectorial para almacenar y consultar embeddings.      |
| **ollama**          | `ollama/ollama:latest`| `11434`                  | 🟢 **Up**         | Servidor para ejecutar modelos de lenguaje grandes (LLMs) localmente. |

**Puntos de Acceso (Endpoints):**
*   **n8n (Editor/UI):** `http://localhost:5678`
*   **Qdrant (API/UI):** `http://localhost:6333`
*   **Ollama (API):** `http://localhost:11434`

---

## 💾 3. Volúmenes de Datos y Persistencia

La persistencia de datos es un pilar del proyecto, garantizando que los workflows, credenciales, modelos de IA y bases de datos no se pierdan al detener o reiniciar los contenedores. Esto se logra mediante volúmenes nombrados de Docker.

| Volumen Nombrado | Servicio Asociado | Uso Principal | Persistencia |
| :--------------- | :---------------- | :------------------------------------ | :------------- |
| `n8n_storage`    | `n8n`             | Workflows, credenciales y configuración | ✅ **Persistente** |
| `postgres_storage` | `postgres`        | Datos de la base de datos PostgreSQL  | ✅ **Persistente** |
| `qdrant_storage` | `qdrant`          | Vectores y colecciones de Qdrant      | ✅ **Persistente** |
| `ollama_storage` | `ollama`          | Modelos LLM descargados              | ✅ **Persistente** |

Adicionalmente, se utiliza un **bind mount** para la carpeta `./shared`, que mapea una carpeta local del sistema anfitrión a la ruta `/data/shared` dentro del contenedor de `n8n`. Esto facilita el intercambio de archivos (CSVs, PDFs, etc.) entre la máquina local y los workflows.

---

## 🧩 4. Variables del Entorno

La configuración se gestiona principalmente a través de un archivo `.env`. El servicio `n8n` en `docker-compose.yml` muestra una configuración extensa para conectarse a la base de datos y definir su comportamiento público.

*   **Base de Datos:** Las variables `DB_TYPE`, `DB_POSTGRESDB_HOST`, `DB_POSTGRESDB_USER`, etc., configuran n8n para usar el contenedor `postgres` como su base de datos.
*   **Seguridad:** `N8N_ENCRYPTION_KEY` y `N8N_USER_MANAGEMENT_JWT_SECRET` son cruciales para la encriptación de credenciales y la gestión de sesiones.
*   **Acceso Externo:** Las variables como `N8N_HOST`, `WEBHOOK_URL` y `N8N_EDITOR_BASE_URL` están configuradas para un dominio específico (`n8ntunnel.federicomosqueira.site`), indicando un uso detrás de un túnel o proxy inverso. Para un uso puramente local, estas variables podrían necesitar ser ajustadas o eliminadas.
*   **Entorno:** `NODE_ENV: production` indica que n8n se ejecuta en modo de producción, lo cual es una buena práctica para el rendimiento y la seguridad.

---

## 🛠️ 5. Configuración en Docker Compose

El archivo `docker-compose.yml` está bien estructurado y sigue las mejores prácticas para un entorno de desarrollo y experimentación.

*   **Dependencias de Servicio:** El servicio `n8n` utiliza `depends_on` con la condición `service_healthy` para esperar a que `postgres` esté completamente listo antes de iniciar, previniendo errores de conexión.
*   **Healthcheck:** El servicio `postgres` incluye un `healthcheck` que verifica activamente si la base de datos está lista para aceptar conexiones, lo que robustece el arranque del stack.
*   **Imágenes:** Se utilizan imágenes `:latest` para `n8n` y `ollama`, lo que facilita obtener las últimas características, pero puede introducir cambios inesperados en actualizaciones. `postgres` usa una versión fija (`16-alpine`), lo cual es bueno para la estabilidad.
*   **Perfiles (Profiles):** El `README.md` menciona el uso de perfiles (`--profile cpu`, `--profile gpu-nvidia`) para adaptar el despliegue al hardware. Sin embargo, estos perfiles no están definidos en el `docker-compose.yml` base, lo que sugiere que podrían existir en archivos de `override` no incluidos en el contexto, o que es una instrucción para configuraciones más avanzadas.

---

## ✅ 6. Estado General del Sistema

El sistema está diseñado para ser **robusto y resiliente para fines de aprendizaje y experimentación**, tal como se define en `CONTRIBUTING.md`.

*   **Estabilidad:** 🟢 **Alta**. La configuración con `restart: unless-stopped` y el `healthcheck` de Postgres aseguran que el sistema se recupere automáticamente de reinicios o fallos.
*   **Seguridad:** ⚪ **Básica/Local**. La gestión de secretos vía `.env` es adecuada para un entorno local. No se incluyen configuraciones de seguridad avanzadas (como reverse proxies con SSL), ya que está explícitamente fuera del alcance del proyecto.
*   **Preparado para IA:** 🟢 **Sí**. La inclusión nativa de Ollama y Qdrant, junto con el workflow de demostración (`Demo workflow`), permite a los usuarios empezar a construir y experimentar con RAG (Retrieval-Augmented Generation) y otros patrones de IA local de inmediato.

---

## 🧠 7. Recomendaciones Finales

*   **Gestión de Secretos:** Es fundamental que los usuarios modifiquen los valores por defecto del archivo `.env.example` al crear su `.env`, especialmente `N8N_ENCRYPTION_KEY` y las contraseñas de la base de datos, para asegurar su entorno.
*   **Descarga de Modelos:** El primer uso de un workflow que utilice un LLM (ej. `llama3.2:latest` según el workflow de demo) puede ser lento, ya que Ollama necesitará descargar el modelo. Se puede monitorizar el progreso con el comando `docker logs -f ollama`.
*   **Configuración de Red Externa:** Si no se va a usar un túnel como `n8ntunnel.federicomosqueira.site`, se deben comentar o ajustar las variables `N8N_HOST`, `WEBHOOK_URL`, etc., en el `docker-compose.yml` o en el `.env` para que n8n genere las URLs usando `localhost`.
*   **No es para Producción:** Como se reitera en la documentación, este kit es una plataforma de aprendizaje. Para un entorno de producción, se deberían considerar capas adicionales de seguridad, monitorización, backups y alta disponibilidad.

---

## 🧰 8. Script Opcional de Reinicio (Ejemplo para PowerShell)

Para agilizar el ciclo de desarrollo, se puede utilizar un simple script para detener, reconstruir y levantar todo el stack. Este script asume el uso del perfil `cpu` mencionado en el `README.md`.

**`restart-stack.ps1`**
```powershell
# Navega al directorio del proyecto (ajusta la ruta si es necesario)
Set-Location "C:\dev_space\docker_projects\self_hosted_n8n_starter_kit"

# Detiene y elimina los contenedores, redes y volúmenes anónimos
Write-Host "🔵 Deteniendo el entorno..."
docker compose down

# Levanta los servicios en modo detached (segundo plano) usando el perfil de CPU
Write-Host "🟢 Iniciando el entorno con perfil 'cpu'..."
docker compose --profile cpu up -d

# Espera unos segundos para que n8n inicie
Start-Sleep -Seconds 10

# Abre n8n en el navegador por defecto
Start-Process "http://localhost:5678"

Write-Host "✅ Entorno n8n reiniciado. Accede en http://localhost:5678"
```

