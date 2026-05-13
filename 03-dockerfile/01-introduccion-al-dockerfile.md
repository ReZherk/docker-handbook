# Introducción al Dockerfile

Un **Dockerfile** es un archivo de texto con instrucciones para construir una imagen Docker. Es como una **receta de cocina**: le dices a Docker qué ingredientes usar y en qué orden.

---

## Tu Dockerfile

Este es el Dockerfile de tu proyecto:

```dockerfile
FROM node:25-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --include=dev

COPY . .
RUN npx prisma generate && npm run build

FROM node:25-alpine

WORKDIR /app

COPY --from=builder /app/package*.json ./
RUN npm ci --only=production

COPY --from=builder /app/dist/src ./dist
COPY --from=builder /app/src/generated ./src/generated
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/prisma.config.ts ./prisma.config.ts

EXPOSE 3000

CMD ["node", "dist/main"]
```

En los próximos capítulos desglosaremos **cada línea** de este archivo.

---

## Estructura básica de un Dockerfile

```dockerfile
# 1. Imagen base (FROM)
FROM node:25-alpine

# 2. Directorio de trabajo
WORKDIR /app

# 3. Copiar archivos
COPY package*.json ./

# 4. Ejecutar comandos
RUN npm ci

# 5. Exponer puertos (documentación)
EXPOSE 3000

# 6. Comando por defecto
CMD ["node", "dist/main"]
```

---

## Sintaxis general

```
INSTRUCCIÓN argumentos
```

- Las instrucciones van en MAYÚSCULAS (convención)
- Cada instrucción crea una **capa** en la imagen
- El orden importa (afecta el cacheo)
- `#` se usa para comentarios

---

## Las instrucciones que usa tu proyecto

| Instrucción | Tu proyecto | Para qué sirve |
|-------------|-------------|---------------|
| `FROM` | `FROM node:25-alpine AS builder` | Imagen base |
| `WORKDIR` | `WORKDIR /app` | Directorio de trabajo |
| `COPY` | `COPY package*.json ./` | Copiar archivos |
| `RUN` | `RUN npm ci --include=dev` | Ejecutar comandos |
| `EXPOSE` | `EXPOSE 3000` | Documentar puerto |
| `CMD` | `CMD ["node", "dist/main"]` | Comando al iniciar |

---

## Cómo se construye la imagen

```bash
# Desde la raíz del proyecto (donde está el Dockerfile)
docker build -t mi-app .
```

El `.` es el **contexto de build** (todo lo que Docker puede copiar).

```bash
# Ver las capas de la imagen
docker history mi-app
```

Cada línea del Dockerfile aparece como una capa.

---

## Regla de oro

Un Dockerfile debe ser:
- **Reproducible**: misma imagen siempre
- **Ligero**: pocas capas, imagen base pequeña
- **Seguro**: no exponer secrets, no correr como root

Tu Dockerfile cumple bien estas reglas. Veamos por qué en los próximos capítulos.
