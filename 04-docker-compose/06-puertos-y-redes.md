# Puertos y Redes en Docker

Cómo se comunican los contenedores entre sí y con el mundo exterior.

---

## Puertos en tu proyecto

```yaml
db:
  ports:
    - "3307:3306"
    #  ↑ host   ↑ contenedor

app:
  ports:
    - "3000:3000"
```

El formato es: `puerto_host:puerto_contenedor`

- `3307:3306` → Accedes a MySQL desde tu PC en `localhost:3307`
- `3000:3000` → Accedes a la app en `localhost:3000`

---

## ¿Por qué 3307 y no 3306?

Tu PC quizá ya tiene MySQL instalado en el puerto 3306. Usando `3307:3306` evitas conflicto.

```bash
# Acceder a MySQL del contenedor desde tu PC
mysql -h localhost -P 3307 -u root -p
```

Dentro de la red de Docker, los servicios se ven por **nombre** y **puerto interno** (3306):

```
app ──► db:3306  (no localhost:3307)
```

---

## Comentario de producción en tu proyecto

```yaml
# EN PRODUCCIÓN: Eliminar el mapeo de puertos (ports) para evitar acceso externo a la DB
ports:
  - "3307:3306"
```

**¿Por qué?**

- En desarrollo: accedes a MySQL con DBeaver, TablePlus, etc.
- En producción: nadie debe acceder a la DB directamente
- Solo la app debe conectarse a la DB, y lo hace por la **red interna**

En producción, tu docker-compose debería ser:

```yaml
db:
  # ports: ← eliminado
  # La app sigue conectándose via red interna
```

La app sigue funcionando porque `db:3306` está en la misma red de Docker.

---

## Redes en Docker Compose

Por defecto, Docker Compose crea una **red** para todos los servicios.

```bash
# Ver redes
docker network ls

# Inspeccionar la red de tu proyecto
docker network inspect docker-handbook_default
```

Dentro de esta red:
- Cada contenedor tiene su propio IP
- Los servicios se resuelven por **nombre de servicio**
- `app` se conecta a `db` usando el hostname `db`

```
Red: docker-handbook_default
├── db  → 172.18.0.2
└── app → 172.18.0.3

app puede hacer ping a "db" y obtener 172.18.0.2
```

---

## Cómo se conecta tu app a MySQL

```yaml
# docker-compose.yml
environment:
  DATABASE_URL: "mysql://root:root@db:3306/globtecx"
  #                          ↑
  #                    nombre del servicio
  DATABASE_HOST: db
  DATABASE_PORT: 3306  # puerto interno del contenedor, no el mapeado
```

La URL usa `db` como hostname. Docker resuelve `db` a la IP del contenedor MySQL.

---

## Tipos de redes

| Tipo | Comportamiento |
|------|---------------|
| **bridge** (default) | Red privada, contenedores se ven entre sí |
| **host** | Comparte la red del host (sin aislamiento) |
| **none** | Sin red |

```yaml
services:
  app:
    networks:
      - frontend
      - backend
  db:
    networks:
      - backend

networks:
  frontend:
  backend:
```

Útil para separar servicios públicos de privados.

---

## DNS interno de Docker

Docker tiene un **DNS interno** que resuelve nombres de servicio a IPs.

```bash
# Desde dentro del contenedor app
docker compose exec app sh
/ # ping db
PING db (172.18.0.2): 56 data bytes
```

Esto funciona porque Docker Compose configura automáticamente un DNS.

---

## Exponer vs Publicar puertos

| Concepto | Instrucción | ¿Afecta al host? |
|----------|------------|-----------------|
| **Exponer** | `EXPOSE 3000` en Dockerfile | No (documentación) |
| **Publicar** | `ports: - "3000:3000"` en compose | Sí (accesible desde host) |

- `EXPOSE` es solo documentación
- `ports` es la que realmente abre el puerto

---

## Buenas prácticas con puertos

| Situación | Recomendación |
|-----------|--------------|
| Desarrollo | Mapear puertos para pruebas locales |
| Producción | No exponer DB, solo la app |
| Microservicios | Que se comuniquen por red interna |
| Puerto ocupado | Cambiar el lado izquierdo (`3307:3306`) |

En tu proyecto:
- `db` expone `3307:3306` solo en dev (con el comentario de producción)
- `app` expone `3000:3000` (necesario para que los usuarios accedan)
