# npm ci vs npm install

Tu Dockerfile usa `npm ci` en lugar de `npm install`. No es un error tipográfico.

---

## ¿Cuál es la diferencia?

```dockerfile
# Tu proyecto usa esto:
RUN npm ci --include=dev

# Mucha gente usa esto (mal):
RUN npm install
```

---

## npm ci (clean install)

| Característica | `npm ci` |
|----------------|----------|
| Requiere `package-lock.json` | **Sí** |
| Borra `node_modules` primero | Sí |
| Instala versiones **exactas** | Sí (del lockfile) |
| Velocidad | Rápido (sin resolución de versiones) |
| ¿Modifica `package.json`? | No |

**Usa `npm ci` en CI/CD y Docker.** Garantiza que todos obtengan exactamente las mismas versiones.

---

## npm install

| Característica | `npm install` |
|----------------|---------------|
| Requiere `package-lock.json` | No |
| Borra `node_modules` primero | No |
| Instala versiones **exactas** | Depende |
| Velocidad | Lento (resuelve versiones) |
| ¿Modifica `package.json`? | Puede (agrega `^` si no hay lock) |

**Usa `npm install` solo en desarrollo local** cuando agregas/quitas paquetes.

---

## El problema que `npm ci` resuelve

```bash
# En tu PC (Linux):
npm install
# package-lock.json queda con resoluciones de Linux

# Tu compañero (Mac):
npm install
# package-lock.json cambia (resoluciones de Mac)

# Docker (Alpine Linux):
npm install
# package-lock.json cambia otra vez
```

Con `npm ci`:
- Usa el `package-lock.json` tal cual
- No lo modifica
- Instalación **determinística**: misma versión en todas partes

---

## Flags en tu proyecto

```dockerfile
# En la etapa de BUILD (necesita devDependencies para compilar)
RUN npm ci --include=dev
# --include=dev: instala también devDependencies

# En la etapa FINAL (solo producción)
RUN npm ci --only=production
# --only=production: instala solo dependencias de producción
```

Esto también es **buena práctica**:
- Para compilar necesitas TypeScript, Prisma CLI, etc. (devDependencies)
- Para ejecutar solo necesitas express, @prisma/client, etc. (dependencies)

---

## Cuándo usar cada uno

| Situación | Comando |
|-----------|---------|
| Desarrollo local, agregar paquete | `npm install express` |
| Desarrollo local, instalar todo | `npm install` |
| CI/CD, Docker, despliegues | `npm ci` |
| Producción (solo necesario) | `npm ci --only=production` |

---

## package.json vs package-lock.json

```
package.json
  ├── "express": "^4.18.0"   → versión flexible
  │
  └── package-lock.json
      ├── "express@4.18.0"   → versión exacta
      ├── "express@4.18.1"   → versión exacta (si se actualizó)
      └── "express@4.19.0"   → versión exacta

npm install → puede instalar 4.19.0 (la más reciente dentro del rango ^)
npm ci      → instala EXACTAMENTE lo que dice package-lock.json
```

En Docker, `npm ci` evita sorpresas. Lo que funciona hoy, funciona mañana.
