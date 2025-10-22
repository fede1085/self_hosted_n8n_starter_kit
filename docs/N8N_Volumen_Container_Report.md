@# 💾 Persistencia de Datos en n8n

Este documento explica cómo y dónde se guardan los datos de n8n (workflows, credenciales, etc.) en la configuración del proyecto `self_hosted_n8n_starter_kit`.

## 1. ¿Hay un volumen persistente para workflows y credenciales?

**Sí, absolutamente.** La configuración utiliza un volumen de Docker para asegurar que todos tus workflows, credenciales y configuraciones de n8n se guarden de forma permanente, incluso si detienes o reinicias los contenedores.

En el archivo `docker-compose.yml`, la sección del servicio `n8n` tiene esta línea clave:

```yaml
services:
  n8n:
    # ... otras configuraciones
    volumes:
      - n8n_storage:/home/node/.n8n
```

Esto le indica a Docker que debe mapear el directorio interno de n8n donde se guardan todos los datos (`/home/node/.n8n`) a un volumen administrado por Docker llamado `n8n_storage`.

El informe de estado (`docs/Estado_Actual_Proyecto_V0.1.md`) también lo confirma:

| Volumen       | Uso                      | Persistencia  |
| ------------- | ------------------------ | ------------- |
| `n8n_storage` | Workflows y credenciales | ✅ Persistente |
> **Nota:** Docker Compose antepone el nombre del directorio del proyecto al nombre del volumen. Por lo tanto, al listar los volúmenes con `docker volume ls`, es posible que lo veas como `self_hosted_n8n_starter_kit_n8n_storage`.

## 2. ¿Dónde se encuentra físicamente ese volumen?

Aquí es donde está la distinción importante:

*   **No está en tu carpeta de Windows:** El volumen `n8n_storage` **no se guarda** directamente en tu directorio de proyecto `C:\dev_space\docker_projects\self_hosted_n8n_starter_kit`.
*   **Está en el sistema de archivos de WSL para Docker:** Este tipo de volumen (llamado "named volume" o volumen nombrado) es gestionado internamente por Docker. Si estás usando Docker Desktop con el backend de WSL 2 (lo cual es estándar en Windows), estos datos residen dentro del sistema de archivos de la máquina virtual de Linux que Docker utiliza. Es una ubicación interna y administrada por Docker para garantizar la integridad y el rendimiento, separada de los archivos de tu proyecto en Windows.

La otra carpeta que sí ves en tu directorio de Windows es la carpeta `shared`, gracias a esta línea en el `docker-compose.yml`:

```yaml
    volumes:
      - ./shared:/data/shared
```

Esta es una carpeta diferente, diseñada para que puedas colocar archivos (como PDFs, CSVs, etc.) en tu explorador de Windows y que el contenedor de n8n pueda acceder a ellos desde la ruta `/data/shared` para usarlos en tus workflows.

## Resumen

*   ✅ **Tus workflows y credenciales están seguros y son persistentes** gracias al volumen `n8n_storage`.
*   📍 Este volumen **vive dentro del entorno de WSL/Linux que Docker administra**, no directamente en tu carpeta de proyecto en Windows.
*   📁 La carpeta `self_hosted_n8n_starter_kit` en Windows contiene los archivos de configuración del proyecto y la carpeta `shared` para intercambiar archivos con n8n, pero no los datos internos de la base de datos o los workflows.
