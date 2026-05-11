# Instalación de Docker

Docker se instala diferente según tu sistema operativo.

---

## Linux (Ubuntu/Debian)

```bash
# 1. Actualizar paquetes
sudo apt update

# 2. Instalar dependencias
sudo apt install ca-certificates curl

# 3. Agregar clave GPG oficial de Docker
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 4. Agregar repositorio
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. Instalar Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 6. Verificar
sudo docker run hello-world

# 7. (Opcional) Ejecutar Docker sin sudo
sudo usermod -aG docker $USER
# Cerrar sesión y volver a entrar
```

---

## Windows

1. Descargar [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Instalar (requiere WSL2)
3. Aceptar términos y reiniciar
4. Docker aparece en la bandeja del sistema

### ¿Qué es WSL2?

WSL2 (Windows Subsystem for Linux) permite ejecutar Linux dentro de Windows. Docker Desktop lo usa para correr contenedores Linux en Windows.

---

## macOS

1. Descargar [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Arrastrar a Applications
3. Abrir Docker desde Applications

---

## Verificar instalación

```bash
# Versión de Docker
docker --version

# Ver que el daemon está corriendo
docker info

# Ejecutar contenedor de prueba
docker run hello-world
```

Salida esperada de `docker run hello-world`:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

---

## Docker funciona con privilegios de root

Docker necesita permisos especiales porque crea interfaces de red, monta sistemas de archivos, etc.

Por eso en Linux necesitas `sudo` o estar en el grupo `docker`.

⚠️ **Seguridad**: estar en el grupo `docker` es equivalente a tener acceso root. Solo agrega usuarios de confianza.

---

## Probar que todo funciona con tu proyecto

```bash
# Clonar el proyecto
git clone <tu-repo>
cd docker-handbook

# Construir la imagen
docker build -t mi-app .

# Ejecutar
docker compose up
```

Si ves los logs y la app responde en `http://localhost:3000`, ¡Docker funciona!
