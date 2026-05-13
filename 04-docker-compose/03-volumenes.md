# Volúmenes en Docker

Los volúmenes resuelven el problema más grande de Docker: **los datos se pierden al borrar contenedores**.

---

## El problema

```bash
# Crear contenedor MySQL
docker run -d --name db mysql:9.6

# Guardar datos importantes
docker exec db mysql -e "CREATE DATABASE produccion;"

# Detener y eliminar
docker stop db && docker rm db

# ¡Los datos han desaparecido para siempre!
```

Los contenedores son **efímeros** por diseño. Los datos dentro del contenedor mueren con él.

---

## La solución: volúmenes

```yaml
services:
  db:
    image: mysql:9.6
    volumes:
      - mysql_data:/var/lib/mysql
      #   ↑ nombre          ↑ ruta dentro del contenedor

volumes:
  mysql_data:
```

Los datos de MySQL se guardan en el volumen `mysql_data`, que vive **fuera del contenedor**.

```
┌────────────────────────────────┐
│         CONTENEDOR DB          │
│                                │
│  /var/lib/mysql/               │
│    ├── datos.db                │
│    ├── ibdata1                 │
│    └── ...                     │
└────────────┬───────────────────┘
             │  montaje (mount)
             ▼
┌────────────────────────────────┐
│       VOLUMEN mysql_data       │
│    (almacenado en el host)     │
└────────────────────────────────┘
```

---

## Tipos de volúmenes

### 1. Named Volumes (usado en tu proyecto)

```yaml
volumes:
  mysql_data:      # Docker lo gestiona
```

- Docker maneja la ubicación (usualmente `/var/lib/docker/volumes/`)
- No necesitas saber dónde está
- Persiste aunque borres el contenedor

```bash
# Ver volúmenes
docker volume ls

# Inspeccionar
docker volume inspect mysql_data

# Eliminar volumen (¡cuidado! borra datos)
docker volume rm mysql_data
```

### 2. Bind Mounts

```yaml
services:
  app:
    volumes:
      - ./src:/app/src   # directorio local : directorio contenedor
```

- Montas un directorio de tu PC directamente
- Útil para desarrollo (cambias código y se refleja al instante)
- No recomendado para producción

### 3. tmpfs Mounts

```yaml
services:
  app:
    tmpfs: /app/cache
```

- Almacenamiento en RAM
- Se borra al detener el contenedor
- Útil para datos temporales (cachés, sesiones)

---

## ¿Dónde están los volúmenes en tu PC?

```bash
# Linux (Docker Engine)
/var/lib/docker/volumes/mysql_data/_data/

# Windows/Mac (Docker Desktop)
# Dentro de la VM de Docker
# Accedes via: docker volume inspect
```

No deberías tocar estos archivos directamente. Siempre usa comandos Docker.

---

## Buenas prácticas con volúmenes

```yaml
volumes:
  mysql_data:
    # Sin driver específico → usa el driver local
```

### ¿Cuándo usar cada tipo?

| Tipo | Uso | Ejemplo |
|------|-----|---------|
| **Named Volume** | Producción, datos persistentes | `mysql_data:/var/lib/mysql` |
| **Bind Mount** | Desarrollo, hot-reload | `./src:/app/src` |
| **tmpfs** | Caché, sesiones | `/tmp` en RAM |

### Backup de volúmenes

```bash
# Crear backup de un volumen
docker run --rm -v mysql_data:/data -v $(pwd):/backup alpine tar czf /backup/mysql_backup.tar.gz -C /data .

# Restaurar
docker run --rm -v mysql_data:/data -v $(pwd):/backup alpine tar xzf /backup/mysql_backup.tar.gz -C /data
```

---

## Volúmenes en tu proyecto

```yaml
services:
  db:
    volumes:
      - mysql_data:/var/lib/mysql
      # MySQL guarda aquí sus bases de datos
      #   ├── globtecx/
      #   ├── mysql/
      #   ├── performance_schema/
      #   └── sys/

volumes:
  mysql_data:  # Se crea al hacer docker compose up
```

Si haces `docker compose down`, el contenedor se elimina pero el volumen **no** (datos seguros).

Si haces `docker compose down -v`, el volumen **también** se elimina (datos perdidos).

⚠️ **Nunca uses `-v` en producción sin verificar.**

---

## ¿Y qué hay de persistencia en la app?

Tu app Node.js no tiene volúmenes. ¿Por qué?

Porque la app es **stateless**:
- No guarda archivos localmente
- Todo se guarda en la base de datos
- Puedes reiniciar la app sin perder datos

Si la app subiera archivos (imágenes, PDFs), necesitarías un volumen para `./uploads`.
