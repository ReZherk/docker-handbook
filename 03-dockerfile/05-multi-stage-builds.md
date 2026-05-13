# Multi-Stage Builds

Esta es probablemente la técnica más importante de tu Dockerfile.

---

## El problema

Una imagen de Node.js con TypeScript necesita:

1. TypeScript (para compilar)
2. Prisma CLI (para generar cliente)
3. Dependencias de desarrollo (testing, linting)
4. El código fuente completo

Pero en **producción** solo necesitas:
1. El JavaScript compilado
2. Las dependencias de producción
3. Prisma client ya generado

Si usas una sola etapa, tu imagen de producción lleva **todo lo innecesario**.

---

## La solución: dos etapas

```dockerfile
# ──── ETAPA 1: BUILDER (todo incluido) ────
FROM node:25-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --include=dev

COPY . .
RUN npx prisma generate && npm run build

# ──── ETAPA 2: IMAGEN FINAL (solo lo necesario) ────
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

---

## ¿Qué logramos?

```
ETAPA 1 (builder)
  Contiene:
  ✓ TypeScript
  ✓ Prisma CLI
  ✓ Dev dependencies
  ✓ Código fuente completo
  Tamaño: ~1.2GB

ETAPA 2 (final)
  Contiene:
  ✓ JavaScript compilado
  ✓ Prisma client
  ✓ Dependencies de producción
  ✗ TypeScript (no necesario)
  ✗ Prisma CLI (no necesario)
  ✗ Código fuente (no necesario)
  Tamaño: ~180MB
```

**Ahorro: ~85% de tamaño**

---

## Cómo funciona `COPY --from`

La sintaxis especial `--from=builder` copia archivos **desde otra imagen/etapa**:

```dockerfile
# Desde otra etapa del mismo Dockerfile
COPY --from=builder /app/dist ./dist

# Desde una imagen externa (¡sí, se puede!)
COPY --from=nginx:alpine /etc/nginx/nginx.conf ./nginx.conf
```

Esto permite construir artefactos en una etapa grande y copiar solo lo necesario a la etapa final.

---

## ¿Qué más podrías pasar a la etapa final?

Cosas que **no** necesitas en producción:

| Archivo/Directorio | ¿Necesario en producción? | Razón |
|-------------------|--------------------------|-------|
| `node_modules` completos | Solo producción | `npm ci --only=production` |
| `src/` (TypeScript) | No | Ya está compilado a `dist/` |
| `node_modules/.cache` | No | Cache de npm/prisma |
| `tests/` | No | Solo testing |
| `.git/` | No | Control de versiones |

---

## Alternativa: una sola etapa

```dockerfile
FROM node:25-alpine
WORKDIR /app
COPY . .
RUN npm ci && npx prisma generate && npm run build
EXPOSE 3000
CMD ["node", "dist/main"]
```

Funciona, pero la imagen final es **~1.2GB** vs **~180MB**.

Multi-stage es siempre la mejor práctica para apps compiladas.

---

## ¿Cuándo NO necesitas multi-stage?

- Apps en Python/Ruby/JavaScript **puro** (sin compilación)
- Scripts simples
- Prototipos rápidos

Pero para tu app con TypeScript + Prisma, **es obligatorio**.

---

## Ventajas adicionales

1. **Seguridad**: menos herramientas en la imagen final = menos vulnerabilidades
2. **Velocidad de deploy**: imágenes más pequeñas suben más rápido
3. **Almacenamiento**: menos espacio en disco
4. **Aislamiento**: la etapa builder puede tener herramientas que no quieres en producción

---

## Tu proyecto específico

```dockerfile
# Builder: instala TypeScript, Prisma CLI, compila
RUN npx prisma generate && npm run build
#   ↑ Prisma CLI está en devDependencies

# Final: solo runtime
RUN npm ci --only=production
#   ↑ No incluye TypeScript ni Prisma CLI
COPY --from=builder /app/dist/src ./dist         # Código compilado
COPY --from=builder /app/src/generated ./src/generated  # Prisma client
COPY --from=builder /app/prisma ./prisma         # Schema de Prisma
COPY --from=builder /app/prisma.config.ts ./prisma.config.ts  # Config
```

Cada `COPY --from=builder` es una **selección quirúrgica** de lo que necesitas.
