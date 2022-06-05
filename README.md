# SISTEMAS INFORMÁTICOS - DAW1

## DESPLIEGUE DE APLICACIÓN USANDO DOCKER COMPOSE

### Josep Maria Castell Colom, Mateu Santandreu Truyols y Rafael Ivorra Llodrá

---

## Introducción

Esta práctica consiste en desplegar nuestro proyecto de final de curso usando Docker y, concretamente, Docker Compose.

Docker-compose es una herramienta que ayuda a administrar de forma centralizada el despliegue de varios contenedores Docker distintos.

## Pre-requisitos e instalación de Docker Compose [¹]

Para poder utilizar Docker Compose es imprescindible tener instalado [Docker Engine](https://docs.docker.com/engine/install/) en el sistema.  

Para instalar Docker Compose vamos a descargar el binario desde su [repositorio oficial](https://github.com/docker/compose).

Con el siguiente comando descargaremos la version `1.29.2`y guardaremos el ejecutable en `/usr/local/bin/docker-compose`, que sera accesible con el comando `docker-compose`:

```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Acto seguido vamos a conceder los permisos de ejecución: 

```
$ sudo chmod +x /usr/local/bin/docker-compose
```

Para verificar la instalación vamos a ejecutar:

```
$ docker-compose --version
```

Y deberíamos obtener un resultado similar al siguiente: 

```
docker-compose version 1.29.2, build 5becea4c
```

## Configuración del archivo `docker-compose.yml` [²]

El archivo `docker-compose.yml` es el archivo que usaremos para configurar nuestro set de contenedores. En él debemos incluir las imágenes que necesita nuestro proyecto y la configuración de cada una de ellas.  
En nuestro caso vamos a usar Nginx, Tomcat y MySQL.

Cada una de estas imágenes incluirá el nombre que le otorgaremos y los parámetros de configuración, los cuales vamos a detallar a continuación:

- `image`: Es el nombre de la imagen que Docker buscará para montar en el contenedor. Si lo deseamos podemos incluir la versión en concreto que necesitemos añadiendo dos puntos (`:`) y el número de versión. Por ejemplo: `mysql:8.0.29`.  
Si no incluimos ninguna versión, Docker buscará la más nueva (`latest`).

- `volumes`: Son los diferentes volumenes que queremos montar dentro del contenedor. Se compone de dos rutas separadas por dos puntos (`:`) donde la primera hace referencia a la ruta en el host y la segunda hace referencia al contenedor. Por ejemplo: `./opt/test:/var/lib/mysql`.

- `environment`: Son variables de entorno necesarias para el correcto funcionamiento del contenedor. Son pares `clave:valor`. Por ejemplo: `MYSQL_USER: testuser`.

- `ports`: Se trata del mapeo de los puertos, se indica escribiendo en primer lugar el puerto del host, dos puntos (`:`) y el puerto interno del contenedor. Las peticiones que lleguen al puerto del host serán reenviadas al puerto del contenedor, de forma similar a un router.

- `depends_on`: Este parámetro indica el orden en que seran cargados los contenedores. De forma que el contenedor del qual depende otro contenedor será cargado primero.

Cabe comentar que la primera línea del archivo `docker-compose.yml` será utilizada para definir la versión del mismo.

Dicho esto, veamos cómo quedaria nuestro `docker-compose.yml`:

```docker
version: '3.3'
services:
  db:
    image: mysql:8.0.29
    volumes:
      - /opt/test:/var/lib/mysql
      - ./back/db:/docker-entrypoint-initdb.d
    environment:
      MYSQL_ROOT_PASSWORD: **********
      MYSQL_DATABASE: alimentacion
      MYSQL_USER: sic
      MYSQL_PASSWORD: *******
    ports:
      - "3306:3306"
  backend:
    depends_on:
      - db
    image: tomcat:9.0
    ports:
      - "8080:8080"
    volumes:
      - ./back/app/target/app.war:/usr/local/tomcat/webapps/app.war
  web:
    image: nginx
    ports:
      - "80:80"
    volumes: 
      - ./front:/usr/share/nginx/html

```

## Despliegue de la aplicación [²]

Para arrancar todos los contenedores vamos a usar el comando:

```
docker-compose up -d
```

Este comando buscara por defecto el archivo `docker-compose.yml` en el mismo directorio. Si éste tuviera un nombre o una ruta diferentes habria que especificarlo usando la bandera `-f` (`file`) e indicando a continuación el nombre o la ruta específicos. La bandera `-d` usará el modo `detached` es decir, lo arrancará en segundo plano.

Si quisiéramos detener los contenedores usariamos del mismo modo el comando:

```
docker-compose down
```

## Subir las imágenes a nuestro repositorio de Docker Hub [³]

Para poder subir nuestras imágenes a [Docker Hub](https://hub.docker.com/) debemos tener un usuario registrado y hacer login desde nuestra terminal.  
Esto se consigue mediante el comando: 

```
docker login
```

Si es la primera vez, nos pedirá nuestras credenciales de Docker Hub; nombre de usuario y contraseña.
Si el resultado es satisfactorio nos informará mediante el output:

```
Login Succeeded
```

A continuación, vamos a listar las imágenes de las que disponemos localmente usando el comando:

```
docker images
```

De las imágenes que tenemos vamos a seleccionar las que queremos subir al repositorio remoto y les vamos a asignar una etiqueta (con el comando `docker tag`) conteniendo `nombre_de_usuario/nombre_imagen`. Por ejemplo, en nuestro caso:

```
docker tag tomcat:9.0 josepcastellcolom/sic.tomcat
```

Así con cada una de las imágenes que deseamos subir y, a continuación, usaremos el comando `docker push`, seguido de del nombre que le hemos dado, de la siguiente forma:

```
docker push josepcastellcolom/sic.tomcat
```

Ésto iniciará un proceso de subida de la imágen y nos mostrará una salida de consola similar a la siguiente:

```
Using default tag: latest
The push refers to repository [docker.io/josepcastellcolom/sic.tomcat]
3d9d2f8c01ed: Mounted from library/tomcat 
57969ac33e77: Mounted from library/tomcat 
f0bbc1d8b6f7: Mounted from library/tomcat 
e5ce43743a3d: Mounted from library/tomcat 
d744b7303bde: Mounted from library/tomcat 
817e710a8d04: Mounted from library/tomcat 
ee509ed6e976: Mounted from library/tomcat 
9177197c67d0: Mounted from library/tomcat 
7dbadf2b9bd8: Mounted from library/tomcat 
e7597c345c2e: Mounted from library/tomcat 
latest: digest: sha256:c4f98c74c3f90bb9240b79d787b65a1b49e101ca2a47abb262ee6942fc88a661 size: 2422
```

En éste momento, si accedemos a nuestro perfil de [Docker Hub](https://hub.docker.com/u/josepcastellcolom) veremos nuestro repositorio actualizado con las imágenes recién subidas.

Si queremos descargar estas imágenes en otro equipo o compartirlas con alguien, es suficiente utilizar el comando que aparece en la imagen de nuestro repositorio, por ejemplo:

```
docker pull josepcastellcolom/sic.tomcat
```

## Anexos

1.  Repositorio donde encontrar las imágenes de la presente práctica:  
https://hub.docker.com/u/josepcastellcolom

2. Comandos de descarga de las tres imágenes:  
    ```
    docker pull josepcastellcolom/sic.tomcat
    ```
    ```
    docker pull josepcastellcolom/sic.mysql
    ```
    ```
    docker pull josepcastellcolom/sic.nginx
    ```

## Bibliografia

[¹] [How To Install and Use Docker Compose on Ubuntu 20.04 | DigitalOcean/Community by Tony Tran & Erika Heidi](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04)

[²] [Deploy a tomcat application using docker-compose | DevOps4Solutions by Nidhi Gupta](https://devops4solutions.com/deploy-a-tomcat-application-using-docker-compose/)

[³] [How to push a container image to a Docker Repo | Azure Tips and Tricks](https://www.youtube.com/watch?v=r_tGl4zF1ZQ)