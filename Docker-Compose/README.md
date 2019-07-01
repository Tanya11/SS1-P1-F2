# Docker-Compose
Un pequeno ejemplo de como utilizar docker-compose


### Pre-requisitos 📋
Tener instalado Docker.
Si no podemos instalarlo de esta forma.
Actualizar los paquetes.
```
sudo apt-get update
```
Seguido:
```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```
Despues:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
Agregar la key:
```
sudo apt-key fingerprint 0EBFCD88
```
Al obtener las llaves, veran algo de este estilo:
```
pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
```
Actualizar los paquetes.
```
sudo apt-get update
```
instalar Docker:
```
sudo apt-get install -y docker.io
```
Por ultimo docker-compose:
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose
```

## Comenzando 🚀
Para empezar creamos el archivo docker-compose.yml
```
nano docker-compose.yml
```
Script .yml
```
version: '3'
services:
  web:
    build: .
    volumes:
      - .:/app/
      - /app/node_modules
    ports:
      - "3000:3000"
    tty: true
    networks:
      app_subnet:
        ipv4_address: 172.16.1.4
  db:
    image: mysql:latest
    restart: always
    environment:
      MYSQL_DATABASE: 'db_p1'
      # So you don't have to use root, but you can if you like
      MYSQL_USER: 'root'
      # You can use whatever password you like
      MYSQL_PASSWORD: '1234'
      # Password for root access
      MYSQL_ROOT_PASSWORD: '1234'
    ports:
      # <Port exposed> : < MySQL Port running inside container>
      - '3306:3306'
    expose:
      # Opens port 3306 on the container
      - '3306'
      # Where our data will be persisted
    volumes:
      - my-db:/var/lib/mysql
    networks:
      app_subnet:
        ipv4_address: 172.16.1.3
# Names our volume
volumes:
  my-db:

networks:
  app_subnet:
    external: true

```
Tambien creamos los siguientes archivos para este ejemplo:
### index.js
```
'use strict';
const express = require('express');
// App
const app = express();
var ip = process.env.IP || '127.0.0.1:3306';
var h = process.env.HOST ||'0.0.0.0';
// Constants
const PORT = 3000;
const HOST = h;

var body_parser = require('body-parser').json();

const mysql = require('mysql');
// connection configurations
const mc = mysql.createConnection({
    host:  ip,
    user: 'root',
    password: '1234',
    database: 'db_p1'
});
mc.connect();



app.get('/viewAlumno', (req, res) => {
	mc.query("Select * from Alumno",function (err, result, fields) {
    if (err) {throw err;}
    else{
    	res.send(result);	
	}
	});
});


app.post('/insertAlumno',body_parser, function (req, res) {

	var dpi = req.body.dpi || '';
    var carnet = req.body.carnet || '';
    var nombre = req.body.nombre || '';
    var apellido = req.body.apellido || '';
    var email = req.body.email || '';
    var telefono = req.body.telefono || ''; 

	var query = 'insert into Alumno(carnet,dpi,nombre,apellido,email,telefono) values('+carnet+','+dpi+',"'+nombre+'","'+apellido+'","'+email+'","'+telefono+'");'
  	  mc.query(query, function (err, result) {
    if (err){res.send("FAIL!!!!");throw err;}
    else{
    	res.send("SUCCESS");}
  });
})


app.listen(PORT,HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```
### Dockerfile
```
FROM node
WORKDIR /app
ADD . /app
COPY package.json .
RUN npm install --quiet
ENV PORT 3000
ENV IP "172.16.1.4"

```
### package.json
```
{
  "name": "app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "author": "Andree",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1",
    "mysql": "^2.17.1"
  }
}
```
Creamos una subred donde operara nuestro contenedor
```
docker network create --gateway 172.16.1.1 --subnet 172.16.1.0/24 app_subnet
```

Una vez hecho lo anterior, nos dirigimos hacia la carpeta donde creamos el .yml y ejecutamos el siguiente comando
```
docker-compose up
```
para confirmar que se hayan creado los contenedores
```
docker ps 
```

## Ejecutando las pruebas ⚙️

1. Ingresamos al contenedor 
```
docker exec -it api_db_1 bash
```
2. Ingresamos a mysql
```
mysql -u root -p 1234
```
3. Fijamos como determinada a la base de datos creada
```
use db_p1;
```
4. Creamos una tabla y un registro, para la prueba 
```
CREATE TABLE Alumno (
carnet int,
dpi bigint,
nombre varchar(25),
apellido  varchar(25),
email  varchar(50),
telefono varchar(8)
);
--------------------------
insert into Alumno(carnet,dpi,nombre,apellido,email,telefono)values(201408580,29888879212,"Andree","Avalos","prueba@gmail.com","55555555");

```
5. Me dio un error a la hora de conectarse con la base de datos, pero fue por no tener las credenciales necesarias.
```
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '1234';
```
con eso lo solucione.

6. salimos con
```
exit
```
7. Ingresamos a la API
```
docker exec -it api_web_1 bash
```
8. Nos dirigimos donde se encuentra el archivo index.js y lo ejecutamos
```
HOST=172.16.1.4 IP=api_db_1 node index.js -e
```
* **HOST** Es una variable de entorno que posiciona el host de la api
* **IP** Variable de entorno para indicar la direccion ip de la base de datos
* **api_db_1** Al estar en el mismo contenedor, sabe inmediatamente a que ip se refiere.
* **-e** indica que existen variables de entorno en el comando

9. Para probar el funcionamiento correcto de la api
```
curl http://172.16.1.4/viewAlumno
```
10. Nos desplegara un json

## Deployment 📦
Para poder subir la imagen a Docker hub
1. Agregamos un tag
```
docker tag nombreimagen:tag user/server:tagname
```
2. Damos push
```
docker push user/server:tagname
```
Example:
```
docker tag server:latest andreeavalos/server:01
docker push andreeavalos/server:01
```
3. Para poder bajar el repositorio 
```
docker pull andreeavalos/server
```

## Construido con 🛠️
* [Docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/) - Ambiente de trabajo
* [Node JS](https://nodejs.org/es/docs/)- Framework
* [MYSQL](https://dev.mysql.com/doc/)- MYSQL

## Guias  🖇️
* [Medium](https://medium.com/@chrischuck35/how-to-create-a-mysql-instance-with-docker-compose-1598f3cc1bee)- GUIA para crear docker-compose.yml

## Solucion de Errores 📌
* [Stackoverflow](https://stackoverflow.com/questions/50093144/mysql-8-0-client-does-not-support-authentication-protocol-requested-by-server) - Solucion Autentificacion


## Autores ✒️

* *Carlos Andree Avalos Soto*-*Trabajo Inicial*-[andreeavalos](https://github.com/andreeavalos)
