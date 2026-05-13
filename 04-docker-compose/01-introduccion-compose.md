# Introducción a Docker Compose

Docker Compose permite definir y ejecutar **múltiples contenedores** como un solo grupo.

---

## El problema que resuelve

Sin Docker Compose, para levantar tu proyecto necesitarías:

```bash
# 1. Crear una red
docker network create mi-red

# 2. Iniciar MySQL
docker run -d \
  --name db \
  --network mi-red \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=globtecx \
  -v mysql_data:/var/lib/mysql \
  mysql:9.6

# 3. Construir la app
docker build -t mi-app .

# 4. Iniciar la app
docker run -d \
  --name app \
  --network mi-red \
  -p 3000:3000 \
  -e DATABASE_URL="mysql://root:root@db:3306/globtecx" \
  mi-app
```

Con Compose, todo eso se resume a:

```bash
docker compose up -d
```

---

## Tu docker-compose.yml

```yaml
services:
  db:
    image: mysql:9.6
    ports:
      - "3307:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: globtecx
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h localhost -u root -proot"]
      interval: 10s
      timeout: 5s
      retries: 5

  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: "mysql://root:root@db:3306/globtecx"
      DATABASE_HOST: db
      JWT_SECRET: ${JWT_SECRET}
      # ... más variables
    depends_on:
      db:
        condition: service_healthy
    command: sh -c "npx prisma migrate deploy && node dist/main.js || tail -f /dev/null"

volumes:
  mysql_data:
```

---

## Estructura de un docker-compose.yml

```yaml
# Versión del formato (opcional en versiones modernas)
version: "3.8"

# Servicios (contenedores)
services:
  nombre-del-servicio:
    image: ...      # Imagen a usar
    build: .        # O construir desde Dockerfile
    ports: [...]    # Puertos
    environment:    # Variables de entorno
    volumes: [...]  # Volúmenes
    depends_on:     # Dependencias entre servicios

# Volúmenes persistentes
volumes:
  nombre-del-volumen:

# Redes (opcional, compose crea una por defecto)
networks:
  nombre-de-red:
```

---

## Servicios en tu proyecto

Tus servicios:

| Servicio | Función | Depende de |
|----------|---------|------------|
| **db** | Base de datos MySQL | Nada |
| **app** | API de Node.js | db (espera healthcheck) |

---

## Comandos básicos de compose

```bash
# Iniciar servicios
docker compose up

# En background
docker compose up -d

# Reconstruir imágenes y iniciar
docker compose up --build

# Detener servicios
docker compose down

# Ver logs de todos
docker compose logs -f

# Ver logs de un servicio
docker compose logs -f app

# Ejecutar comando en servicio
docker compose exec app sh

# Entrar a MySQL
docker compose exec db mysql -u root -p

# Ver estado
docker compose ps
```

---

## Ventajas de Docker Compose

1. **Declarativo**: todo en un archivo YAML
2. **Reproducible**: mismo resultado en cualquier máquina
3. **Red automática**: los servicios se ven por su nombre
4. **Volúmenes nombrados**: datos persistentes
5. **Variables de entorno**: separas config del código

---

## Tu proyecto necesita compose

Porque tienes **múltiples servicios** que deben comunicarse:

```
app (Node.js) ──► db (MySQL)
    │                  │
    │  localhost:3000   │  localhost:3307 (solo dev)
    │                  │
    └── red interna de compose ──┘
```

Dentro de la red de compose, `app` ve a `db` por el nombre del servicio: `mysql://root:root@db:3306/globtecx`.
