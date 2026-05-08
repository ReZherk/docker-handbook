# Docker vs Máquinas Virtuales

Si has usado VirtualBox o VMWare, esto te ayudará a entender Docker por contraste.

---

## La diferencia clave

**Máquina Virtual**: cada VM tiene su propio sistema operativo completo.

**Contenedor**: todos los contenedores **comparten el mismo sistema operativo** (kernel) del anfitrión.

```
┌───────────────────────────────────────┐
│           MÁQUINA VIRTUAL              │
│                                       │
│  ┌────────────┐  ┌────────────┐       │
│  │  App A     │  │  App B     │       │
│  │  Libs A    │  │  Libs B    │       │
│  │  SO Invit. │  │  SO Invit. │       │
│  └────────────┘  └────────────┘       │
│         Hypervisor                    │
│         SO Anfitrión                  │
│         Hardware                      │
└───────────────────────────────────────┘

┌───────────────────────────────────────┐
│             CONTENEDORES               │
│                                       │
│  ┌────────────┐  ┌────────────┐       │
│  │  App A     │  │  App B     │       │
│  │  Libs A    │  │  Libs B    │       │
│  └────────────┘  └────────────┘       │
│         Docker Engine                 │
│         SO Anfitrión                  │
│         Hardware                      │
└───────────────────────────────────────┘
```

---

## Comparación directa

| Característica | VM | Contenedor |
|----------------|----|-----------|
| **Arranque** | Minutos | Segundos |
| **Tamaño** | GB (~10-50GB) | MB (~5-500MB) |
| **RAM** | GB (~2-4GB) | MB (~50-500MB) |
| **Aislamiento** | Fuerte (OS completo) | Medio (comparte kernel) |
| **Portabilidad** | Pesada | Ligera |
| **Cada app necesita** | Su propio OS | Solo librerías |

---

## ¿Entonces cuándo usar cada uno?

### Usa VM cuando:
- Necesitas otro sistema operativo (correr Linux en Windows y viceversa)
- Requieres aislamiento total de seguridad
- Trabajas con kernel modules o drivers específicos

### Usa Docker cuando:
- Desarrollo y despliegue de aplicaciones
- Microservicios
- CI/CD (integración continua)
- Escalar horizontalmente
- Asegurar que todos en el equipo tengan el mismo entorno

---

## Ejemplo práctico con tu proyecto

Tu app Node.js en Docker:

```bash
# Tamaño de la imagen final
docker images mi-app
REPOSITORY   TAG       SIZE
mi-app       latest    180MB
```

Si usaras VM:

```bash
# Ubuntu Server + Node.js + MySQL
~2.5GB
```

**Diferencia: ~12x más pequeño**

---

## Seguridad: el punto débil de Docker

- Los contenedores **comparten el kernel** del host
- Si alguien escala privilegios dentro de un contenedor, podría acceder al host
- Por eso **no se debe correr como root** dentro del contenedor
- Por eso tu docker-compose tiene el comentario: *"En producción: eliminar mapeo de puertos"*

---

## Resumen

```
VM:   Aislamiento fuerte, pesado, lento
Docker: Aislamiento suficiente, ligero, rápido

Elige Docker para apps. Elige VM para sistemas operativos.
```
