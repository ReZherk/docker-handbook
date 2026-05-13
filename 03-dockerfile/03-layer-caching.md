# Layer Caching

El **layer caching** es la razón por la que tu Dockerfile tiene este orden:

```dockerfile
COPY package*.json ./
RUN npm ci --include=dev

COPY . .
RUN npx prisma generate && npm run build
```

No es casualidad. Hay una estrategia detrás.

---

## ¿Qué es una capa (layer)?

Cada instrucción en el Dockerfile crea una **capa** en la imagen final.

```
Dockerfile                           Capas
─────────────────────────────────────────────
FROM node:25-alpine          →  capa 1 (base)
WORKDIR /app                  →  capa 2
COPY package*.json ./         →  capa 3
RUN npm ci --include=dev      →  capa 4
COPY . .                      →  capa 5
RUN npx prisma generate &&... →  capa 6
```

Cada capa es una **diferencia** respecto a la anterior. Docker reutiliza capas que no han cambiado.

---

## El problema que resuelve

Sin caching, cada vez que haces `docker build` tendrías que:

1. Descargar Node.js (otra vez)
2. Instalar todas las dependencias (otra vez)
3. Compilar todo (otra vez)

Con caching, si no cambiaste `package.json`, Docker **reusa** la capa de `npm ci`.

---

## Cómo funciona

Docker compara cada instrucción con su cache:

```dockerfile
FROM node:25-alpine     → ¿Cambió la imagen base? No → Usa cache
WORKDIR /app             → Siempre igual → Usa cache
COPY package*.json ./    → ¿Cambió package.json?     ─┐
                         →   NO → Usa cache           │
RUN npm ci               → Usa cache (porque la capa  │
                         → anterior no cambió)        │
                         →                            │
COPY . .                 → ¿Cambió algún archivo?  ←──┘
                         →   SÍ → Cache inválido
RUN npx prisma generate → Se ejecuta de nuevo
```

**Si cambias código fuente pero no dependencias, `npm ci` se reutiliza.**

---

## En tu proyecto: optimización

Tu Dockerfile está optimizado:

❌ **Mal ordenado** (sin caching):
```dockerfile
COPY . .                  # Todo el código
RUN npm ci                # npm ci se ejecuta SIEMPRE
RUN npx prisma generate   # Aunque solo cambió una línea de código
```

✅ **Bien ordenado** (con caching):
```dockerfile
COPY package*.json ./     # Solo package.json
RUN npm ci --include=dev  # Solo si cambian las dependencias
COPY . .                  # Resto del código
RUN npx prisma generate   # Solo si cambió el código
```

Si cambias solo una línea de código:
- Sin optimizar: **2 minutos** de build (npm ci completa)
- Optimizado: **5 segundos** (solo los últimos pasos)

---

## Reglas del cacheo

1. **Si una capa cambia, todas las siguientes se reconstruyen**
2. **Si una capa se reutiliza, las siguientes se reutilizan si no cambian**
3. `COPY` detecta cambios por el checksum del archivo
4. `RUN` se reutiliza si la capa anterior es igual

---

## ¿Cuándo se invalida el cache?

```bash
# Forzar rebuild sin cache
docker build --no-cache -t mi-app .

# O cuando cambia la imagen base
FROM node:25-alpine  →  FROM node:26-alpine  # cache inválido
```

---

## Consejos prácticos

1. **Copia `package.json` antes que el código** (lo más importante)
2. **Agrupa `RUN` que no cambian frecuentemente**
3. **Separa `RUN` que cambian frecuentemente**
4. **`COPY . .` debe ir lo más tarde posible** (el código cambia todo el tiempo)
5. **Usa `.dockerignore`** para no copiar basura al contenedor

---

## .dockerignore

Así como `.gitignore` ignora archivos en git, `.dockerignore` ignora archivos en el build:

```
node_modules
dist
.git
.env
*.md
```

Sin `.dockerignore`, Docker envía todo al daemon de build, incluyendo `node_modules` que pesa cientos de MB.

```bash
# Ver el contexto de build
docker build -t test -f- . <<< "FROM alpine"
# Docker envía TODO el directorio actual al daemon
```

---

## Ver las capas

```bash
# Ver capas de una imagen
docker history mi-app

# Ver tamaño de cada capa
docker history --no-trunc mi-app
```

Salida:
```
IMAGE          CREATED         SIZE
abc123def456   2 minutes ago   0B        CMD ["node" "dist/main"]
def456abc789   2 minutes ago   180MB     RUN npx prisma generate && npm run build
789abc123def   2 minutes ago   2KB       COPY . .  (el código pesa 2KB)
456def789abc   2 minutes ago   250MB     RUN npm ci --include=dev
123abc456def   2 minutes ago   1KB       COPY package*.json ./
```

Nota: la capa de `npm ci` pesa ~250MB, mientras que tu código solo unos KB. Por eso es tan importante cachearla.
