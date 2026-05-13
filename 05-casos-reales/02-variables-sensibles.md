# Variables Sensibles y Seguridad

Cómo manejar secrets (contraseñas, tokens, claves) de forma segura en Docker.

---

## El error más común

```dockerfile
# MAL: el secret queda grabado en la imagen
ENV JWT_SECRET=mi-super-secreto
ENV DATABASE_URL=mysql://root:root@db:3306/globtecx
```

```bash
# Cualquiera puede ver los secrets
docker history mi-app
# o
docker inspect mi-app
```

**Los secrets en el Dockerfile son visibles para siempre.** No importa que luego los sobreescribas en compose.

---

## La solución: variables de entorno externas

```yaml
app:
  environment:
    # Bien: referencia a variable externa
    JWT_SECRET: ${JWT_SECRET}
    # Mal: hardcodeado
    JWT_SECRET: mi-super-secreto
```

Las variables se inyectan **en tiempo de ejecución**, no quedan en la imagen.

---

## Estrategias para manejar secrets

### 1. Archivo .env (desarrollo)

```
JWT_SECRET=desarrollo-secreto-123
```

Agregar `.env` a `.gitignore`:

```
.env
```

### 2. Docker Secrets (producción)

```yaml
# docker-compose.yml (producción)
app:
  secrets:
    - jwt_secret

secrets:
  jwt_secret:
    file: ./secrets/jwt_secret.txt
```

```bash
# Docker monta el secret como archivo
# En /run/secrets/jwt_secret
```

### 3. Variables de entorno del sistema

```bash
# Exportar variable antes de ejecutar compose
export JWT_SECRET="produccion-secreto-456"
docker compose up
```

### 4. Gestores de secrets (producción real)

- **HashiCorp Vault**
- **AWS Secrets Manager**
- **Azure Key Vault**
- **Google Secret Manager**

---

## Comentario de producción en tu compose

```yaml
db:
  ports:
    - "3307:3306"
  # EN PRODUCCIÓN: Eliminar el mapeo de puertos (ports)
```

No exponer la DB al exterior es una **medida de seguridad básica**:

```
❌ Producción (DB expuesta):
  Internet ──► Host:3307 ──► DB
  Cualquiera puede intentar conectarse

✅ Producción (DB aislada):
  Internet ──► Host:3000 ──► App ──► Red interna ──► DB:3306
  Solo la app puede conectar a DB
```

---

## Buenas prácticas de seguridad

### En el Dockerfile

```dockerfile
# 1. No correr como root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# 2. No hardcodear secrets
ENV JWT_SECRET=...  # NUNCA

# 3. Imagen base específica (no latest)
FROM node:25-alpine  # ✅
FROM node            # ❌

# 4. Minimizar capas y herramientas
RUN npm ci --only=production  # Solo lo necesario
```

### En docker-compose.yml

```yaml
# 1. No exponer servicios internos
db:
  # ports: - "3307:3306"  ← Comentar/eliminar en producción

# 2. Usar env_file para secrets
app:
  env_file:
    - .env  # No subir al repo

# 3. Restart policy adecuada
app:
  restart: unless-stopped
```

### En el código

```javascript
// Siempre desde environment, nunca hardcodeado
const jwtSecret = process.env.JWT_SECRET;

// Validar que existe
if (!jwtSecret) {
  throw new Error('JWT_SECRET no está definido');
}
```

---

## .gitignore para proyectos Docker

```
# Node
node_modules/
dist/

# Docker
.env
*.log

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db
```

---

## Checklist de seguridad

- [ ] `.env` en `.gitignore` ✅
- [ ] `JWT_SECRET` y similares vienen de variables de entorno ✅
- [ ] DB no expuesta en producción ✅ (tienes el comentario)
- [ ] Imagen base con tag específico ✅ (`node:25-alpine`)
- [ ] Usuario no root ❌ (opcional agregar)
- [ ] Puerto de DB diferente en desarrollo (3307) ✅
- [ ] `npm ci` para builds determinísticos ✅
