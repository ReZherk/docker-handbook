# ¿Qué es Docker?

## El problema antes de Docker

Imagina que estás desarrollando una app Node.js con MySQL. En tu equipo funciona perfecto, pero cuando se lo pasas a un compañero:

- Él tiene otra versión de Node
- Su MySQL tiene un puerto diferente
- Usa Windows y tú usas Linux
- Faltan dependencias del sistema

**Docker resuelve esto**: empaqueta tu aplicación con **todo lo que necesita** para funcionar (código, librerías, variables de entorno, configuraciones) en algo llamado **contenedor**.

---

## Analogía simple

Docker es como una **mudanza organizada**:

| Sin Docker | Con Docker |
|------------|------------|
| Llevas tus cosas sueltas en bolsas | Todo en **cajas etiquetadas** |
| Al llegar, armas muebles desde cero | Llegas, abres la caja y todo funciona |
| Cosas se pierden o rompen | Cada caja tiene exactamente lo que necesita |

Un **contenedor Docker** es esa "caja" que contiene tu app + todo lo que necesita.

---

## Contenedor vs Imagen

Dos conceptos que siempre van juntos:

| Imagen | Contenedor |
|--------|------------|
| El **plano** o **receta** | La **casa** construida del plano |
| Es un archivo (read-only) | Es el proceso corriendo |
| Se comparte, se descarga | Se crea, se inicia, se detiene |
| Ej: `node:25-alpine` | Ej: `mi-app corriendo en el puerto 3000` |

```bash
# Descargas la imagen (el plano)
docker pull node:25-alpine

# Creas y ejecutas un contenedor (la casa)
docker run node:25-alpine
```

---

## Docker no es una VM

Máquina Virtual:
- Simula un **sistema operativo completo**
- Ocupa GB
- Arranca en minutos

Contenedor:
- Comparte el **mismo kernel** que tu PC
- Ocupa MB
- Arranca en segundos

```
VM:   [App] [Libs] [SO invitado] → [Hypervisor] → [SO anfitrión]
Docker: [App] [Libs] → [Docker] → [SO anfitrión]
```

---

## Terminología clave

| Término | Significado |
|---------|-------------|
| **Image** | Plantilla read-only para crear contenedores |
| **Container** | Instancia en ejecución de una imagen |
| **Dockerfile** | Receta para construir una imagen |
| **Docker Hub** | "Play Store" de imágenes Docker |
| **Registry** | Donde se almacenan las imágenes |
| **Daemon** | El motor de Docker que corre en segundo plano |
| **CLI** | El comando `docker` que usas en terminal |

---

## Flujo típico

```
1. Dockerfile  → 2. docker build  → 3. Imagen  → 4. docker run  → 5. Contenedor
   (receta)       (cocinar)           (comida)     (servir)         (plato listo)
```

---

## ¿Qué necesitas instalar?

- **Docker Desktop** (Windows/Mac) o **Docker Engine** (Linux)
- Veremos instalación en el próximo capítulo

---

## Para seguir aprendiendo

En los siguientes capítulos veremos:
- Cómo instalar Docker
- Comandos básicos para manejar contenedores
- **Tu Dockerfile** línea por línea
- **Tu docker-compose.yml** paso a paso
