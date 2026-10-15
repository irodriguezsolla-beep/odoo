# Instalación de Odoo
#### Por: Alan, Adrián e Isaac

## Preparación del archivo .yaml
![Captura desde 2026-10-15 11-08-51.png](Fotos/Captura%20desde%202026-10-15%2011-08-51.png)

Para definir la infraestructura de la aplicación mediante **Docker Compose**, se crea el archivo `compose.yaml` (o `docker-compose.yml`) ejecutando el comando `touch compose.yaml`. A continuación, se detalla la función de cada bloque del archivo:

- **version**: Especifica la versión del formato de sintaxis de Docker Compose utilizada.
- **services**: Agrupa los contenedores independientes que componen la arquitectura de la aplicación:
  - **db** (Base de datos PostgreSQL):
    - **image**: Descarga e instancia la imagen oficial de PostgreSQL v15 desde Docker Hub.
    - **environment**: Define las variables de entorno para inicializar el gestor de base de datos:
      - `POSTGRES_DB`: Nombre de la base de datos inicial creada automáticamente.
      - `POSTGRES_USER`: Usuario administrador para la base de datos.
      - `POSTGRES_PASSWORD`: Contraseña de acceso para dicho usuario.
    - **volumes**: Asocia un volumen persistente gestionado por Docker con la ruta interna `/var/lib/postgresql/data`, garantizando que la información no se pierda al reiniciar o eliminar los contenedores.
  - **web** (Servidor de aplicaciones Odoo):
    - **image**: Imagen oficial de Odoo en su versión 18.
    - **ports**: Redirecciona los puertos entre el equipo host y el contenedor (`"puerto_host:puerto_contenedor"`). Permite acceder a la interfaz web mediante el puerto `8069`.
    - **environment**: Establece los parámetros de conexión de Odoo con PostgreSQL:
      - `HOST`: Nombre del servicio de base de datos (`db`), que Docker resuelve internamente a nivel de red.
      - `USER`: Usuario con permisos de conexión a PostgreSQL.
      - `PASSWORD`: Contraseña de autenticación de la base de datos.
    - **depends_on**: Establece un orden de arranque, garantizando que el contenedor `db` se inicie antes que el contenedor `web`.
    - **volumes**: Conecta un volumen persistente con la carpeta interna de Odoo (`/var/lib/odoo`) para conservar los archivos adjuntos, sesiones y configuraciones del sistema.
- **volumes**: Declara los volúmenes globales (`odoo-db-data` y `odoo-web-data`) gestionados por el motor de Docker para la persistencia permanente de datos.

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=triade
      - POSTGRES_PASSWORD=alanguapo
    volumes:
      - odoo-db-data:/var/lib/postgresql/data

  web:
    image: odoo:18
    ports:
      - "8069:8069"
    environment:
      - HOST=db
      - USER=triade
      - PASSWORD=alanguapo
    depends_on:
      - db
    volumes:
      - odoo-web-data:/var/lib/odoo

volumes:
  odoo-db-data:
  odoo-web-data:
```

## Instalación e inicio del sistema

![Captura desde 2026-10-08 11-19-45.png](Fotos/Captura%20desde%202026-10-08%2011-19-45.png)

Para desplegar la pila de servicios definida en la misma carpeta del archivo, se ejecuta el siguiente comando:

```bash
docker compose up -d
```

- `docker compose up`: Lee la configuración de `compose.yaml`, descarga las imágenes necesarias si no están presentes en el equipo y crea/inicia los contenedores.
- `-d` (*detached mode*): Ejecuta los contenedores en segundo plano, liberando la terminal para seguir interactuando con el sistema operativo.

## Configuración inicial de Odoo desde la web

Para completar la instalación, abre un navegador web e ingresa en la barra de direcciones la URL con la IP de la máquina y el puerto expuesto: `http://<IP_DEL_EQUIPO>:8069` (o `http://localhost:8069` si se ejecuta localmente).

En la primera visita se mostrará el formulario de configuración inicial de Odoo donde se deben definir:
1. **Master Password**: Clave maestra para administrar, crear, respaldar o eliminar bases de datos en Odoo.
2. **Database Name**: Nombre asignado a la base de datos de trabajo.
3. **Email / Password**: Credenciales de acceso para la cuenta de usuario administrador principal.
4. **Language & Country**: Idioma predeterminado del sistema y la localización del país.

![Captura desde 2026-10-08 11-23-28.png](Fotos/Captura%20desde%202026-10-08%2011-23-28.png)
![Captura desde 2026-10-08 11-25-57.png](Fotos/Captura%20desde%202026-10-08%2011-25-57.png)

## Sistema desplegado correctamente

Tras procesar la creación de la base de datos, el sistema redirigirá al panel principal (*Dashboard*) de Odoo. Desde este panel se pueden activar e instalar los módulos requeridos (CRM, Ventas, Inventario, Facturación, etc.).

![Captura desde 2026-10-14 14-31-29.png](Fotos/Captura%20desde%202026-10-14%2014-31-29.png)

