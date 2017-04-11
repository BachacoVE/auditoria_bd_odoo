Manual para hacer funcionar el script de auditoria

#Instancias docker

## preparo las carpetas que serán montadas como volumenes

```
$ sudo mkdir /opt/odoo9_aud/
$ sudo mkdir /opt/odoo9_aud/addons
$ sudo mkdir /opt/odoo9_aud/config/
$ sudo mkdir /opt/odoo9_aud/log
$ sudo chown -R vijoin:vijoin /opt/odoo9_aud/addons
$ sudo chmod -R 755 /opt/odoo9_aud/
$ sudo chmod -R 777 /opt/odoo9_aud/log
```

Además aprovecho de crear un punto de montaje hacia /home en el contenedor

`$ sudo mkdir ~/Proyectos/auditoria_BD_Odoo` (en esa ruta tendo el repo)

## creo los contenederos de base de datos y de odoo

`$ sudo docker run -d -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo -p 5431:5432 --name db9_aud postgres`

Fíjate en el puerto asignado al contenedor db9_aud para acceder desde pgadmin --> `5431` 

`$ sudo docker run -v /opt/odoo9-aud/addons:/mnt/extra-addons -v /opt/odoo9-aud/log:/var/log/odoo -v /opt/odoo9-aud/config:/etc/odoo -v /home/vijoin/Dropbox-respaldo-10032017/Proyectos/auditoria_BD_Odoo:/home/aud -p 8068:8069 --name odoo9_aud --link db9_aud:db -t odoo:9`

Fíjate en el puerto asignado al contenedor odoo9_aud para acceder desde el navegador --> `8068` 

## Cargo el .conf y los addons

Para cargar el `openerp-server.conf` revisar el manual https://gist.github.com/vijoin/3ccb0df8997babb37603#un-cotill%C3%B3n-de-soluci%C3%B3n

Los addons se suben de la forma que se desee

se reinicia el contenedor

`$ sudo docker restart odoo9_aud`

## creo la bd 

ver imagen

## instalo algún modúlo

en mi caso instalo fleet (Flota Vehicular)

Creo un par de registros

# Asigno una contraseña al usuario postgres

Ingreso al contenedor de postgres con el usuario root

`sudo docker exec -it -u 0 db9_aud bash`

**Nota para usuarios no Dockerizados:** Si no estás usando Docker puedes empezar desde aquí

me cambio al usuario postgres

`su - postgres`

me conecto al servidor postgres

`psql`

Cambio la contraseña de postgres

`ALTER USER postgres WITH PASSWORD '123456';`


## Modifico el script en base a mis necesidades

`$ subl .../Scripts/script_auditoria_v2.4.py`

Modifico las variables de conexión

```
database='prueba'
host='172.17.0.3'
user='postgres'
password='123456'
```

Para obtener la IP del contenedor db9_aud ejecutamos el siguiente comando: 
`docker inspect db9_aud`

**Nota para usuarios no Dockerizados:** probablemente tu IP es `localhost`

# Entramos al contenedor de odoo y desde allí ejecutamos el script

`sudo docker exec -it -u 0 odoo9_aud bash`

`python /home/aud/Scripts/script_auditoria_v2.4.py`

# Verificamos que ahora tenemos 3 archivos nuevos

* audiBash.sh
* script_bd_auditoria.sql
* script_trigger_auditoria.sql

Para este ejercicio ignoraremos el archivo `audiBash.sh`

#Creamos la base de datos aud_prueba
Esta será nuestra base de datos de auditoría. Es necesario crearla fuera del Script (Revisar, puede mejorarse)

psql -h 172.17.0.2 -U postgres -c "CREATE DATABASE aud_prueba;"

#Ejecutamos el archivo de script_bd_auditoria.sql

Aún conectados al conectedor db9_aud, ejecutamos:

psql -h 172.17.0.2 -U postgres -f script_bd_auditoria.sql

(Revisar por qué da algunas errores de sintaxis en la version 9.6)

#Ejecutamos el archivo script_trigger_auditoria.sql

`psql -h 172.17.0.2 -U postgres -f script_trigger_auditoria.sql`


Y Listo, y ahora a probar!