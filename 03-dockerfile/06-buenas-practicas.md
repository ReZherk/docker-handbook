# Buenas Prácticas en Dockerfile

Tu Dockerfile ya implementa varias buenas prácticas. Aquí las identificamos y agregamos más.

---

## Lo que tu Dockerfile hace bien

### ✅ Imagen base Alpine

```dockerfile
FROM node:25-alpine
```

Alpine es ~5MB vs Ubuntu ~200MB. Menos superficie de ataque, builds más rápidos.

### ✅ Multi-stage build

```dockerfile
FROM node:25-alpine AS builder
# ... build ...
FROM node:25-alpine
# ... solo producción ...
```

Imagen final más pequeña y segura.

### ✅ Orden optimizado para cache

```dockerfile
COPY package*.json ./
RUN npm ci --include=dev
COPY . .
```

Las dependencias se cachean separadas del código.

### ✅ COPY selectivo

```dockerfile
COPY --from=builder /app/dist/src ./dist
COPY --from=builder /app/src/generated ./src/generated
```

No copia todo, solo lo necesario.

### ✅ npm ci en lugar de npm install

Garantiza builds determinísticos.

### ✅ Forma exec en CMD

```dockerfile
CMD ["node", "dist/main"]
```

Maneja señales correctamente.

---

## Prácticas que podrías agregar

### .dockerignore

Crea un archivo `.dockerignore` en la raíz del proyecto:

```
node_modules
dist
.git
.gitignore
.env
*.md
Dockerfile
docker-compose.yml
```

Sin esto, Docker envía todo el directorio al daemon de build. Si tienes `node_modules` local, serán cientos de MB innecesarios.

### Usar USER no root

Por defecto, los contenedores corren como **root**. Esto es inseguro.

```dockerfile
# Crear usuario no root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

USER appuser

CMD ["node", "dist/main"]
```

Si un atacante entra al contenedor, no tiene acceso root.

### Minimizar capas

Cada `RUN`, `COPY`, etc. crea una capa. Agrupa comandos relacionados:

```dockerfile
# MAL (muchas capas):
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*

# BIEN (una capa):
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### LABEL para metadata

```dockerfile
LABEL maintainer="tu-email@ejemplo.com"
LABEL version="1.0.0"
LABEL description="API de GloTecX"
```

Útil cuando tienes muchas imágenes.

### HEALTHCHECK

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
```

Docker puede reiniciar contenedores que fallen el healthcheck. En tu `docker-compose.yml` ya tienes healthcheck para MySQL.

---

## Anti-patrones (lo que NO debes hacer)

### ❌ Copiar todo y luego instalar

```dockerfile
# MAL
COPY . .
RUN npm ci     # Se ejecuta cada vez que cambia cualquier archivo
```

### ❌ Imagen base sin tag específico

```dockerfile
# MAL
FROM node

# BIEN
FROM node:25-alpine
```

`latest` puede cambiar y romper tu app.

### ❌ Guardar secrets en la imagen

```dockerfile
# MUY MAL
ENV JWT_SECRET=supersecreto
ENV DATABASE_URL=mysql://root:root@db:3306/globtecx
```

Los secrets quedan grabados en la imagen. Cualquiera que descargue la imagen puede verlos con `docker history`.

### ❌ Instalar herramientas innecesarias

```dockerfile
# MAL (en producción)
RUN apt-get install -y vim curl wget git build-essential
```

Cada herramienta extra es un riesgo de seguridad.

### ❌ Usar ADD en lugar de COPY

```dockerfile
# MAL
ADD . /app

# BIEN
COPY . /app
```

`ADD` tiene comportamientos mágicos (descomprimir archivos, descargar URLs). `COPY` es explícito. Usa `ADD` solo cuando necesites esas características.

---

## Checklist de Dockerfile

- [ ] Imagen base con tag específico (`node:25-alpine` ✅)
- [ ] Multi-stage si compilas código ✅
- [ ] Orden optimizado para cache ✅
- [ ] `.dockerignore` presente ❌ (agregarlo)
- [ ] `npm ci` en lugar de `npm install` ✅
- [ ] `COPY` en lugar de `ADD` ✅
- [ ] `CMD` en forma exec ✅
- [ ] `USER` no root ❌ (opcional pero recomendado)
- [ ] Sin secrets hardcodeados ✅
- [ ] `WORKDIR` definido ✅
