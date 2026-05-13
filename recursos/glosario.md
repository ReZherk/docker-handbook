# Dockerfile

# Instrucciones para construir una imagen Docker. Cada instrucción crea una "capa".

**FROM** — Define la imagen base sobre la que se construye la nueva imagen.
- Ej: `FROM node:25-alpine`

**WORKDIR** — Establece el directorio de trabajo dentro del contenedor.
- Ej: `WORKDIR /app`

**COPY** — Copia archivos del host al contenedor.
- Ej: `COPY package*.json ./`

**RUN** — Ejecuta comandos durante la construcción de la imagen.
- Ej: `RUN npm ci`

**EXPOSE** — Documenta el puerto que la app usará (solo documentación).
- Ej: `EXPOSE 3000`

**CMD** — Comando por defecto que se ejecuta al iniciar el contenedor.
- Ej: `CMD ["node", "dist/main"]`

**ENTRYPOINT** — Similar a CMD pero no se sobrescribe fácilmente.
- Ej: `ENTRYPOINT ["npm"]`

**LABEL** — Agrega metadata a la imagen.
- Ej: `LABEL version="1.0"`

**USER** — Cambia el usuario que ejecuta las instrucciones siguientes.
- Ej: `USER appuser`

**HEALTHCHECK** — Define un test para verificar que el contenedor funciona.
- Ej: `HEALTHCHECK CMD curl -f http://localhost || exit 1`

**ARG** — Variable disponible solo durante el build.
- Ej: `ARG NODE_ENV=production`

**ENV** — Variable de entorno disponible en el contenedor en ejecución.
- Ej: `ENV NODE_ENV=production`

---

# Docker Compose

**services** — Define los contenedores que se van a ejecutar.
- Ej: `services: db: ... app: ...`

**image** — Imagen a usar para el servicio.
- Ej: `image: mysql:9.6`

**build** — Directorio del Dockerfile para construir la imagen.
- Ej: `build: .`

**ports** — Mapeo de puertos (host:contenedor).
- Ej: `ports: - "3000:3000"`

**volumes** — Montaje de volúmenes para persistencia de datos.
- Ej: `volumes: - mysql_data:/var/lib/mysql`

**environment** — Variables de entorno para el servicio.
- Ej: `environment: - NODE_ENV=production`

**depends_on** — Define dependencias entre servicios.
- Ej: `depends_on: - db`

**command** — Sobrescribe el CMD del Dockerfile.
- Ej: `command: sh -c "npm start"`

**healthcheck** — Define prueba de salud para el servicio.
- Ej: `healthcheck: test: ["CMD", "mysqladmin", "ping"]`

**restart** — Política de reinicio del contenedor.
- Valores: `no`, `always`, `on-failure`, `unless-stopped`

---

# Comandos Docker (CLI)

**docker pull** — Descarga una imagen del registro.
- Ej: `docker pull node:25-alpine`

**docker build** — Construye una imagen desde un Dockerfile.
- Ej: `docker build -t mi-app .`

**docker run** — Crea y ejecuta un contenedor.
- Ej: `docker run -d -p 3000:3000 mi-app`

**docker ps** — Lista contenedores en ejecución.
- Ej: `docker ps -a` (incluye detenidos)

**docker stop** — Detiene un contenedor.
- Ej: `docker stop mi-app`

**docker rm** — Elimina un contenedor.
- Ej: `docker rm mi-app`

**docker rmi** — Elimina una imagen.
- Ej: `docker rmi node:25-alpine`

**docker exec** — Ejecuta un comando dentro de un contenedor activo.
- Ej: `docker exec -it mi-app sh`

**docker logs** — Muestra los logs de un contenedor.
- Ej: `docker logs -f mi-app`

**docker compose up** — Inicia todos los servicios definidos en compose.
- Ej: `docker compose up -d`

**docker compose down** — Detiene y elimina todos los servicios.
- Ej: `docker compose down -v` (también elimina volúmenes)

**docker compose logs** — Muestra logs de todos los servicios.
- Ej: `docker compose logs -f app`

**docker compose exec** — Ejecuta comando en un servicio.
- Ej: `docker compose exec app sh`

---

# Conceptos

**Imagen** — Plantilla read-only para crear contenedores. Se construye con un Dockerfile.

**Contenedor** — Instancia en ejecución de una imagen. Es efímero (los cambios se pierden al borrarlo, a menos que uses volúmenes).

**Volumen** — Mecanismo para persistir datos fuera del contenedor. Sobrevive a la eliminación del contenedor.

**Capa (layer)** — Cada instrucción del Dockerfile crea una capa. Las capas se cachean para builds más rápidos.

**Multi-stage build** — Técnica que usa múltiples etapas (FROM) para separar el entorno de compilación del de ejecución. Reduce el tamaño de la imagen final.

**Registry** — Repositorio de imágenes Docker. El público por defecto es Docker Hub.

**Alpine** — Distribución Linux mínima (~5MB). Muy usada como imagen base por su tamaño reducido.

**Healthcheck** — Prueba periódica que Docker ejecuta para determinar si un contenedor está funcionando correctamente.

**Named volume** — Volumen con nombre gestionado por Docker. Ej: `mysql_data:/var/lib/mysql`

**Bind mount** — Montaje de un directorio del host en el contenedor. Ej: `./src:/app/src`

---

# Flags comunes

**-d** — Detached mode (ejecuta en segundo plano).

**-it** — Interactivo + terminal (para ejecutar shells).

**-p** — Publicar puerto (host:contenedor).

**-v** — Montar volumen.

**-e** — Variable de entorno.

**--rm** — Eliminar contenedor al detenerse.

**--name** — Asignar nombre al contenedor.

**-f** — Follow (seguir logs en tiempo real).
