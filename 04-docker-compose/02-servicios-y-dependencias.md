# Servicios y Dependencias

En Docker Compose, cada **servicio** es un contenedor. Los servicios se definen bajo `services:` y pueden depender unos de otros.

---

## Definición de servicios

```yaml
services:
  db:        # ← nombre del servicio (dentro de la red)
    image: mysql:9.6
    # ...

  app:       # ← otro servicio
    build: .
    # ...
```

Cada servicio puede usar:
- `image`: una imagen existente (MySQL, Redis, etc.)
- `build`: construir desde un Dockerfile local

En tu proyecto:
- `db` usa **image** (ya existe en Docker Hub)
- `app` usa **build** (se construye desde tu Dockerfile local)

---

## depends_on — ¿Quién arranca primero?

```yaml
app:
  depends_on:
    db:
      condition: service_healthy
```

`depends_on` controla el **orden de arranque**. Sin esto, `app` podría iniciar antes que `db` y fallar.

### Niveles de depends_on

```yaml
# 1. Básico (solo orden, no espera que esté listo)
depends_on:
  - db

# 2. Espera a que el servicio esté "iniciado"
depends_on:
  db:
    condition: service_started

# 3. Espera a que pase el healthcheck (el que usas)
depends_on:
  db:
    condition: service_healthy
```

**Tu proyecto usa el nivel 3**, el más estricto y correcto.

---

## ¿Qué pasa sin depends_on?

```yaml
app:
  build: .
  # Sin depends_on
```

```
1. Docker inicia app y db simultáneamente
2. app intenta conectar a db
3. db todavía no está listo
4. app CRASHEA
```

Por eso tu docker-compose tiene:
1. `healthcheck` en MySQL
2. `condition: service_healthy` en app
3. `command: sh -c "npx prisma migrate deploy && node dist/main.js"` que reintenta

---

## build vs image

```yaml
# Usar una imagen existente (no necesita Dockerfile)
app:
  image: node:25-alpine   # imagen pre-construida

# Construir desde Dockerfile (necesita Dockerfile)
app:
  build: .                # busca Dockerfile en el directorio actual
  # o
  build:
    context: .            # directorio del build
    dockerfile: Dockerfile  # nombre del Dockerfile
```

Tu proyecto usa ambos:
- `db` → `image: mysql:9.6` (no necesitas construir MySQL)
- `app` → `build: .` (tu Dockerfile personalizado)

---

## Múltiples servicios, una red

Cuando usas compose, todos los servicios están en la **misma red** automáticamente.

Esto significa:
- `app` puede conectarse a `db` usando el nombre del servicio como hostname
- La URL de conexión: `mysql://root:root@db:3306/globtecx`
- No necesitas IPs ni configurar redes manualmente

---

## Escalar servicios

```bash
# Ejecutar 3 instancias de app (balanceo de carga)
docker compose up -d --scale app=3
```

Esto funciona porque `app` no expone puertos fijos. Si expusieras `3000:3000`, no podrías tener 2 instancias en el mismo puerto.

---

## Resumen útil

| Directiva | En tu proyecto | Propósito |
|-----------|---------------|-----------|
| `image` | `mysql:9.6` | Usar imagen existente |
| `build` | `.` | Construir desde Dockerfile |
| `depends_on` | `condition: service_healthy` | Esperar a MySQL |
| `ports` | `"3000:3000"` | Exponer app al host |
| `environment` | `DATABASE_URL: ...` | Configurar servicios |
| `volumes` | `mysql_data:/var/lib/mysql` | Persistir datos DB |
| `command` | `sh -c "..."` | Personalizar inicio |
