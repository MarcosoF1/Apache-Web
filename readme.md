# Servidor Web Apache

## Docker Compose
~~~
version: "2.2"
services:
~~~
#### Este es el contenedor del dns
~~~
  asir_bind9:
    image: internetsystemsconsortium/bind9:9.16
    ports:
      -53:53
    volumes:
      -dns_conf:/etc/bind
    networks:
      red01: Aquí está la IP que tendrá en la red creada: red01
        ipv4_address: 10.1.0.254
~~~
#### Este es el cliente con la imagen kasmweb/desktop
~~~
  asir_cliente:
    image: kasmweb/desktop:1.10.0-rolling
    ports: Aquí tiene vinculado dos puertos
      -6901:6901
~~~
#### El apartado environment nos sirve para ponerme la contraseña al usuario del contenedor
~~~
    environment:
      VNC_PW: password
    networks:
      -red01
    dns:
      -10.1.0.254
    stdin_open: true  # docker run -i
    tty: true         # docker run -t
~~~
#### Este es el servidor web apache
~~~
  asir_webb:
    image: httpd:latest
    ports:
      -"80:80"
    networks:
      red01:
        ipv4_address: 10.1.0.5
    volumes:
      -apache_index:/usr/local/apache2/htdocs
      -apache_conf:/usr/local/apache2/conf
~~~
#### Aqui creo el contenedor del wireshark 
~~~
asir_wireshark:
    image: linuxserver/wireshark
    cap_add:  
      - NET_ADMIN
    network_mode: host  
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    ports:
      - 3000:3000 #optional
~~~
- Para acceder a wireshark desde el navegador ponemos localhost:3000 y accederemos a el
#### Aqui, al final del compose ponemos las redes y volumenes que tenemos, y el external lo ponemos por que ya estan creados
~~~
networks:
 red01:
  external: true
volumes:
  dns_conf:
    external: true
  apache_conf:
    external: true
  apache_index:
    external: true
~~~

## Configuración maquinas   

### En el servidor web

- A continuación en en servidor apache en la carpeta htdocs creo dos carpetas cada una con un index, para hacer dos paginas web.

- En el volumen de configuración del servidor apache buscamos esta linea: Include conf/extra/httpd-vhosts.conf y la descomentamos.

- En la carpeta extra buscamos el archivo: httpd-vhosts.conf y creamos dos virtual host para cada pagina.
~~~
<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host.example.com
    DocumentRoot "/usr/local/apache2/htdocs/adios"
    ServerName adios.ejemplo.com
    ErrorLog "logs/dummy-host.example.com-error_log"
    CustomLog "logs/dummy-host.example.com-access_log" common
</VirtualHost>

<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host2.example.com
    DocumentRoot "/usr/local/apache2/htdocs/hola"
    ServerName hola.ejemplo.com
    ErrorLog "logs/dummy-host2.example.com-error_log"
    CustomLog "logs/dummy-host2.example.com-access_log" common
</VirtualHost>

 - ServerAdmin: es un contacto al administrador o otra persona
 - DocumentRoot: es la ruta donde estara la pagina web
 - ServerName: Es el nombre de dominio del virtual host
 - ErrorLog: es donde se van a almacenar los logs de errores
 - CustomLog: Es donde se va a almacenar los logs de acceso
~~~

### En el Servidor dns

- Creamos una zona dns como en la practica dns, y agregamos estos dos cnames:
~~~
hola IN CNAME ejemplo
adios IN CNAME ejemplo
~~~

- Reiniciamos los equipos

### En el cliente

- Vamos al navegador y hacemos dos busquedas para comprobar que funcione, primero busco hola.ejemplo.com y nos aparece el index que haya creado a la pagina hola.

- Despues buscamos adios.ejemplo.com y nos aparecera otra pagina distinta.


## Protocolo HTTPS

~~~
Primero realizo el comando para crear las claves y si queremos que se creen los archivos en una ruta que qreramos le cambiamos el path y estaria:
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt


Cuando se haya ejecutado, utilizamos el comando cp .key o .crt contenedor:ruta volumen
Ejemplo:

sudo docker cp apache-selfsigned.key asir_webb:/usr/local/apache2/conf

docker cp apache-selfsigned.crt asir_webb:/usr/local/apache2/conf

Despues, vamos a inspeccionar el volumen y tenemos que entrar en el archivo httpd-ssl.conf y en required modules vamos viendo copiando y viendo en httpd.conf las lineas que tenemos que descomentar para que nos funcione el htpps.

~~~

## Wireshark


~~~
Con esto en el docker compose creamos el contenedor de wireshark y para acceeder a el en el navegador ponemos localhost:3000 y ya podremos escanear los paquetes de la red.
La configuracion que tengo se consigue en la propia pagina de la imagen.
asir_wireshark:
    image: linuxserver/wireshark
    cap_add:  
      - NET_ADMIN
    network_mode: host  
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    ports:
      - 3000:3000 #optional

Ahora podremos ccaptar paquetes dns, http y https y tambien podemos comprobar que los http van en texto plano y los https cifrados
~~~
