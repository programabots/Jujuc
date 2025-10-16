# Servidor de Minecraft 24/7 con IP fácil

Este repositorio contiene una configuración basada en **Docker** para desplegar un servidor de Minecraft Java Edition que se mantenga encendido 24/7 y pueda publicarse mediante un dominio o subdominio sencillo. Incluye:

- `docker-compose.yml` con la imagen oficial de [itzg/minecraft-server](https://github.com/itzg/docker-minecraft-server).
- Variables de entorno parametrizables en `.env` para personalizar mundo, memoria, modo de juego y complementos.
- Instrucciones para exponer el servidor con un dominio a través de Cloudflare Tunnel o servicios DynDNS gratuitos.

## Requisitos

- Un servidor o VPS con Ubuntu 22.04 (o similar) y al menos 2 GB de RAM.
- Docker 24+ y Docker Compose Plugin (`docker compose`).
- Un dominio propio o subdominio administrado en Cloudflare (opcional pero recomendado para "IP fácil").

## Pasos de instalación

1. **Clonar el repositorio y preparar el entorno**
   ```bash
   git clone https://github.com/<tu-usuario>/Jujuc.git
   cd Jujuc
   cp .env.example .env
   ```
   > Si aún no tienes Docker instalado, ejecuta `./scripts/install_docker.sh` y vuelve a iniciar sesión para que se aplique la pertenencia al grupo `docker`.
2. **Editar `.env`**
   - Acepta el EULA (`EULA=TRUE`).
   - Ajusta memoria (`MEMORY=4G` recomendado) y resto de variables.
   - Si deseas habilitar plugins de Paper/Spigot, cambia `TYPE=PAPER` y agrega los `MODS` necesarios.
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

