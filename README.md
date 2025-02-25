# Instalación de Zabbix con Docker

=====================================

Este proyecto utiliza Docker para instalar y configurar Zabbix, una herramienta de monitoreo de redes y sistemas. La configuración ha sido diseñada en base a lo propuesto por (**José María Labarta**)[https://www.youtube.com/@josemarialabarta]

## Copyright

---

Este proyecto está basado en la configuración original de **José María Labarta**. Este documento es una adaptación para fines educativos y de referencia.

## Requisitos Previos

---

- Docker instalado en tu sistema.
- Conocimientos básicos sobre Docker y Zabbix.

## Componentes del Proyecto

---

Este proyecto utiliza varios contenedores Docker para desplegar una instalación completa de Zabbix:

- **Zabbix Server**: Utiliza la imagen `zabbix/zabbix-server-pgsql:alpine-7.0-latest`.
- **Zabbix Web**: Utiliza la imagen `zabbix/zabbix-web-nginx-pgsql:alpine-7.0-latest`.
- **Zabbix Agent**: Utiliza la imagen `zabbix/zabbix-agent2:alpine-7.0-latest`.
- **Zabbix SNMP Traps**: Utiliza la imagen `zabbix/zabbix-snmptraps:alpine-7.0-latest`.
- **PostgreSQL Server**: Utiliza la imagen `postgres:16-alpine`.

## Configuración

---

La configuración se realiza mediante un archivo `docker-compose.yml`. Este archivo define los servicios, puertos, volúmenes y variables de entorno necesarias para cada contenedor.

### Servicios

#### Zabbix Server

- **Puertos**: Exposición del puerto `10051`.
- **Volúmenes**: Mapeo de directorios locales para scripts de alertas, exportaciones, módulos, claves SSH, MIBs y trampas SNMP.
- **Dependencias**: Depende del servicio `postgres-server`.
- **Variables de Entorno**: Configuración de PostgreSQL y parámetros de Zabbix.

#### Zabbix Web

- **Puertos**: Exposición de los puertos `8080` y `8443`.
- **Volúmenes**: Mapeo de directorios para certificados SSL y módulos.
- **Dependencias**: Depende de los servicios `server` y `postgres-server`.
- **Variables de Entorno**: Configuración de PostgreSQL, Zabbix Server y parámetros de PHP.

#### Zabbix Agent

- **Puertos**: Exposición del puerto `10050`.
- **Volúmenes**: Mapeo de directorios para configuraciones del agente y módulos.
- **Dependencias**: Depende del servicio `server`.
- **Variables de Entorno**: Configuración del host del servidor Zabbix.

#### Zabbix SNMP Traps

- **Puertos**: Exposición del puerto `162/udp`.
- **Volúmenes**: Mapeo de directorios para trampas SNMP.
- **Dependencias**: Depende del servicio `server`.
- **Variables de Entorno**: Configuración del host del servidor Zabbix.

#### PostgreSQL Server

- **Volúmenes**: Persistencia de datos en un directorio local.
- **Variables de Entorno**: Configuración de la base de datos y credenciales.

## Instrucciones de Uso

---

1. **Clonar el Repositorio**: Descarga este proyecto a tu máquina local.

2. **Crear Volúmenes**: Asegúrate de que los directorios locales mapeados en el archivo `docker-compose.yml` existan y estén correctamente configurados.

3. **Iniciar Servicios**: Ejecuta el comando `docker-compose up -d` para iniciar todos los servicios en segundo plano.

4. **Acceder a Zabbix Web**: Abre un navegador y accede a `http://localhost:8080` o `https://localhost:8443` para configurar Zabbix.

5. **Configurar Zabbix**: Sigue las instrucciones en la interfaz web para completar la configuración inicial.

## Problemas Comunes

---

- **Puertos Ocupados**: Asegúrate de que los puertos configurados no estén en uso por otros servicios.
- **Errores de Conexión**: Verifica que las credenciales de PostgreSQL sean correctas y que el servicio esté disponible.

## Contribuciones

---

Si encuentras algún error o tienes mejoras, por favor, contribuye al proyecto abriendo una issue o enviando un pull request.

---

Espero que esta guía te sea útil. Recuerda adaptar los nombres y detalles según sea necesario para tu proyecto específico.
