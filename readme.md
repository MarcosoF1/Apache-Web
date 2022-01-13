# Servidor Web Apache

## Docker compose
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
      red01:
        ipv4_address: 10.1.0.254
~~~
#### Este es el cliente con la imagen kasmweb/desktop
~~~
  asir_cliente:
    image: kasmweb/desktop:1.10.0-rolling
    ports:
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

- A continuación en en servidor apache en la carpeta htdocs creo dos carpetas cada una con un index, para hacer dos paginas web

- Y en el volumen de configuración del servidor apache buscamos esta linea: Include conf/extra/httpd-vhosts.conf y la descomentamos

- Y en la carpeta extra buscamos el archivo: httpd-vhosts.conf y creamos dos virtual host para cada pagina
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
~~~

### En el Servidor dns

- Creamos una zona dns como en la practica dns, y agregamos estos dos cnames:
~~~
hola IN CNAME ejemplo
adios IN CNAME ejemplo
~~~

- Reiniciamos los equipos

### En el cliente

- Vamos al navegador y hacemos dos busquedas para comprobar que funcione, primero busco hola.ejemplo.com y nos aparece el index que haya creado a la pagina hola

- y despues buscamos adios.ejemplo.com y nos aparecera otra pagina distinta

