# El Ecosistema Docker

Docker no es solo un comando. Es un conjunto de herramientas.

---

## Componentes principales

```
┌─────────────────────────────────────┐
│           DOCKER DESKTOP             │
│  (Interfaz gráfica opcional)         │
├─────────────────────────────────────┤
│           DOCKER ENGINE              │
│  ┌──────────┐  ┌──────────────────┐ │
│  │  dockerd  │  │   containerd     │ │
│  │  (daemon) │  │  (runtime)       │ │
│  └──────────┘  └──────────────────┘ │
├─────────────────────────────────────┤
│            DOCKER CLI                │
│  Comandos: docker, docker compose    │
└─────────────────────────────────────┘
```

### 1. Docker Engine (el motor)

- **dockerd**: el daemon que corre en segundo plano
- Escucha comandos del CLI y gestiona contenedores
- Usa **containerd** (el runtime) para ejecutar los contenedores

### 2. Docker CLI (tu terminal)

- El comando `docker` que escribes
- Se comunica con el daemon via API
- Lo que usas el 100% del tiempo

### 3. Docker Desktop (interfaz gráfica)

- Opcional, pero útil para ver contenedores, logs, volúmenes
- Disponible en Windows y Mac
- En Linux se usa solo el Engine

---

## Docker Hub

Es el **repositorio público** de imágenes Docker.

```bash
# Buscar imágenes
docker search node

# Descargar
docker pull node:25-alpine

# Subir tu propia imagen
docker tag mi-app tu-usuario/mi-app:v1
docker push tu-usuario/mi-app:v1
```

Algunas imágenes oficiales importantes:
- `node` - Node.js
- `mysql` - MySQL
- `postgres` - PostgreSQL
- `nginx` - Servidor web
- `alpine` - Linux mínimo (~5MB)
- `ubuntu` - Ubuntu completo

---

## Cómo se usa una imagen de Docker Hub

```bash
# Sin especificar tag: usa "latest"
docker pull node

# Tag específico (recomendado)
docker pull node:25-alpine
```

Las etiquetas (tags) tienen este formato: `nombre:versión-sabor`

- `node:25` → Node 25, sabor completo
- `node:25-alpine` → Node 25, versión Alpine (más ligera)
- `node:25-slim` → Node 25, versión recortada

**Siempre usa un tag específico**. `latest` puede cambiar y romper tu app.

---

## Docker Compose

Docker Compose orquesta **múltiples contenedores** como un solo grupo.

```yaml
# docker-compose.yml
services:
  db:
    image: mysql:9.6
  app:
    build: .
    depends_on:
      - db
```

Un comando levanta todo: `docker compose up`

Sin compose tendrías que hacer:

```bash
docker network create mi-red
docker run -d --network mi-red --name db mysql:9.6
docker run -d --network mi-red -p 3000:3000 --name app mi-app
```

Compose simplifica todo eso en un archivo YAML.

---

## Dockerfile vs docker-compose.yml

| Dockerfile | docker-compose.yml |
|------------|-------------------|
| Construye **una** imagen | Orquesta **varios** contenedores |
| Define **cómo se crea** la imagen | Define **cómo se ejecutan** los servicios |
| FROM, RUN, COPY, CMD | services, ports, volumes, environment |

**No compiten, se complementan:**

```
Dockerfile → build → Imagen
docker-compose.yml → orquesta → Contenedores
```

Tu proyecto usa ambos: el Dockerfile para la app Node y el compose para conectar app + MySQL.

---

## Alternativas y complementos

| Herramienta | Para qué sirve |
|-------------|---------------|
| **Docker Compose** | Múltiples contenedores locales |
| **Docker Swarm** | Orquestación nativa de Docker |
| **Kubernetes** | Orquestación avanzada (producción) |
| **Portainer** | UI web para gestionar Docker |
| **Watchtower** | Actualizar contenedores automáticamente |

Para tu proyecto, con Docker Compose es suficiente.

---

## Flujo completo

```
Código fuente
    │
    ▼
Dockerfile ──build──► Imagen Docker
    │                      │
    │                      ▼
    │              docker-compose.yml
    │                      │
    └───────orquesta───────┘
                │
                ▼
         Contenedores corriendo
         (app + db + etc.)
```
