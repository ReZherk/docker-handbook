# Imágenes vs Contenedores

Este es el concepto más importante para dominar Docker.

---

## Imagen = la receta de cocina

Una **imagen** es un **archivo inmodificable** que contiene:

- Un sistema operativo base (ej: `alpine`, `ubuntu`)
- Dependencias (Node.js, Python, librerías)
- Tu código fuente
- Variables de entorno por defecto
- Instrucciones de arranque

```bash
# Listar imágenes que tienes en tu PC
docker images

# Ver la imagen de node que usas en tu proyecto
docker image inspect node:25-alpine
```

Características:
- **Read-only**: no se puede modificar
- **Capas (layers)**: cada instrucción del Dockerfile crea una capa
- **Ligera**: las imágenes Alpine pesan ~5MB (vs Ubuntu ~200MB)

---

## Contenedor = el plato servido

Un **contenedor** es una **instancia en ejecución** de una imagen.

```bash
# Crear un contenedor y ejecutarlo
docker run node:25-alpine

# Listar contenedores activos
docker ps

# Listar todos los contenedores (incluyendo detenidos)
docker ps -a

# Listar solo IDs (útil para scripts)
docker ps -q
```

Características:
- **Read-Write**: puede crear/modificar archivos dentro
- **Aislado**: no ve los procesos de tu PC
- **Efímero**: si lo borras, los cambios dentro se pierden (a menos que uses volúmenes)
- **Tiene un ciclo de vida**: creas → ejecutas → detienes → borras

---

## Analogía de la cocina

| Concepto | Cocina | Docker |
|----------|--------|--------|
| Imagen | Receta escrita | `node:25-alpine` |
| Contenedor | Plato cocinado | `docker run node:25-alpine` |
| Dockerfile | La receta paso a paso | Tu archivo Dockerfile |
| Capas | Cada paso de la receta | Cada instrucción del Dockerfile |

---

## ¿Qué pasa con tus datos?

- Los cambios dentro del contenedor **se pierden** al borrarlo
- Para persistir datos necesitas **volúmenes** (lo veremos en docker-compose)
- Ejemplo: tu base de datos MySQL guarda datos en `/var/lib/mysql` dentro del contenedor. Si borras el contenedor, ¡adiós datos!

```yaml
# Solución: montar un volumen
volumes:
  - mysql_data:/var/lib/mysql  # los datos sobreviven
```

---

## Ciclo de vida de una imagen

```
docker pull node:25-alpine
    │
    ▼
Imagen guardada localmente
    │
    ▼
docker run node:25-alpine
    │
    ├── docker stop → Contenedor detenido
    │   (puedes reiniciarlo con docker start)
    │
    ├── docker rm → Contenedor eliminado (datos perdidos)
    │
    └── docker rmi node:25-alpine → Imagen eliminada
```

---

## Comandos esenciales para practicar

```bash
# Descargar una imagen
docker pull node:25-alpine

# Crear y ejecutar contenedor interactivo (terminal dentro)
docker run -it node:25-alpine sh

# Ejecutar comando dentro de contenedor existente
docker exec -it mi-contenedor sh

# Ver logs
docker logs mi-contenedor

# Ver detalles del contenedor
docker inspect mi-contenedor
```

---

## En tu proyecto

Tu Dockerfile tiene dos etapas:
1. `AS builder`: la imagen **temporal** que contiene todo (dev dependencies, compilación)
2. La imagen **final**: solo lo necesario para correr

```
Builder (imagen grande) → Compila → Artefactos → Imagen final (pequeña)
```

Esto se llama **multi-stage build** y lo veremos a detalle en el capítulo de Dockerfile.
