# Instrucciones Básicas del Dockerfile

Desglose de cada instrucción que aparece en tu Dockerfile.

---

## FROM — Imagen base

```dockerfile
FROM node:25-alpine AS builder
FROM node:25-alpine
```

`FROM` define el punto de partida. Tu imagen se construye **sobre** otra imagen existente.

Tu proyecto usa `node:25-alpine`:

- `node` → imagen oficial de Node.js
- `25` → versión de Node
- `alpine` → basada en Alpine Linux (~5MB)

¿Por qué Alpine?

```
Ubuntu: ~200MB (imagen base)
Alpine: ~5MB  (imagen base)
```

Alpine incluye solo lo esencial para correr apps. Esto hace tus imágenes más ligeras y seguras (menos superficie de ataque).

```dockerfile
# También podrías usar:
FROM node:25          # ~1GB (completa, muchas herramientas)
FROM node:25-slim     # ~200MB (recortada)
FROM node:25-alpine   # ~120MB (la más ligera con Node)
```

---

## WORKDIR — Directorio de trabajo

```dockerfile
WORKDIR /app
```

Define dónde estás "parado" dentro del contenedor.

- Si el directorio no existe, lo crea
- Todas las instrucciones siguientes (`COPY`, `RUN`, `CMD`) parten de aquí
- Es como hacer `mkdir -p /app && cd /app`

Sin `WORKDIR`:

```dockerfile
RUN mkdir -p /app
COPY ./app /app
RUN cd /app && npm ci
```

Con `WORKDIR`:

```dockerfile
WORKDIR /app
COPY package*.json ./
RUN npm ci
```

**Más limpio y profesional.**

---

## COPY — Copiar archivos

```dockerfile
COPY package*.json ./
COPY . .
COPY --from=builder /app/dist ./dist
```

Copia archivos desde tu PC (o desde otra etapa) al contenedor.

```
COPY <origen> <destino>
```

- `./` (con WORKDIR /app) → equivale a `/app/`
- `package*.json` → patrón glob (coincide con `package.json` y `package-lock.json`)
- `--from=builder` → copia desde otra etapa del build (multi-stage)

---

## RUN — Ejecutar comandos

```dockerfile
RUN npm ci --include=dev
RUN npx prisma generate && npm run build
RUN npm ci --only=production
```

Ejecuta comandos **durante la construcción** de la imagen. Todo lo que instales o generes aquí queda guardado en la imagen.

Se puede escribir de dos formas:

```dockerfile
# Forma shell (como en terminal)
RUN npm ci

# Forma exec (recomendada para scripts complejos)
RUN ["npm", "ci"]
```

---

## EXPOSE — Documentar puertos

```dockerfile
EXPOSE 3000
```

**Solo documentación.** No abre el puerto realmente. Le dice a quien lea el Dockerfile: "esta app usa el puerto 3000".

Para exponer el puerto de verdad necesitas usar `-p` o `ports:` en compose:

```bash
docker run -p 3000:3000 mi-app
```

Sin `-p` el contenedor corre pero no es accesible desde afuera.

---

## CMD — Comando por defecto

```dockerfile
CMD ["node", "dist/main"]
```

Define **qué se ejecuta cuando el contenedor arranca**.

Formas de escribirlo:

```dockerfile
# Forma exec (JSON array) → RECOMENDADA
CMD ["node", "dist/main"]

# Forma shell
CMD node dist/main
```

La diferencia:

- **Exec**: el proceso es PID 1, responde a señales (`SIGTERM`)
- **Shell**: se envuelve en `/bin/sh -c`, no responde directamente a señales

Usa siempre la forma exec (`["comando", "argumento"]`).

---

## Diferencia entre RUN y CMD

| RUN                                     | CMD                                     |
| --------------------------------------- | --------------------------------------- |
| Se ejecuta al **construir** la imagen   | Se ejecuta al **iniciar** el contenedor |
| Modifica la imagen                      | No modifica la imagen                   |
| `RUN npm ci`                            | `CMD ["node", "app.js"]`                |
| Se usa para instalar, compilar, generar | Se usa para arrancar la app             |

En tu proyecto:

```dockerfile
RUN npm ci              # Se ejecuta cuando haces docker build
CMD ["node", "dist/main"]  # Se ejecuta cuando haces docker run
```

---

## Resumen visual de tu Dockerfile

```dockerfile
FROM node:25-alpine AS builder  # Imagen base + nombre de etapa
WORKDIR /app                     # Nos paramos en /app
COPY package*.json ./            # Copiamos solo package.json (cache)
RUN npm ci --include=dev         # Instalamos TODO (dev + prod)
COPY . .                         # Copiamos el resto del código
RUN npx prisma generate && npm run build  # Generamos Prisma + compilamos

# ──── Segunda etapa (imagen final) ────

FROM node:25-alpine              # Nueva imagen base limpia
WORKDIR /app                     # Nos paramos en /app
COPY --from=builder ...          # Copiamos SOLO lo necesario
RUN npm ci --only=production     # Instalamos SOLO producción
EXPOSE 3000                      # Documentamos el puerto
CMD ["node", "dist/main"]        # Arrancamos la app
```
