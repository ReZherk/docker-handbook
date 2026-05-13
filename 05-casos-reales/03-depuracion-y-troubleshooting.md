# Depuración y Troubleshooting

Comandos y técnicas para cuando algo no funciona.

---

## El comando más importante: logs

```bash
# Logs de todos los servicios
docker compose logs

# Logs en tiempo real (como tail -f)
docker compose logs -f

# Logs de un servicio específico
docker compose logs -f app

# Últimas N líneas
docker compose logs --tail=100 app
```

**Siempre revisa los logs primero.** El 80% de los problemas se ven ahí.

---

## Entrar al contenedor

```bash
# Shell interactivo
docker compose exec app sh

# Shell como root
docker compose exec -u root app sh

# Ejecutar comando específico
docker compose exec app npx prisma migrate status

# Sin compose (si el contenedor ya existe)
docker exec -it mi-app sh
```

---

## Problemas comunes y soluciones

### La app no conecta a MySQL

```bash
# 1. Verificar que MySQL está corriendo
docker compose ps

# 2. Ver healthcheck
docker ps

# 3. Probar conexión desde app
docker compose exec app sh
/ # apk add mysql-client
/ # mysql -h db -u root -proot -e "SHOW DATABASES;"

# 4. Ver logs de MySQL
docker compose logs db
```

### Error: port is already allocated

```bash
# Algo ya está usando el puerto 3000 o 3307
# Solución 1: Detener el otro proceso
# Solución 2: Cambiar puerto en compose
ports:
  - "3001:3000"  # usa 3001 en tu PC
  - "3308:3306"  # usa 3308 en tu PC
```

### Contenedor se detiene inmediatamente

```bash
# Ver qué pasó
docker compose logs app

# Entrar antes de que muera (si es muy rápido, usa tail -f)
# Tu compose ya tiene esto: || tail -f /dev/null
# El contenedor se mantiene vivo aunque falle
```

### npm ci falla en el build

```bash
# 1. Verificar que package-lock.json existe
ls package-lock.json

# 2. Si no existe, generarlo
npm install  # esto crea package-lock.json

# 3. Reconstruir
docker compose up --build
```

### Prisma generate falla

```bash
# Verificar que schema.prisma existe
docker compose exec app sh
ls prisma/schema.prisma

# Verificar DATABASE_URL
echo $DATABASE_URL

# Regenerar manualmente
npx prisma generate
```

---

## Comandos de diagnóstico

```bash
# Ver estado de todos los servicios
docker compose ps

# Ver recursos (CPU, memoria) de cada contenedor
docker stats

# Ver IPs y red
docker compose exec app sh
/ # cat /etc/hosts

# Ver variables de entorno dentro del contenedor
docker compose exec app sh
/ # env | grep -E "DATABASE|JWT|MAIL"

# Ver diferencias entre imagen construida y Dockerfile
docker diff mi-app

# Ver capas de la imagen
docker history mi-app
```

---

## Trucos de debugging

### 1. Mantener contenedor vivo para investigar

```yaml
# Si todo falla, sobreescribe el command
command: tail -f /dev/null
```

### 2. Inspeccionar el filesystem del contenedor

```bash
# Ver qué archivos hay
docker compose exec app sh
/ # ls -la /app/dist/
/ # ls -la /app/node_modules/

# Ver que prisma client existe
/ # ls /app/node_modules/@prisma/client/
```

### 3. Probar la app localmente (sin Docker)

```bash
# A veces es más fácil debuggear local
npm install
npx prisma generate
npm run build
node dist/main.js
```

### 4. Build paso a paso

```bash
# Construir hasta una etapa específica
docker build --target builder -t mi-app-builder .

# Inspeccionar la etapa builder
docker run -it mi-app-builder sh
```

### 5. Ver el contenido de una imagen

```bash
# Exportar imagen como tar
docker save mi-app -o mi-app.tar
tar -xvf mi-app.tar
# Revisar layers
```

---

## Errores comunes en tu proyecto

| Síntoma | Causa probable | Solución |
|---------|---------------|----------|
| `ECONNREFUSED db:3306` | MySQL no listo | Verificar healthcheck |
| `prisma: error: Required environment variable is empty` | `.env` faltante | Crear `.env` |
| `Cannot find module @prisma/client` | Falta prisma generate | `npx prisma generate` |
| `port 3000 already in use` | Otra app en el puerto | Cambiar puerto |
| `docker: permission denied` | Usuario no en grupo docker | `sudo usermod -aG docker $USER` |
| `no matching manifest for linux/arm64` | Arquitectura incorrecta | Usar `--platform linux/amd64` |
| `tail -f /dev/null` activo | App crasheó | Revisar logs con `docker compose logs app` |

---

## Resumen: qué hacer cuando algo falla

```
1. docker compose logs -f app
   ¿Hay errores?
   ├── Sí → Lees el error, buscas solución
   └── No → Siguiente paso

2. docker compose ps
   ¿Todos los servicios están Up?
   ├── No → docker compose logs <servicio>
   └── Sí → Siguiente paso

3. docker compose exec app sh
   Entra al contenedor, revisa archivos, variables, conexiones

4. ¿Sigue fallando?
   Prueba sin Docker (node local) para aislar el problema
```
