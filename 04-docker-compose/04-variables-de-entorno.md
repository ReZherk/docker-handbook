# Variables de Entorno

Tu docker-compose.yml está lleno de variables de entorno. Es la forma correcta de pasar configuración a los contenedores.

---

## ¿Por qué variables de entorno?

Nunca debes hardcodear configuraciones sensibles en el código:

```javascript
// MAL: hardcodeado
const DATABASE_URL = "mysql://root:root@db:3306/globtecx";
const JWT_SECRET = "mi-super-secreto-123";

// BIEN: desde variable de entorno
const DATABASE_URL = process.env.DATABASE_URL;
const JWT_SECRET = process.env.JWT_SECRET;
```

Beneficios:
- Mismo código para dev, staging, producción
- Secrets no están en el repositorio
- Fácil de cambiar sin rebuild

---

## environment vs env_file

En docker-compose puedes pasar variables de dos formas:

### 1. Directamente en el YAML

```yaml
app:
  environment:
    DATABASE_URL: "mysql://root:root@db:3306/globtecx"
    DATABASE_HOST: db
    NODE_ENV: production
```

**Ventaja**: visible y explícito
**Desventaja**: secrets visibles en el archivo

### 2. Desde un archivo .env

```yaml
app:
  env_file:
    - .env
```

Y en `.env`:

```
JWT_SECRET=mi-super-secreto-123
JWT_EXPIRES_IN=1d
MAIL_HOST=smtp.gmail.com
```

**Ventaja**: `.env` se puede agregar a `.gitignore` (no se sube al repo)
**Desventaja**: un archivo más que gestionar

---

## Variables compartidas: docker-compose.env

```yaml
app:
  env_file:
    - docker-compose.env  # compartido entre servicios
```

**Tu proyecto** mezcla ambos: las vars de DB están en el YAML (no son secretos), mientras que JWT y MAIL vienen del `.env`:

```yaml
app:
  environment:
    # Públicas (no secretos)
    DATABASE_URL: "mysql://root:root@db:3306/globtecx"
    DATABASE_HOST: db
    # ... más vars de DB

    # Privadas (secretos, vienen de .env)
    JWT_SECRET: ${JWT_SECRET}
    JWT_EXPIRES_IN: ${JWT_EXPIRES_IN:-1d}
```

---

## ${VAR} y ${VAR:-default}

Docker Compose soporta **interpolación de variables** desde un archivo `.env`:

```yaml
# Si JWT_SECRET existe en .env, usa ese valor
JWT_SECRET: ${JWT_SECRET}

# Si JWT_EXPIRES_IN no existe, usa "1d" como valor por defecto
JWT_EXPIRES_IN: ${JWT_EXPIRES_IN:-1d}
```

Los dos formatos:

| Formato | Significado |
|---------|-------------|
| `${VAR}` | Usa el valor de VAR. **Error** si no existe |
| `${VAR:-default}` | Usa `default` si VAR no existe o está vacío |
| `${VAR:?error}` | Error personalizado si VAR no existe |

En tu proyecto:
```yaml
JWT_EXPIRES_IN: ${JWT_EXPIRES_IN:-1d}   # default: 1 día
SALT_ROUNDS: ${SALT_ROUNDS:-12}          # default: 12 rounds
JWT_RESET_EXPIRES_IN: ${JWT_RESET_EXPIRES_IN:-15m}  # default: 15 min
```

Esto permite que funcione **sin archivo .env** usando valores por defecto.

---

## El archivo .env

Crea un archivo `.env` en la raíz del proyecto:

```
JWT_SECRET=mi-super-secreto-cambiame-en-produccion
JWT_EXPIRES_IN=1d
JWT_RESET_SECRET=otro-secreto-cambiame
JWT_RESET_EXPIRES_IN=15m
SALT_ROUNDS=12
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USER=tu-correo@gmail.com
MAIL_PASSWORD=tu-contrasena-de-app
MAIL_FROM=tu-correo@gmail.com
```

Y agrega `.env` al `.gitignore`:

```
.env
```

Crea un `.env.example` (sin valores reales) para que otros sepan qué necesitan:

```
JWT_SECRET=
JWT_EXPIRES_IN=1d
JWT_RESET_SECRET=
JWT_RESET_EXPIRES_IN=15m
SALT_ROUNDS=12
MAIL_HOST=
MAIL_PORT=587
MAIL_USER=
MAIL_PASSWORD=
MAIL_FROM=
```

---

## Variables en tu proyecto: tabla

| Variable | ¿Dónde se define? | ¿Es secreto? |
|----------|-------------------|-------------|
| `DATABASE_URL` | YAML directo | No (solo dev) |
| `DATABASE_HOST` | YAML directo | No |
| `MYSQL_ROOT_PASSWORD` | YAML directo | En dev no, en prod sí |
| `JWT_SECRET` | `.env` | **Sí** |
| `JWT_EXPIRES_IN` | `.env` (con default) | No |
| `MAIL_PASSWORD` | `.env` | **Sí** |

---

## Buenas prácticas

1. **Nunca subir `.env` al repositorio** (agregar a `.gitignore`)
2. **Siempre tener `.env.example`** con valores dummy
3. **Valores por defecto** para vars no críticas (`${VAR:-default}`)
4. **Separar secrets de configuración** (environment vs env_file)
5. **En producción**: usar secrets de Docker o un vault (HashiCorp Vault, AWS Secrets Manager)

---

## ¿Cómo accede tu app a estas variables?

```javascript
// Tu app Node.js
const express = require('express');
const app = express();

// Docker inyecta estas variables automáticamente
const databaseUrl = process.env.DATABASE_URL;
const jwtSecret = process.env.JWT_SECRET;

// Prisma las usa para conectar
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: databaseUrl,
    },
  },
});
```

Docker Compose inyecta las variables de entorno en el contenedor. Tu app las lee con `process.env`.
