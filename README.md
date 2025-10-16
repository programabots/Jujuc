# Servidor de Minecraft 24/7 con IP fácil

Este repositorio contiene una configuración basada en **Docker** para desplegar un servidor de Minecraft Java Edition que se mantenga encendido 24/7 y pueda publicarse mediante un dominio o subdominio sencillo. Incluye:

- `docker-compose.yml` con la imagen oficial de [itzg/minecraft-server](https://github.com/itzg/docker-minecraft-server).
- Variables de entorno parametrizables en `.env` para personalizar mundo, memoria, modo de juego y complementos.
- Instrucciones para exponer el servidor con un dominio a través de Cloudflare Tunnel o servicios DynDNS gratuitos.

## Requisitos

- Un servidor o equipo con un sistema operativo compatible con Docker y al menos 2 GB de RAM. Recomendado:
  - **Linux** (Debian 12, Ubuntu 22.04/24.04, Fedora 39+).
  - **Windows 10/11 Pro o Enterprise** con Docker Desktop o WSL 2.
  - **macOS 13 Ventura o superior** con Docker Desktop.
- Docker 24+ y Docker Compose Plugin (`docker compose`).
- Git 2.x (opcional; solo si prefieres clonar el repositorio en lugar de descargar el ZIP).
- Un dominio propio o subdominio administrado en Cloudflare (opcional pero recomendado para "IP fácil").

> ¿No puedes utilizar Ubuntu? No hay problema: más abajo encontrarás rutas específicas para Debian, Fedora, Windows y macOS.

## Pasos de instalación

1. **Obtener el proyecto y preparar el entorno**
   - **Con Git (recomendado)**
     ```bash
     git clone https://github.com/<tu-usuario>/Jujuc.git
     cd Jujuc
     cp .env.example .env
     ```
   - **Sin Git**: descarga el ZIP desde el botón **Code → Download ZIP** del repositorio, súbelo al servidor y descomprímelo:
     ```bash
     unzip Jujuc-main.zip
     cd Jujuc-main
     cp .env.example .env
     ```
   > Si aún no tienes Docker instalado, revisa la sección **Instalación de Docker por sistema operativo** y vuelve a iniciar sesión para que se aplique la pertenencia al grupo `docker` (en Linux).
2. **Editar `.env`**
   - Acepta el EULA (`EULA=TRUE`).
   - Ajusta memoria (`MEMORY=4G` recomendado) y resto de variables.
   - Si deseas habilitar plugins o mods, cambia `TYPE` según la plataforma (Paper, Forge, Fabric, Quilt, etc.) y completa las listas `PLUGINS`/`MODS`.
3. **Crear carpetas de datos y permisos**
   ```bash
   mkdir -p data/backups
   ```
4. **Levantar el servidor**
   ```bash
   docker compose up -d
   ```
   - El servidor se mantendrá siempre activo gracias a `restart: always`.
   - Los mundos, configuraciones y backups quedarán en el directorio `data/`.

5. **Verificar logs**
   ```bash
   docker compose logs -f mc
   ```

### Ejemplo práctico: iniciar en Windows (PowerShell o WSL)

Si trabajas desde Windows 10/11 con Docker Desktop ya instalado, puedes seguir estos comandos:

1. Abre **PowerShell** y navega al proyecto:
   ```powershell
   Set-Location "C:\Users\tuusuario\Jujuc"
   ```
   > Si usas WSL, coloca el proyecto dentro de tu distribución (por ejemplo `/home/<usuario>/Jujuc`) y ejecuta los mismos comandos de Linux.
2. Copia la plantilla de variables:
   ```powershell
   Copy-Item .env.example .env
   ```
3. Edita `.env` con tus parámetros (EULA, memoria, mods/plugins).
4. Crea la carpeta de datos y respaldos persistentes:
   ```powershell
   New-Item -ItemType Directory -Path data\backups -Force | Out-Null
   ```
5. Arranca el servidor en segundo plano:
   ```powershell
   docker compose up -d
   ```
6. Supervisa el arranque revisando los logs:
   ```powershell
   docker compose logs -f mc
   ```

> Si prefieres mover todo el flujo a WSL (Debian/Ubuntu), ejecuta `cp .env.example .env` y `mkdir -p data/backups` en lugar de los equivalentes de PowerShell, pero el comando `docker compose up -d` es el mismo.

## Backups programados

El contenedor incluye un `cron` interno que puede ejecutar respaldos automáticos si habilitas `ENABLE_AUTOPAUSE=false` y `BACKUP_INTERVAL`. Edita la sección de "Opciones avanzadas" en `.env` para configurarlo.

## IP fácil con dominio propio

### Opción A: Cloudflare Tunnel (recomendado)

1. Crea un túnel gratuito en [Cloudflare Zero Trust](https://one.dash.cloudflare.com/).
2. Instala el conector `cloudflared` en tu VPS:
   ```bash
   curl -fsSL https://pkg.cloudflare.com/cloudflared/install.sh | sudo bash
   sudo cloudflared service install <TOKEN_DEL_TUNEL>
   ```
3. Añade una ruta TCP al túnel hacia `localhost:25565`.
4. En la sección **Public Hostname**, asigna `mc.midominio.com` al túnel.
5. Conecta desde Minecraft usando `mc.midominio.com` sin exponer puertos en tu router.

### Opción B: DynDNS / DuckDNS

1. Registra un subdominio gratuito en [DuckDNS](https://www.duckdns.org/).
2. Configura un script que actualice la IP pública del servidor cada 5 minutos.
3. Abre el puerto `25565` en tu router hacia el servidor.
4. Usa el subdominio `tuservidor.duckdns.org` para ingresar al servidor.

## Despliegue administrado en Railway

Si prefieres delegar la infraestructura y mantener el servidor encendido sin gestionar un VPS propio, puedes alojarlo en [Railway](https://railway.app/) aprovechando que soportan imágenes Docker.

1. **Preparar el proyecto**
   - Haz un fork de este repositorio o conéctalo directamente desde Railway.
   - Añade un archivo `.env` con las variables necesarias desde el panel de **Variables** de Railway (no subas tu `.env` local al repositorio).
2. **Configurar la imagen**
   - Railway detectará el `docker-compose.yml`, pero para un servicio administrado conviene apuntar a la imagen oficial.
   - En la sección **Deployments → Docker Image** selecciona `itzg/minecraft-server:latest` o indica este repositorio con el builder `Docker` activado para que utilice la imagen base automáticamente.
3. **Configurar puertos y recursos**
   - Define el puerto `25565` como público en la pestaña **Networking**.
   - Ajusta la memoria del plan contratado para que sea, como mínimo, 2 GB.
4. **Persistencia del mundo**
   - Crea un volumen desde **Storage → Volumes** y asígnalo al servicio montándolo en la ruta `/data`.
   - Opcionalmente crea un segundo volumen para `/backups` si deseas separar respaldos automáticos.
5. **Primer despliegue**
   - Pulsa **Deploy** y monitorea los logs desde Railway (`railway logs`).
   - Una vez que el contenedor finalice la inicialización, podrás conectarte usando la URL asignada por Railway o tu dominio personalizado apuntando al host proporcionado.

> Consejo: Puedes automatizar actualizaciones con `railway up --service mc --detach` cada vez que cambies la configuración o quieras reiniciar el servidor.

## Administración diaria

- Para apagar el servidor:
  ```bash
  docker compose down
  ```
- Para actualizar a la última versión de Minecraft:
  ```bash
  docker compose pull
  docker compose up -d
  ```
- Para ejecutar comandos de la consola:
  ```bash
  docker compose exec mc rcon-cli <comando>
  ```

## Personalización avanzada

- Modpacks: habilita `TYPE=CURSEFORGE` y define `CF_SERVER_MOD`.
- Whitelist: configura `WHITELIST` en `.env` con una lista separada por comas.
- Resource packs: define `RESOURCE_PACK` y `RESOURCE_PACK_SHA1`.
- Rendimiento: ajusta `SERVER_MAX_PLAYERS`, `VIEW_DISTANCE` y `SIMULATION_DISTANCE`.

## Cómo añadir plugins y mods

El contenedor de `itzg/minecraft-server` soporta múltiples plataformas. Todo el contenido se guarda en `./data`, por lo que puedes administrar plugins y mods editando variables en `.env` o copiando archivos manualmente dentro de esa carpeta.

### Plugins para Paper / Spigot / Purpur

1. En `.env`, establece `TYPE=PAPER` (o `PURPUR`).
2. Añade en `PLUGINS` los archivos `.jar` a descargar automáticamente. Deben ser URLs directas, por ejemplo:
   ```env
   PLUGINS="https://hangarcdn.papermc.io/plugins/PlaceholderAPI/PlaceholderAPI/versions/2.11.5/Paper/PlaceholderAPI-2.11.5.jar \
   https://cdn.modrinth.com/data/gvQqBUdM/versions/latest/spark.jar"
   ```
3. Si prefieres gestionarlos manualmente, coloca los `.jar` dentro de `data/plugins/` (se crea tras el primer arranque) y reinicia el servicio con `docker compose restart mc`.
4. Opcional: pon `REMOVE_OLD_PLUGINS=true` para que el contenedor limpie plugins obsoletos antes de cada arranque.

### Mods para Forge / Fabric / Quilt

1. Cambia `TYPE` a la plataforma deseada (`TYPE=FORGE`, `TYPE=FABRIC`, etc.).
2. Usa `MODS` para listar URLs directas a los mods que quieras descargar:
   ```env
   MODS="https://media.forgecdn.net/files/4592/858/jei-1.19.4.jar \
   https://cdn.modrinth.com/data/P7dR8mSH/versions/latest/lithium-fabric-mc1.19.4.jar"
   ```
3. Para cargar muchos mods a la vez, apunta `MODS_FILE` a un archivo `.txt` con una URL por línea (puedes guardarlo en `./data/mods.txt`).
4. También puedes copiar los mods manualmente al directorio `data/mods/` y reiniciar el contenedor.
5. Si actualizas la lista de mods frecuentemente, habilita `REMOVE_OLD_MODS=true` para que se eliminen los que ya no estén en la lista.

### Modpacks CurseForge o Modrinth

1. Cambia `TYPE=CURSEFORGE` o `TYPE=MODRINTH` según el origen.
2. Configura `CF_SERVER_MOD` o `MODRINTH_PROJECT`/`MODRINTH_VERSION` con la URL o ID del modpack.
3. Ajusta la memoria (`MEMORY`) y distancia de renderizado para cubrir los requisitos del modpack.

> Después de cualquier cambio en `.env`, ejecuta `docker compose up -d` para aplicar la configuración. Revisa `docker compose logs -f mc` para monitorear la descarga automática de mods/plugins.

## Seguridad

- Usa contraseñas seguras para RCON (`RCON_PASSWORD`).
- Limita los puertos abiertos en el firewall (`ufw allow 25565/tcp`).
- Mantén el sistema y Docker actualizados (`sudo apt update && sudo apt upgrade`).

## Troubleshooting

- **El servidor no arranca**: revisa `docker compose logs -f mc` y verifica memoria disponible.
- **Lag**: incrementa la memoria (`MEMORY=6G`) o reduce `VIEW_DISTANCE`.
- **Jugadores no pueden conectarse**: confirma que el túnel o puerto está correctamente configurado y que la IP pública no cambió.

---

Si necesitas automatizar despliegues en múltiples servidores, puedes integrar esta configuración con Ansible o Terraform.

## Instalación de Docker por sistema operativo

Dependiendo de tu plataforma puedes seguir uno de los siguientes caminos:

### Debian 12 (o derivados sin Ubuntu)

1. Actualiza paquetes básicos:
   ```bash
   sudo apt update && sudo apt install -y ca-certificates curl gnupg
   ```
2. Añade el repositorio oficial de Docker:
   ```bash
   curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
     $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
     sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt update
   ```
3. Instala Docker Engine y el plugin Compose:
   ```bash
   sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
4. Añade tu usuario al grupo `docker`:
   ```bash
   sudo usermod -aG docker $USER
   ```
   Vuelve a iniciar sesión para que surta efecto.

### Fedora 39+

1. Habilita el repositorio y paquetes necesarios:
   ```bash
   sudo dnf -y install dnf-plugins-core
   sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
   sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
2. Habilita y arranca el servicio de Docker:
   ```bash
   sudo systemctl enable --now docker
   sudo usermod -aG docker $USER
   ```
3. Cierra sesión y vuelve a entrar para usar Docker sin `sudo`.

### Windows 10/11 (sin Ubuntu)

1. Instala [Docker Desktop para Windows](https://www.docker.com/products/docker-desktop/) asegurándote de que WSL 2 esté habilitado.
2. En **Settings → Resources**, asigna al menos 4 GB de RAM.
3. Clona o descomprime el proyecto dentro de tu carpeta de usuario (por ejemplo `C:\Users\tuusuario\Jujuc`).
4. Abre **PowerShell** o **WSL** y ejecuta los mismos comandos `docker compose` descritos en la guía.

> Nota: si prefieres no usar Docker Desktop, puedes instalar [Docker Engine sobre WSL 2](https://learn.microsoft.com/windows/wsl/tutorials/wsl-containers) utilizando distribuciones como Debian o AlmaLinux disponibles en Microsoft Store.

### macOS 13+

1. Descarga e instala [Docker Desktop para Mac](https://www.docker.com/products/docker-desktop/).
2. Permite la asignación de recursos necesarios (mínimo 4 GB de RAM) desde **Settings → Resources**.
3. Abre la aplicación **Terminal** y navega al directorio del proyecto para ejecutar `docker compose up -d`.

---

