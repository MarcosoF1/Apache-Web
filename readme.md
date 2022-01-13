# Servidor Web Apache

## Docker compose
version: "2.2"
services:
#### este es el dns
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
#### este es el cliente con la imagen kasmweb/desktop
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
#### y este es el servidor web apache
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