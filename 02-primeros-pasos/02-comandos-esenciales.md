# Comandos Esenciales de Docker

Esta es la guía de comandos que usarás el 90% del tiempo.

---

## Gestión de imágenes

```bash
# Descargar imagen
docker pull node:25-alpine

# Listar imágenes
docker images
docker image ls

# Ver información detallada
docker inspect node:25-alpine

# Eliminar imagen
docker rmi node:25-alpine

# Eliminar imágenes no usadas
docker image prune
```

---

## Gestión de contenedores

```bash
# Crear y ejecutar contenedor
docker run node:25-alpine

# Ejecutar en modo detached (background)
docker run -d node:25-alpine

# Ejecutar con nombre
docker run --name mi-app node:25-alpine

# Mapear puertos (-p host:contenedor)
docker run -p 3000:3000 mi-app

# Montar volumen (-v)
docker run -v mysql_data:/var/lib/mysql mysql

# Variables de entorno (-e)
docker run -e DATABASE_URL="mysql://root:root@db:3306/globtecx" mi-app

# Listar contenedores activos
docker ps

# Listar todos (activos + detenidos)
docker ps -a

# Detener contenedor
docker stop mi-app

# Iniciar contenedor detenido
docker start mi-app

# Ver logs
docker logs mi-app

# Seguir logs en tiempo real
docker logs -f mi-app

# Ejecutar comando dentro del contenedor
docker exec -it mi-app sh

# Eliminar contenedor
docker rm mi-app

# Eliminar contenedor aunque esté corriendo
docker rm -f mi-app

# Eliminar contenedores detenidos
docker container prune
```

---

## Build de imágenes

```bash
# Construir imagen desde Dockerfile
docker build -t mi-app .

# Build con tag específico
docker build -t mi-app:v1.0 .

# Sin cache (build limpio)
docker build --no-cache -t mi-app .

# Build con argumentos
docker build --build-arg NODE_ENV=production -t mi-app .
```

---

## Docker Compose

```bash
# Iniciar servicios
docker compose up

# En background
docker compose up -d

# Construir imágenes antes de iniciar
docker compose up --build

# Ver logs de todos los servicios
docker compose logs -f

# Ver logs de un servicio específico
docker compose logs -f app

# Ejecutar comando en un servicio
docker compose exec app sh

# Ejecutar en servicio de base de datos
docker compose exec db mysql -u root -p

# Detener servicios
docker compose down

# Detener y eliminar volúmenes (¡cuidado! borra datos)
docker compose down -v

# Ver estado
docker compose ps

# Ver recursos
docker compose top
```

---

## Flags importantes explicados

| Flag | Significado | Ejemplo |
|------|-------------|---------|
| `-d` | Detached (background) | `docker run -d mi-app` |
| `-it` | Interactivo + terminal | `docker run -it alpine sh` |
| `-p` | Puerto (host:contenedor) | `docker run -p 3000:3000 app` |
| `-v` | Volumen | `docker run -v data:/data app` |
| `-e` | Variable de entorno | `docker run -e KEY=value app` |
| `--rm` | Eliminar contenedor al detener | `docker run --rm alpine` |
| `--name` | Asignar nombre | `docker run --name mi-app app` |

---

## Comodines útiles

```bash
# Detener todos los contenedores
docker stop $(docker ps -q)

# Eliminar todos los contenedores
docker rm $(docker ps -aq)

# Eliminar todas las imágenes
docker rmi $(docker images -q)

# Limpieza general (imágenes, contenedores, volúmenes no usados)
docker system prune -a --volumes
```

⚠️ **Cuidado** con `docker system prune --volumes`: borra volúmenes no usados, incluyendo datos de base de datos.

---

## Resumen rápido

| Quiero... | Comando |
|-----------|---------|
| Correr mi app | `docker compose up -d` |
| Ver logs | `docker compose logs -f` |
| Entrar al contenedor | `docker compose exec app sh` |
| Reconstruir | `docker compose up --build` |
| Detener todo | `docker compose down` |
| Ver qué está corriendo | `docker ps` |
| Ver imágenes | `docker images` |
