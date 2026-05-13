# Command y Entrypoint

El `command` en tu docker-compose.yml hace algo interesante:

```yaml
app:
  command: sh -c "npx prisma migrate deploy && node dist/main.js || tail -f /dev/null"
```

---

## ¿Qué hace este comando?

Desglosemoslo:

```bash
sh -c "npx prisma migrate deploy && node dist/main.js || tail -f /dev/null"
```

1. `sh -c "..."` → ejecuta el string como comando shell
2. `npx prisma migrate deploy` → ejecuta migraciones de Prisma
3. `&&` → si la migración funciona, ejecuta lo siguiente
4. `node dist/main.js` → arranca la app
5. `||` → si la app falla (o si la migración falló en el paso 2)
6. `tail -f /dev/null` → mantiene el contenedor vivo para debugging

---

## ¿Por qué `tail -f /dev/null`?

Sin esto, si la app crashea al iniciar (ej: no puede conectar a DB), el contenedor se detiene.

```bash
# Sin tail -f: contenedor se detiene al fallar
docker ps  # no aparece

# Con tail -f: contenedor sigue vivo
docker ps  # aparece como "Up"
docker compose exec app sh  # puedes entrar a investigar
```

Es un **truco de debugging**. En producción no deberías usarlo.

Para producción usarías:

```yaml
app:
  command: sh -c "npx prisma migrate deploy && node dist/main.js"
  restart: always  # Docker reinicia si falla
```

---

## CMD en Dockerfile vs command en compose

```dockerfile
# Dockerfile
CMD ["node", "dist/main"]
```

```yaml
# docker-compose.yml
command: sh -c "npx prisma migrate deploy && node dist/main.js || tail -f /dev/null"
```

**El `command` en compose SOBRESCRIBE el `CMD` del Dockerfile.**

Sin el `command` en compose, el contenedor ejecutaría `CMD ["node", "dist/main"]`. Con el `command`, ejecuta el sh -c.

---

## Entrypoint vs CMD vs command

| Instrucción | Se define en | Se sobrescribe con | Propósito |
|-------------|-------------|-------------------|-----------|
| `ENTRYPOINT` | Dockerfile | `--entrypoint` | Comando base (no cambia) |
| `CMD` | Dockerfile | `command:` en compose | Argumentos por defecto |
| `command` | docker-compose.yml | - | Sobrescribe CMD |

Relación:

```
ENTRYPOINT + CMD = comando final
CMD = argumentos por defecto para ENTRYPOINT
```

Ejemplo:

```dockerfile
ENTRYPOINT ["npm"]
CMD ["start"]
# → ejecuta: npm start
```

```yaml
# con compose
command: run build
# → ejecuta: npm run build
```

---

## Formas de escribir CMD/command

```dockerfile
# Forma exec (JSON array) → RECOMENDADA
CMD ["node", "dist/main"]

# Forma shell
CMD node dist/main

# Forma shell con sh -c explícito
CMD sh -c "npx prisma migrate deploy && node dist/main.js"
```

La **forma exec** es mejor porque:
- El proceso es PID 1
- Recibe señales del sistema (SIGTERM, SIGINT)
- Docker puede detenerlo limpiamente

La **forma shell**:
- Ejecuta `/bin/sh -c "comando"`
- El shell es PID 1, no tu app
- Tu app no recibe SIGTERM directamente

---

## ¿Qué deberías usar en producción?

### Opción 1: Healthcheck + restart

```yaml
app:
  command: sh -c "npx prisma migrate deploy && node dist/main.js"
  restart: always
  healthcheck:
    test: ["CMD", "wget", "--spider", "http://localhost:3000/health"]
    interval: 30s
    retries: 3
```

### Opción 2: Init script + entrypoint

```dockerfile
# Dockerfile
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["node", "dist/main"]
```

```bash
#!/bin/sh
# entrypoint.sh
set -e

# Ejecutar migraciones
npx prisma migrate deploy

# Ejecutar el comando principal
exec "$@"
```

La opción 2 es más profesional y se usa en proyectos grandes.

---

## En tu proyecto (debugging)

El `command` actual es ideal para **desarrollo**:

```bash
sh -c "npx prisma migrate deploy && node dist/main.js || tail -f /dev/null"
```

1. Intenta migrar y arrancar
2. Si falla, mantén vivo el contenedor
3. Entras con `docker compose exec app sh` y ves qué pasó
4. Revisas logs con `docker compose logs app`

Cuando pases a producción, cambia a:

```yaml
app:
  command: sh -c "npx prisma migrate deploy && node dist/main.js"
  restart: unless-stopped
```
