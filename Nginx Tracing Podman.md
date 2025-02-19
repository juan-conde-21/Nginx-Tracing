## Habilitar Tracing para Nginx en Podman con Instana

1. Determinar la version de nginx utilizada.

   Comando:

       podman exec -it [containerID] bash
       nginx -v

   Resultado:

       [root@podman-server ~]# podman exec -it 60699ccd084b bash
       root@60699ccd084b:/# nginx -v
       nginx version: nginx/1.26.2


2. Descargar el binario correspondiente a la version de nginx instalada.

   - Ingresar a la siguiente ruta:

     https://www.ibm.com/docs/en/instana-observability/current?topic=nginx-distributed-tracing-binaries

   - Identificar la version del binario correspondiente.

     ![image](https://github.com/juan-conde-21/Nginx-Tracing/assets/13276404/3a71eb24-5b91-4ba2-a02b-6aa91809a9c9)

   - Descargar el archivo zip correspondiente a la distribucion de Linux utilizado.

     ![image](https://github.com/juan-conde-21/Nginx-Tracing/assets/13276404/b006457e-8b7c-4d7a-9f72-880165970348)

     ![image](https://github.com/juan-conde-21/Nginx-Tracing/assets/13276404/be183c5d-cab0-4c8c-9a8c-c8b6f6fe1f63)

   Para autenticarse en el repositorio de Instana utilizar como usuario "_" (guión bajo) y contraseña el agent key de Instana (utilizado para la descarga del agente) .

   ![image](https://github.com/juan-conde-21/Nginx-Tracing/assets/13276404/b910a8f3-bab3-4746-b8ce-88a0fdf8085b)

3. Utilizar el archivo zip descargado en el paso anterior, subir y descomprimir en el contenedor. 

   Comando:

       unzip linux-amd64-glibc-nginx-{nginx-version}.zip

   Ejemplo de ejecucion:

       root@60699ccd084b:~# unzip linux-amd64-glibc-nginx-1.18.0.zip 
       Archive:  linux-amd64-glibc-nginx-1.18.0.zip
       inflating: glibc-libinstana_sensor.so  
       inflating: glibc-nginx-1.18.0-ngx_http_ot_module.so  
       root@60699ccd084b:~# ls -lart
       total 3924
       -rw-r--r--  1 root root     161 Dec  5  2019 .profile
       -rw-r--r--  1 root root    3106 Dec  5  2019 .bashrc
       -rwxr-xr-x  1 root root 1147832 Jan 22 16:24 glibc-nginx-1.18.0-ngx_http_ot_module.so
       -rwxr-xr-x  1 root root 1783768 Jan 22 16:24 glibc-libinstana_sensor.so
       -rwxrwxr-x  1 root root 1054316 Jan 22 16:29 linux-amd64-glibc-nginx-1.18.0.zip
       drwxr-xr-x 19 root root    4096 Mar 15 03:01 ..
       drwx------  2 root root    4096 Mar 15 03:01 .ssh
       drwx------  3 root root    4096 Mar 15 03:01 snap
       drwx------  4 root root    4096 Mar 15 03:38 .


4. Agregar permisos de ejecucion sobre los binarios descargados

   Comando:

       chmod 775 glibc-*

   Resultado:

       root@60699ccd084b:~# chmod 775 glibc-*
       root@60699ccd084b:~# ls -lart
       total 3924
       -rw-r--r--  1 root root     161 Dec  5  2019 .profile
       -rw-r--r--  1 root root    3106 Dec  5  2019 .bashrc
       -rwxrwxr-x  1 root root 1147832 Jan 22 16:24 glibc-nginx-1.18.0-ngx_http_ot_module.so
       -rwxrwxr-x  1 root root 1783768 Jan 22 16:24 glibc-libinstana_sensor.so		

5. Modificar los nombres de las librerias descargadas.

   Comando:

       mv glibc-nginx-{nginx-version}-ngx_http_ot_module.so ngx_http_opentracing_module.so
       mv glibc-libinstana_sensor.so libinstana_sensor.so

   Resultado:

       root@60699ccd084b:~# mv glibc-nginx-1.18.0-ngx_http_ot_module.so ngx_http_opentracing_module.so
       root@60699ccd084b:~# mv glibc-libinstana_sensor.so libinstana_sensor.so
       root@60699ccd084b:~# ls -lart
       total 3924
       -rw-r--r--  1 root root     161 Dec  5  2019 .profile
       -rw-r--r--  1 root root    3106 Dec  5  2019 .bashrc
       -rwxrwxr-x  1 root root 1147832 Jan 22 16:24 ngx_http_opentracing_module.so
       -rwxrwxr-x  1 root root 1783768 Jan 22 16:24 glibc-libinstana_sensor.so


6. Identificar el path de modulos de nginx.

   Comando:

       nginx -V

   Identificar la ubicacion de la ruta de los modulos de nginx:
	
       --modules-path=

   Resultado:

   ![image](https://github.com/juan-conde-21/Nginx-Tracing/assets/13276404/b6d79588-3fc3-4ca8-b21d-5a5602443542)

   *Nota: esta ruta depende de la configuración aplicada a nginx, para este ejemplo se encuentra en la ruta "/usr/lib/nginx/modules"

7. Crear la carpeta de modulos en caso no existir. (Opcional)

   Comando:
	
       mkdir /usr/lib/nginx
       mkdir /usr/lib/nginx/modules

8. Copiar los binarios en la carpeta de modulos de nginx, para este ejemplo esta en la ruta "/usr/lib/nginx/modules"

   Comando:

       cp ngx_http_opentracing_module.so libinstana_sensor.so /usr/lib/nginx/modules
       ls -lart /usr/lib/nginx/modules

   Resultado:

       root@60699ccd084b:~# cp ngx_http_opentracing_module.so libinstana_sensor.so /usr/lib/nginx/modules
       root@60699ccd084b:~# ls -lart /usr/lib/nginx/modules
       total 3216
       -rw-r--r-- 1 root root  180792 Nov 10  2022 ngx_stream_module.so
       -rw-r--r-- 1 root root  108168 Nov 10  2022 ngx_mail_module.so
       -rw-r--r-- 1 root root   23552 Nov 10  2022 ngx_http_xslt_filter_module.so
       -rw-r--r-- 1 root root   27728 Nov 10  2022 ngx_http_image_filter_module.so
       drwxr-xr-x 3 root root    4096 Mar 15 03:11 ..
       drwxr-xr-x 2 root root    4096 Mar 15 03:56 .
       -rwxrwxr-x 1 root root 1147832 Mar 15 03:57 ngx_http_opentracing_module.so
       -rwxrwxr-x 1 root root 1783768 Mar 15 03:57 libinstana_sensor.so

9. Crear el archivo instana-config.json con el nombre del servicio (con este nombre sera reconocido en Instana ), para este ejemplo se trabaja el nombre nginxtracing_nginx_1.26.2.

   Comando:

       vi /etc/instana-config.json

   Contenido del archivo instana-config.json :

       {
       "service": "nginxtracing_nginx_1.26.2",
       "agent_host": "host.containers.internal",
       "agent_port": 42699,
       "max_buffered_spans": 1000
       }

10. Modificar el archivo de configuracion de nginx agregando los modulos y variables para habilitar el tracing. Para este ejemplo la ruta de modulos es la siguiente "/usr/lib/nginx/modules"

    Comando:

        cd /etc/nginx
        vi nginx.conf

    Agregar las siguientes lineas de configuracion en la parte superior del archivo:

        load_module /usr/lib/nginx/modules/ngx_http_opentracing_module.so; 
			
        env INSTANA_SERVICE_NAME;
        env INSTANA_AGENT_HOST;
        env INSTANA_AGENT_PORT;
        env INSTANA_MAX_BUFFERED_SPANS;
        env INSTANA_DEV;

    Ejemplo del archivo de configuracion modificado:

        load_module /usr/lib/nginx/modules/ngx_http_opentracing_module.so;
		
        env INSTANA_SERVICE_NAME;
        env INSTANA_AGENT_HOST;
        env INSTANA_AGENT_PORT;
        env INSTANA_MAX_BUFFERED_SPANS;
        env INSTANA_DEV;
		
        user www-data;
        worker_processes auto;
        pid /run/nginx.pid;
        include /etc/nginx/modules-enabled/*.conf;
		
        events {
                worker_connections 768;
	        # multi_accept on;
        }
		
        http {
	        ##
	        # Basic Settings
	        ##


11. Luego modificar el archivo donde se han definido las reglas para redireccionar solicitudes web hacia otras URL o servidores.


    Agregar al inicio de la linea del archivo de configuracion:
		
        opentracing_load_tracer /usr/lib/nginx/modules/libinstana_sensor.so /etc/instana-config.json;
        opentracing_propagate_context;


    Ejemplo del archivo de configuracion modificado:


        root@60699ccd084b:/etc/nginx/conf.d# cat default.conf
	
        opentracing_load_tracer /usr/lib/nginx/modules/libinstana_sensor.so /etc/instana-config.json;
        opentracing_propagate_context;

        server {
	    listen       80;
	    listen  [::]:80;
	    server_name  localhost;
	
	    #access_log  /var/log/nginx/host.access.log  main;
	
	    location / {
	        root   /usr/share/nginx/html;
	        index  index.html index.htm;
	    }
	
	    location /nginx_status {
	        stub_status  on;
	        access_log   off;
	    }
	
	    location /status_app {
	        proxy_pass http://192.168.68.60:9090/status;
	
	    }
	
	
	    #error_page  404              /404.html;
	
	    # redirect server error pages to the static page /50x.html
	    #
	    error_page   500 502 503 504  /50x.html;
	    location = /50x.html {
	        root   /usr/share/nginx/html;
	    }

    
  
12. Reiniciar los servicios de nginx para aplicar los cambios:

    Comando:

        nginx -s reload

    Resultado:

        root@60699ccd084b:/etc/nginx/conf.d# nginx -s reload
        Instana C++ Sensor v1.11.0 has been loaded.
        2025/02/19 03:45:15 [notice] 40#40: signal process started

13. Reiniciar el contendor nginx.

    Comando:

        podman restart 60699ccd084b


14. Verificar en la consola de Instana, buscar por el nombre del servicio colocado en el archivo instana-config.json

    ![image](https://github.com/user-attachments/assets/b454ec4c-3a91-49b6-8720-ff3f53df3a9e)

    ![image](https://github.com/user-attachments/assets/e3c7e766-72f2-499e-88be-c8868885ec27)

    ![image](https://github.com/user-attachments/assets/180276e9-b159-4295-ba4d-620c0cb02c44)





