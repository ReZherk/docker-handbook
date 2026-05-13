# Healthchecks en Docker

Un **healthcheck** es una prueba que Docker ejecuta periódicamente para saber si un contenedor está "saludable" o no.

---

## El problema

Cuando tu app Node.js intenta conectar a MySQL:

```
app (Node.js) ── ¿está listo? ──► db (MySQL)
```

Si `app` arranca antes que `db`, falla la conexión.

`depends_on` solo espera a que el contenedor **esté creado**, no a que el servicio **esté listo** para recibir conexiones.

---

## Healthcheck en tu proyecto

```yaml
db:
  image: mysql:9.6
  healthcheck:
    test: ["CMD-SHELL", "mysqladmin ping -h localhost -u root -proot"]
    interval: 10s
    timeout: 5s
    retries: 5
```

Esto ejecuta `mysqladmin ping` cada 10 segundos. Si falla 5 veces seguidas, el contenedor se marca como **unhealthy**.

---

## Cómo se usa con depends_on

```yaml
app:
  depends_on:
    db:
      condition: service_healthy
```

Esto le dice a Docker:
1. Espera a que `db` esté "healthy" (el healthcheck pase)
2. Luego arranca `app`

Sin `condition: service_healthy`, `app` arrancaría inmediatamente después de que `db` se cree, sin esperar que MySQL esté listo.

---

## Sintaxis del healthcheck

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
  interval: 30s      # cada cuánto se ejecuta
  timeout: 10s       # tiempo máximo para completar
  retries: 3         # fallos consecutivos para considerar unhealthy
  start_period: 40s  # tiempo de gracia inicial (no cuenta como fallo)
```

También se puede definir en el Dockerfile:

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

---

## Estados de salud

```
starting    → El contenedor se está iniciando
healthy     → El healthcheck pasó
unhealthy   → El healthcheck falló (n veces)
none        → No hay healthcheck definido
```

```bash
# Ver estado de salud
docker ps

# Filtrar por salud
docker ps --filter "health=healthy"
docker ps --filter "health=unhealthy"
```

---

## ¿Qué pasa si un contenedor está unhealthy?

- Docker **no** lo reinicia automáticamente
- Pero si usas `restart: always`, Docker reinicia contenedores unhealthy

```yaml
db:
  restart: always
  healthcheck:
    # ...
```

---

## Healthcheck para tu app Node.js

Podrías agregar un healthcheck para la app:

```yaml
app:
  healthcheck:
    test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
    interval: 30s
    timeout: 5s
    retries: 3
    start_period: 30s
```

O en el código, crear un endpoint `/health`:

```javascript
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});
```

---

## Buenas prácticas

1. **Siempre definir healthcheck** para servicios críticos (DB, API, caché)
2. **Usar `start_period`** para dar tiempo a que el servicio inicie
3. **No hacer healthchecks costosos** (consultas pesadas a DB)
4. **Combinar con `depends_on: condition: service_healthy`** para orden de arranque

---

## En tu proyecto

Flujo de arranque con healthchecks:

```
1. docker compose up
2. db (MySQL) se inicia
3. Docker ejecuta: mysqladmin ping -h localhost
4. MySQL no responde aún → status: starting
5. MySQL termina de iniciar → ping exitoso → status: healthy
6. app detecta que db está healthy → se inicia
7. app ejecuta: npx prisma migrate deploy
8. app arranca: node dist/main.js
9. ¡Todo funcionando!
```
