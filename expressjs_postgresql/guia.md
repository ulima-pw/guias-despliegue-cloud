# Guía para despliegue de aplicaciones web utilizando ExpressJS (NodeJS) y Postgresql

## Requisitos

- Tener una cuenta github creada (https://github.com/).
- Tener una cuenta en el servicio de Heroku (https://heroku.com/).
- Tener GIT instalado en su máquina de desarrollo (https://git-scm.com/downloads).

## Instalación de Heroku Cl

Descargar la aplicación de línea de comandos Heroku-CLI, que nos permitirá
realizar las acciones de despliegue mediante comandos en un shell.

- https://devcenter.heroku.com/articles/heroku-cli

Si todo se encuentra bien configurado, al abrir un cmd podrá probar su instalación con el siguiente comando que le mostrará todas las opciones de heroku.

```
$ heroku --help
```

## Creación de aplicación en Heroku

Primero nos procedemos a loguear con Heroku.

```
$ heroku login
```

Luego procedemos a crear un nuevo proyecto.

```
$ heroku create
```

Notar que heroku ha creado un proyecto en la nube y le ha asignado un nombre aleatorio. Para cambiarle este nombre escribiremos por linea de
comandos lo siguiente:

```
$ heroku rename -a <nombre_app> <nuevo_nombre_app>
```

El **nuevo_nombre_app** deberá ser único si no Heroku no permitirá el uso de ese nombre.

## Preparación de proyecto

Debido a que el código fuente de nuestro proyecto será subido a la nube para ser construido ahí, tenemos que prepararlo para que Heroku pueda realizar la construcción y la ejecución sin problemas.

### Revisión de fuentes

- Revisar el directorio que queremos subir a la nube y que nos encontramos en este.
- Verificar si hay archivos/directorios que no queremos que se suban a la nube (ya que probablemente van a ser descargadas/construidas ahí): por ejemplo, dependencias, librerías, etc.

### Preparación de Procfile

Primero debemos de crear un archivo llamado Procfile que debe estar en el 
directorio raíz del cuál queremos desplegar. En este archivo se especificará el comando que Heroku deberá ejecutar luego de la construcción del proyecto.

Para nuestro caso:

```
web : npm start
```

Como se ve, se ejecutará el comando que pongamos en el package.json para el script start. Este comando deberá de ejecutar el app.js del proyecto.

Ejemplo del package.json

```
{
  "name": "catalogo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start" : "node ./server/app.js"
    "start:dev": "npx nodemon ./server/app.js"
  },
  "author": "Hernan Quintana",
  "license": "ISC",
  "dependencies": {
    "ejs": "^3.1.5",
    "express": "^4.17.1",
    "pg": "^8.5.1",
    "pg-hstore": "^2.3.3",
    "sequelize": "^6.5.0",
    "sequelize-cli": "^6.2.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.7",
    
  }
}

```

### Modificación del puerto

Al configurar el servidor web express en el app.js al inicio del proyecto se puso que el puerto en el cual se iba a ejecutar el servidor de desarrollo (local) iba a ser el 3000. Como ahora nuestra aplicación ya no va a correr de manera local, el propio Heroku va a asignarle un número de puerto variable. Este parámetro Heroku lo almacena en una variable de entorno de la máquina virtual que se nos asigna (llamada PORT).

Nuestro código en app.js deberá de modificarse de la siguiente manera:

```
const express = require('express');
const bodyParser = require('body-parser');

const app = express();
const PORT = process.env.PORT || 3000;

....

app.listen(PORT, () => {
    console.log(`Servidor iniciado en puerto ${PORT}`);
});
```

### Inicialización de repositorio GIT

Para poder enviar las fuentes de nuestro proyecto a Heroku se deberá hacerlo desde un proyecto GIT.

En caso que se haya creado el proyecto desde shell, Heroku automáticamente inicializa GIT en el directorio donde se ejecutó el create. Caso que la creación del proyecto se haya hecho por la web de heroku se deberá ejecutar el siguiente comando:

```
$ git init
```

Ademas deberá agregar el remote necesario que apunte al repositorio GIT en Heroku (la dirección de este repositorio lo sacará de la web).

```
$ git remote add heroku <ruta_repo_heroku>
```

**Nota:** No olvidarse de crear el archivo .gitignore donde especificará qué archivos y/o directorios no desea que se suban a la nube (por ejm directorio node_modules). Esto es muy importante para que su despliegue no sea demasiado lento.

## Despliegue de cambios

Luego de realizar cambios en el código fuente (y al inico también) y ya tener preparado todo para desplegar la aplicación a Heroku, deberá de posicionarse en la raíz de su proyecto (por shell) y ejecutar los siguientes comandos:"

```
$ git add .
$ git commit -m "<comentario_de_cambios>"
$ git push heroku master
```

Esto hará que sus cambios se suban a Heroku y Heroku despliegue un entorno donde descargará sus dependencias, construirá su proyecto y ejecutará su aplicación.

## Creación y configuración de base de datos Postgresql

Desde el sitio web de Heroku, agregar la base de datos Heroku Postgres. Luego de esto verificar las variables de configuración en la opción Settings de su aplicación (todo en el sitio web). En la opción Config Vars (variables de entorno) deberá de haberse agregado una llamada DATABASE_URL donde están los datos de conexión a la bd.

Con estos datos de conexión podrá configurar una conexión desde PgAdmin o desde shell también podrá conectarse mediante este comando:

```
$ heroku pg:psql <nombre_bd_postgresql> --app <nombre_app>
```

### Configuración para ORM Sequelize

Los datos de conexión a la base de datos se encuentran en el archivo server/dao/config/config.json. Los datos obtenidos de la variable de entorno DATABASE_URL configurarlo en el campo production (o test, dependiendo del uso del ambiente).Tener en cuenta que la variable de entorno NODE_ENV ya se encuentra seteada por defecto a production.

Para el caso de la base de datos postgresql que ofrece Heroku debe de configurar una nueva opción llamada dialectOptions, esto en el archjivo server/dao/config/config.json.

```
{
  "development": {
    "username": "catalogo",
    "password": "catalogo",
    "database": "catalogodb",
    "host": "127.0.0.1",
    "dialect": "postgres"
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "**************",
    "password": ""**************",",
    "database": ""**************",",
    "host": "ec2-18-204-74-74.compute-1.amazonaws.com",
    "dialect": "postgres",
    "dialectOptions": {
      "ssl": { "rejectUnauthorized": false }
    }
  }
}
```

### Ejecución de migrations y seeders

Para ejecutar las migraciones necesarias para crear la estructura de la base de datos, desde el shell local utilizaremos el comando heroku run.

```
$ heroku run "npx sequelize db:migrate --config ./server/dao/config/config.json --migrations-path ./server/dao/migrations"
```

Como se ve, en el parámetro --config indicamos la dirección del archivo config.json dentro de nuestro proyecto, y con la opción --migrations-path indicamos la ruta del directorio donde se encuentran nuestras migraciones.

Para ejecutar los seeders se procede una forma similar.

```
$ heroku run "npx sequelize db:seed:all --config ./server/dao/config/config.json --seeders-path ./server/dao/seeders
```

Donde nuevamente se especifica el archivo de configuración por el --config y el directorio donde se encuentran los seeders en el --seeders-path.




