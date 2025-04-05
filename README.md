**Zabbix con Docker Compose**
==========================

Este proyecto utiliza Docker Compose para desplegar un entorno completo de Zabbix, incluyendo el servidor, el frontend web, el agente y el servidor de PostgreSQL. A continuación, se detallan las características y configuraciones del proyecto.

## Requisitos Previos

- **Docker**: Asegúrate de tener Docker instalado en tu sistema.
- **Docker Compose**: Necesitarás Docker Compose para ejecutar este proyecto.

## Servicios

Este proyecto incluye los siguientes servicios:

### 1. **Zabbix Server**
- **Imagen**: `zabbix/zabbix-server-pgsql:alpine-7.0-latest`
- **Puertos**: Exposición del puerto `10051` para la comunicación con los agentes.
- **Volúmenes**:
  - `/etc/localtime` y `/etc/timezone` para sincronizar la zona horaria.
  - `./zbx_env/usr/lib/zabbix/alertscripts`, `./zbx_env/usr/lib/zabbix/externalscripts`, `./zbx_env/var/lib/zabbix/export`, `./zbx_env/var/lib/zabbix/modules`, `./zbx_env/var/lib/zabbix/enc`, `./zbx_env/var/lib/zabbix/ssh_keys`, `./zbx_env/var/lib/zabbix/mibs`, `./zbx_env/var/lib/zabbix/snmptraps` para scripts y configuraciones personalizadas.
- **Variables de Entorno**:
  - `POSTGRES_USER=zabbix`
  - `POSTGRES_PASSWORD=zabbix`
  - `POSTGRES_DB=zabbixNew`
  - `ZBX_HISTORYSTORAGETYPES=log,text`
  - `ZBX_DEBUGLEVEL=1`
  - `ZBX_HOUSEKEEPINGFREQUENCY=1`
  - `ZBX_MAXHOUSEKEEPERDELETE=5000`
  - `ZBX_PROXYCONFIGFREQUENCY=3600`
- **Dependencias**: Requiere que el servicio `postgres-server` esté disponible.
- **Reinicios**: Configurado para reiniciar a menos que se detenga explícitamente.

### 2. **Zabbix Web (Nginx)**
- **Imagen**: `zabbix/zabbix-web-nginx-pgsql:alpine-7.0-latest`
- **Puertos**: Exposición de los puertos `9080` y `8443` para el acceso web.
- **Volúmenes**:
  - `/etc/localtime` y `/etc/timezone` para sincronizar la zona horaria.
  - `./zbx_env/etc/ssl/nginx` para configuraciones SSL personalizadas.
  - `./zbx_env/usr/share/zabbix/modules/` para módulos adicionales.
- **Healthcheck**: Verifica la disponibilidad del servicio web cada 10 segundos.
- **Sysctls**: Configura `net.core.somaxconn=65535` para mejorar el rendimiento de la red.
- **Variables de Entorno**:
  - `POSTGRES_USER=zabbix`
  - `POSTGRES_PASSWORD=zabbix`
  - `POSTGRES_DB=zabbixNew`
  - `ZBX_SERVER_HOST=server`
  - `ZBX_POSTMAXSIZE=64M`
  - `PHP_TZ=America/Argentina/Buenos_Aires`
  - `ZBX_MAXEXECUTIONTIME=500`
- **Dependencias**: Requiere que los servicios `server` y `postgres-server` estén disponibles.

### 3. **Zabbix Agent**
- **Imagen**: `zabbix/zabbix-agent2:alpine-7.0-latest`
- **Puertos**: Exposición del puerto `10050` para la comunicación con el servidor.
- **Volúmenes**:
  - `/etc/localtime` y `/etc/timezone` para sincronizar la zona horaria.
  - `./zbx_env/etc/zabbix/zabbix_agentd.d` para configuraciones personalizadas del agente.
  - `./zbx_env/var/lib/zabbix/modules`, `./zbx_env/var/lib/zabbix/enc`, `./zbx_env/var/lib/zabbix/ssh_keys` para scripts y configuraciones adicionales.
- **Privilegios**: Ejecuta el contenedor con privilegios elevados (`privileged: true`) y utiliza el PID del host (`pid: "host"`).
- **Variables de Entorno**:
  - `ZBX_SERVER_HOST=server`
- **Dependencias**: Requiere que el servicio `server` esté disponible.

### 4. **Zabbix SNMP Traps**
- **Imagen**: `zabbix/zabbix-snmptraps:alpine-7.0-latest`
- **Puertos**: Exposición del puerto `162` para recibir trampas SNMP.
- **Volúmenes**: `./snmptraps:/var/lib/zabbix/snmptraps` para almacenar las trampas recibidas.
- **Variables de Entorno**:
  - `ZBX_SERVER_HOST=server`
- **Dependencias**: Requiere que el servicio `server` esté disponible.

### 5. **PostgreSQL Server**
- **Imagen**: `postgres:16-alpine`
- **Volúmenes**: `./zbx_env/var/lib/postgresql/data` para persistir los datos de la base de datos.
- **Variables de Entorno**:
  - `POSTGRES_PASSWORD=zabbix`
  - `POSTGRES_USER=zabbix`
  - `POSTGRES_DB=zabbixNew`
- **Healthcheck**: Verifica la disponibilidad del servicio de PostgreSQL cada 10 segundos.

## Instrucciones para Ejecutar

1. **Clonar el Repositorio**:

```bash
git clone https://url.git
```

2. **Acceder al Directorio del Proyecto**:
```bash
cd proyecto
```

3. **Crear y Iniciar los Servicios**:
```bash
docker compose up -d
```

4. **Verificar el Estado de los Servicios**:

```bash
docker compose ps
```

5. **Acceder a la Interfaz Web de Zabbix**:
   - URL: `http://localhost:9080` o `https://localhost:8443` (si tienes SSL configurado)
   - Usuario: `Admin`
   - Contraseña: `zabbix`

## Configuración Adicional

Para personalizar aún más tu entorno, puedes editar los volúmenes y variables de entorno según tus necesidades específicas.

**Nota**: El comando `docker compose` genera dos carpetas: `snmptraps` y `zbx_env`, que son propiedad de `sudo`. Recomiendo copiar la estructura para tener permisos sobre ella y volver a levantar el contenedor.

### Configuración del Agente Zabbix

1. **Crear o editar el archivo de configuración del agente**:
   ```conf
   Server=server
   ServerActive=server:10051
   Hostname=Zabbix_server
   ListenPort=10050
   ```
   Este archivo debe estar en `./zbx_env/etc/zabbix/zabbix_agentd.d/zabbix_agentd.conf`.

2. **Conectar el Agente al Servidor Zabbix**:
   - Inspecciona el contenedor del agente para obtener su IP:
     ```bash
     docker inspect id_contenedor
     ```
   - Actualiza la configuración del host en la interfaz web de Zabbix con la IP del contenedor.

3. **Actualizar el Caché del Servidor Zabbix**:
   ```bash
   docker exec -it idDelContenedorZabbixServer zabbix_server -R config_cache_reload
   ```

¡Listo! Ahora deberías tener todo funcionando correctamente.

---

**Créditos a [José María Labarta](https://www.youtube.com/@josemarialabarta)**

---

**Estructura del Proyecto**

Este proyecto utiliza las siguientes carpetas y archivos:

- `docker-compose.yml`: Define los servicios y configuraciones de Docker.
- `env_vars/`: Contiene archivos de configuración para las variables de entorno.
- `snmptraps/`: Almacena las trampas SNMP recibidas.
- `zbx_env/`: Contiene volúmenes para configuraciones y datos de Zabbix.

---
