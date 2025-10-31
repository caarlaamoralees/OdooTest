# Instalar Odoo en Render

Esta es una guia para instalar el CRM de Odoo en Render

## 1. Crear el proyecto
   Lo primero, hay que crear los ficheros requeridos para que el futuro servidor funcione correctamente, para ello hay que crear la siguiente extructura de proyectos.
   - extra_addons
       - dummy_module
            - init.py
            - __manifest__.py
       -  gitkeep
   - Dockerfile
  
## 2. Rellenar el fichero Dockerfile y __manifest__.py

En el interior de nuestro fichero Dockerfile, hay que introducir lo siguiente:
```
# Imagen base Odoo 17
FROM odoo:17
## (Opcional) módulos propios
COPY ./extra-addons /mnt/extra-addons
## Puerto HTTP de Odoo
EXPOSE 8069
## Puerto por defecto de PostgreSQL
ENV PGPORT=5432
## 1) Inicializa la BD indicada en $PGDATABASE si está vacía (stop-after-init)
## 2) Después arranca el servidor normalmente

## NOTA: usamos $PGDATABASE para que la inicialización vaya contra esa BD
## y --db-filter la fije para evitar que Odoo “coja” otra por error.
CMD ["bash","-lc", "\
echo '==> Checking/initializing DB $PGDATABASE' && \
odoo -d $PGDATABASE -i base --without-demo=all \
--db_host=$PGHOST --db_port=$PGPORT \
--db_user=$PGUSER --db_password=$PGPASSWORD \
--addons-path=/usr/lib/python3/dist-packages/odoo/addons,/mnt/extra-addons \
--stop-after-init || true; \
echo '==> Starting Odoo server' && \
odoo --db_host=$PGHOST --db_port=$PGPORT \
--db_user=$PGUSER --db_password=$PGPASSWORD \
--addons-path=/usr/lib/python3/dist-packages/odoo/addons,/mnt/extra-addons \
--db-filter=$PGDATABASE \
--dev=all"]
```
y dentro del fichero __manifest__.py lo siguiente:
```
{
    "name": "Dummy Module",
    "version": "1.0",
    "summary": "Módulo vacío de prueba",
    "installable": True,
}
```


## 3. Registrarse en render
  Una vez que hemos creado el repositorio de GitHub vamos a dirigirnos a [Render](https://render.com) y nos registramos con nuestra cuenta de GitHub
   
## 4. Crear web service
  Cuando estemos en el dashboard de Render, procedemos a crear un servidor que pueda alojar nuestro contenedor de Docker con Odoo, para ello creamos un nievo "Web service" y en el selector "Git Provider" donde veremos todos nuestros repositorios, seleccionamos el nombre del repo que acabamos de crear.
### * NOTA: antes de crearlo tenemos que asegurarnos de que:
  * El lenguaje esta configurado a "Docker"
  * El tipo de instancia (Instance type) esta seleccionado el "Free"

Una vez que nos aseguremos de esto, podemos roceder con la creacion del web service.

## 5. Crear una base de datos Postgres
  Con el web service creado, podemos proceder a crear la base de datos postgre que alojara nuestros datos de Odoo. Para ello, en el dashboard de Render, crearemos in nuevo Postgres asegurandonos de que el "Instance type" es free

## 6. Copiar datos de conexion
   Una vez que tenemos nuestra base de datos Postgres, debemos copiar los datos de acceso de nuestra base de datos para indicarle a nuestro web service como acceder, para ello debemos entrar en la base de datos en la web Render y bajar hasta el apartado "Connectiones" donde veremos:
   * Hostname
   * Port
   * Database
   * Password
     
Estos parametros tenemos que copiarlos para introducirlos en el siguiente paso como variables de entorno en nuestro web service.

## 7. Configurar variables de entorno en Web Service
   A continuacion, debemos entrar en nuestro web service, y en la barra lateral debemos seleccionar "Environment" y ahi, introducir la clave y el valor de los datos.
###  * IMPORTANTE: la clave de estos datos tenemos que copiarla de nuestro Dockerfile donde encontraremos valores como "$PORT" por ejemplo, esto se aplica para el resto de parametros, cada uno tiene su clave.
El valor lo debemos haber copiado en el paso anterior, por lo que ya solo nos quedaria pegar cada uno con su clave y ya habriamos finalizado los pasos para completar nustro contenedor de Docker con Odoo, listo para su uso. 
