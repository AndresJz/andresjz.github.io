---
title: Proxy reverso con autenticación externa usando nginx
layout: post
author: luisjuarez
thumbnail: "/assets/img/posts/hello.jpg"
category: ops
summary: 
keywords: nginx proxy
permalink: "/blog/nginx-reverse-proxy"
---


En este pequeño tutorial se muestra como crear un "reverse proxy" utilizando nginx, para que sea mucho más fácil observar el concepto las coniguraciones se realizaron utilizando docker y docker-compose. 

Para realizar el proxy utrilizaremos Nginx el cual es un servidor ligero y de código abierto que nos permite de forma sencilla realizar esta operación 

El primer paso es crear un contenedor de nginx utilizando la version stable 

Además configuraremos un volumen donde definiremos el archivo de configuración que se utilizará para el redireccionamiento del tráfico 

```sh
nginx: 
    image: nginx:stable 
    container_name: reverse_proxy 
    volumes: 
        - ./reverse_proxy.conf:/etc/nginx/nginx.conf 
    ports: 
        - 8888:80 
```

El archivo de configuración consta de algunas secciones importantes que se mostraran a continuación 

Contexto Principal 
Primero se definen los valores globales, en este punto no hablaremos a profundida de ellos, por ahora puedes utilizar esta configuración directamente o utilizar los valores por default 
`load_module modules/ngx_http_js_module.so; # Load njs module for javascript `

```sh
user nginx; 
worker_processes 1; 
error_log /var/log/nginx/error.log warn; 
pid /var/run/nginx.pid; 
worker_rlimit_nofile 8192; 
env TOKEN;
```

Contexto de eventos 
En este apartado se define la forma en la que las conexiones son tomadas a nivel general por ejemplo el parámetro definido sirve para delimitar el número de conexiones que se pueden establecer de forma concurrida, 1 es el valor por default 

```sh
events { 
    worker_connections 1024; 
} 
```

Contexto http/https
En este apartado se contiene las directivas y otros contextos ( a nivel más especifico) necesarios para definir como las peticiones serán tomadas, a este nivel de contexto se ha incluido oauth2_attributes.js para que se pueda utilizan dentro del apartado de server  

```sh
http { 
  # Directives 
  include /etc/nginx/mime.types; 
  include /etc/nginx/proxy.conf; 
  index index.html index.htm;

  js_include oauth2_attributes.js; # Include JavaScript code 
  js_set $token fetch_token;

  default_type application/octet-stream; 
  log_format main '$remote_addr - $remote_user [$time_local] $status ' 
  '"$request" $body_bytes_sent "$http_referer" ' 
  '"$http_user_agent" "$http_x_forwarded_for"';

  access_log /var/log/nginx/access.log main;
  sendfile on; 
  tcp_nopush on; 

  server_names_hash_bucket_size 128; # this seems to be required for some vhosts 

```

Contexto servidor 
En este contexto es donde se definen las directivas que toman la petición del cliente y la procesan dependiendo el caso 
```sh
  server { 
    listen 80;
  }
```
En esta sección se definen algunos bloques del contexto de ubicación. 
Aquí es donde se llevan a cabo las configuraciones necesarias para el redireccionanmiento 
En primer lugar se define una ubicación global a modo de ejemplo la cual contiene lo siguiente: 

```sh
    proxy_pass http://web_proxy;
```
El cual permite redirigir todo el tráfico hacia el dominio http://web_proxy 

```sh
    proxy_set_header X-Token $token;
```
El cual permite añadir un header a la petición actual  

```sh
    auth_request_set
```
El cual permite crear una variable de entorno a partir del resultado de la directiva auth_request 

```sh
    auth_request 
```
Es una directiva que permite la autenticación basada en un subrequest mismo que tambien se define como un location/ más, en este caso llamando a _oauth_token_introspection 
En resumen este bloque se muestra asi: 

```sh
    location / { # simple reverse-proxy 
      auth_request /_oauth_token_introspection;  
      auth_request_set $username $sent_http_token_username;  
      proxy_set_header X-Test "Basic key"; 
      proxy_set_header X-Username $username; 
      proxy_set_header X-Token $token; 
      proxy_pass http://web_proxy; 
    }
``` 
La definición de este location es un poco diferente ya que no es de acceso publico, su funcionamiento es interno y se basa en el código de respuesta, por ejemplo si la solicitud devuelve 2xx, el acceso es permitido, pero si devuelve 4xx entonces el proceso se detiene y devuelve error al usuario 

```sh
    js_content introspectAccessToken; 
```
En este caso particular se añadió una directiva para cargar un script de javascript el cual permitirá almacenar en variables los resultados obtenidos como lo son el token a los servicios web privados 
Se definen algunos headers así como la ruta a la que se requiere acceso.  

```sh
    internal
```
Es para establecer que el acceso es solo interno  

```sh
    location = /_oauth_token_introspection { 
    internal; 
    proxy_set_header Authorization "Basic auth_token"; 
    proxy_set_header Content-Type "application/x-www-form-urlencoded"; 
    proxy_set_body "token=$http_apikey&token_hint=access_token"; 
    proxy_pass http://web_proxy/hello-world; 
    js_content introspectAccessToken; 

    } 
```
En este apartado se realiza la petición previa para obtener el token 



Si lo requieres el código fuente de esto se encuentra en el siguiente repositorio https://github.com/AndresJz/nginx-proxy

Links de apoyo

http://nginx.org/en/docs/http/ngx_http_js_module.html#js_include 
https://www.nginx.com/resources/wiki/start/topics/examples/full/ 
https://github.com/mendhak/docker-http-https-echo 
https://serverfault.com/questions/577370/how-can-i-use-environment-variables-in-nginx-conf 
https://www.nginx.com/blog/validating-oauth-2-0-access-tokens-nginx/ 
https://www.mock-server.com/where/docker.html 
