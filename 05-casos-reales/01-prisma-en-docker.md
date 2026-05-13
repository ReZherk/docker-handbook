# Prisma en Docker

Tu proyecto usa Prisma, lo que agrega complejidad al Dockerfile. Aquí explicamos cómo funciona cada pieza.

---

## ¿Qué hace Prisma?

Prisma es un ORM que:

1. **Genera un cliente** (`@prisma/client`) basado en tu esquema
2. **Ejecuta migraciones** para crear/modificar tablas en la DB
3. **Conecta tu app** a la base de datos

Docker complica esto porque:
- `prisma generate` debe ejecutarse **durante el build** (para que el cliente esté disponible)
- `prisma migrate` debe ejecutarse **al iniciar** el contenedor (cuando la DB ya existe)
- La DB no está disponible durante el build

---

## En tu Dockerfile: prisma generate

```dockerfile
RUN npx prisma generate && npm run build
```

### ¿Por qué durante el build?

`prisma generate` crea el Prisma Client (`@prisma/client`) en `node_modules`. Sin esto, tu app no encuentra el cliente al compilar.

```bash
# Lo que hace prisma generate:
npx prisma generate
# Crea: node_modules/@prisma/client/
#         ├── index.js
#         ├── runtime/
#         └── schema.prisma
```

### ¿Y en la etapa final?

```dockerfile
COPY --from=builder /app/src/generated ./src/generated
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/prisma.config.ts ./prisma.config.ts
```

Se copian:
- `src/generated/` → archivos generados por Prisma
- `prisma/` → esquema y migraciones
- `prisma.config.ts` → configuración de Prisma

---

## En tu docker-compose: prisma migrate deploy

```yaml
app:
  command: sh -c "npx prisma migrate deploy && node dist/main.js || tail -f /dev/null"
```

`prisma migrate deploy` ejecuta las migraciones **pendientes** en la DB.

Diferencia entre los comandos de migrate:

| Comando | Propósito | ¿Cuándo usarlo? |
|---------|-----------|----------------|
| `prisma migrate dev` | Desarrollo: crea y aplica migraciones | Local |
| `prisma migrate deploy` | Producción: aplica migraciones existentes | CI/CD, Docker |
| `prisma migrate reset` | Borra datos y re-aplica migraciones | Solo desarrollo |

---

## El problema del orden

Tu app necesita:
1. Que MySQL esté corriendo
2. Que las migraciones se hayan aplicado
3. Después arrancar

Solución en tu compose:
```yaml
depends_on:
  db:
    condition: service_healthy
    # Espera a que MySQL acepte conexiones
command: sh -c "npx prisma migrate deploy && node dist/main.js"
    # Aplica migraciones y luego arranca
```

---

## Esquema típico de Prisma

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
}
```

En Docker, `DATABASE_URL` se inyecta via `environment` en compose.

---

## Prisma Client en producción

```javascript
// Después de prisma generate, el cliente está disponible
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// Tu app usa prisma para consultas
const users = await prisma.user.findMany();
```

El `prisma generate` debe ejecutarse **antes** que `npm run build` para que TypeScript compile correctamente.

Por eso en tu Dockerfile:
```dockerfile
RUN npx prisma generate && npm run build
#    ↑ primero genera         ↑ después compila
```

---

## Buenas prácticas con Prisma en Docker

1. **No ejecutar `prisma migrate dev`** en Docker (es para desarrollo local)
2. **Usar `prisma migrate deploy`** en el entrypoint
3. **Incluir carpeta `prisma/`** en la imagen final (necesaria para migrate)
4. **`DATABASE_URL` solo en el entorno**, nunca en el código
5. **`prisma generate` durante el build**, no al iniciar

---

## Posibles errores y soluciones

| Error | Causa | Solución |
|-------|-------|----------|
| `@prisma/client did not initialize yet` | Falta `prisma generate` | Agregar al Dockerfile |
| `Can't reach database server` | DB no disponible | Healthcheck + depends_on |
| `Migration not found` | Migraciones no copiadas | Incluir `prisma/` en imagen final |
| `DATABASE_URL is not set` | Variable faltante | Verificar environment en compose |

---

## Resumen del flujo de Prisma en Docker

```
1. docker build
   ├── COPY . .                    → schema.prisma incluido
   ├── RUN npx prisma generate     → crea @prisma/client
   └── RUN npm run build           → compila TypeScript

2. docker compose up
   ├── db inicia (MySQL)
   ├── healthcheck → MySQL listo
   ├── app arranca
   ├── npx prisma migrate deploy  → aplica migraciones
   └── node dist/main.js          → app corriendo
```
