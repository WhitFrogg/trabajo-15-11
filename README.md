# trabajo-15-11
Este proyecto utiliza **Docker Compose** para levantar un contenedor con **MySQL** y otro con **PHP/Apache**, mostrando los nombres de los integrantes de la base de datos.
trabajo 1115/
│
├─ docker-compose.yml
├─ db/
│ └─ init.sql
└─ web/
├─ index.php
└─ Dockerfile
## Paso a Paso de la cconfigurcion

## 1. Crear la base de datos y tabla

Creamos un archivo `db/init.sql` con el contenido:

```sql
CREATE DATABASE IF NOT EXISTS integrantes;
USE integrantes;

CREATE TABLE IF NOT EXISTS integrantes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(255) NOT NULL
);

INSERT INTO integrantes (nombre) VALUES ('Benjamin Cerda'), ('Sebastian del Pino');

##Este archivo se ejecutará automáticamente cuando se levante el contenedor de MySQL

##2. Creamos un archivo docker-compose.yml en la raíz del proyecto:
services:
  db:
    image: mysql:8
    container_name: db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: integrantes
    volumes:
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

  web:
    build: ./web
    container_name: web
    ports:
      - "80:80"
    depends_on:
      db:
        condition: service_healthy

##db: contenedor de MySQL.
##web: contenedor de PHP/Apache.
##healthcheck: garantiza que MySQL esté listo antes de levantar el contenedor web.

##3.  Creamos un dockerfile dentro de la carpeta web/:
FROM php:8.2-apache


COPY index.php /var/www/html/


RUN docker-php-ext-install mysqli

##4. Ahora creamos el index.php:
<?php
$host = 'db';
$usuario = 'root';
$password = 'root';
$basededatos = 'integrantes';

$conexion = new mysqli($host, $usuario, $password, $basededatos);

if ($conexion->connect_error) {
    die("Conexión fallida: " . $conexion->connect_error);
}

$resultado = $conexion->query("SELECT nombre FROM integrantes");

echo "<h1>Integrantes</h1>";
while($fila = $resultado->fetch_assoc()){
    echo $fila['nombre'] . "<br>";
}

$conexion->close();
?>
##5. Ahora levantamos los contenedores:
docker-compose up --build

##6. Ahora verificamos el funcionamiento en el navegador:
http://localhost
<img width="232" height="130" alt="image" src="https://github.com/user-attachments/assets/7df207db-76ef-4b3c-a182-795891149687" />
