## Habilitar tracing nginx

1. Determinar version de nginx ejecutando el siguiente comando.

	Comando:

		nginx -v

   	Resultado:

	  	root@ubuntu-server:~# nginx -v
		nginx version: nginx/1.18.0 (Ubuntu)

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


3. Subir el archivo zip descargado en el paso anterior al servidor nginx y descomprimir. 

	Comando:

		unzip linux-amd64-glibc-nginx-{nginx-version}.zip

	Ejemplo de ejecucion:

		root@ubuntu-server:~# unzip linux-amd64-glibc-nginx-1.18.0.zip 
		Archive:  linux-amd64-glibc-nginx-1.18.0.zip
		  inflating: glibc-libinstana_sensor.so  
		  inflating: glibc-nginx-1.18.0-ngx_http_ot_module.so  
		root@ubuntu-server:~# ls -lart
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

		root@ubuntu-server:~# chmod 775 glibc-*
		root@ubuntu-server:~# ls -lart
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

		root@ubuntu-server:~# mv glibc-nginx-1.18.0-ngx_http_ot_module.so ngx_http_opentracing_module.so
		root@ubuntu-server:~# mv glibc-libinstana_sensor.so libinstana_sensor.so
		root@ubuntu-server:~# ls -lart
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


7. Crear la carpeta de modulos en caso no existir.

	Comando:
	
	 	mkdir /usr/lib/nginx
		mkdir /usr/lib/nginx/modules

8. Copiar los binarios en la carpeta de modulos de nginx

	Comando:

	   cp ngx_http_opentracing_module.so libinstana_sensor.so /usr/lib/nginx/modules
	   ls -lart /usr/lib/nginx/modules

   	Resultado:

		root@ubuntu-server:~# cp ngx_http_opentracing_module.so libinstana_sensor.so /usr/lib/nginx/modules
		root@ubuntu-server:~# ls -lart /usr/lib/nginx/modules
		total 3216
		-rw-r--r-- 1 root root  180792 Nov 10  2022 ngx_stream_module.so
		-rw-r--r-- 1 root root  108168 Nov 10  2022 ngx_mail_module.so
		-rw-r--r-- 1 root root   23552 Nov 10  2022 ngx_http_xslt_filter_module.so
		-rw-r--r-- 1 root root   27728 Nov 10  2022 ngx_http_image_filter_module.so
		drwxr-xr-x 3 root root    4096 Mar 15 03:11 ..
		drwxr-xr-x 2 root root    4096 Mar 15 03:56 .
		-rwxrwxr-x 1 root root 1147832 Mar 15 03:57 ngx_http_opentracing_module.so
		-rwxrwxr-x 1 root root 1783768 Mar 15 03:57 libinstana_sensor.so

9. Crear el archivo instana-config.json con el nombre del servicio (con este nombre sera reconocido en Instana )  y los datos del agente Instana.

	Comando:

		vi /etc/instana-config.json

   	Contenido del archivo instana-config.json :

  		{
		  "service": "nginxtracing_nginx",
		  "agent_host": "127.0.0.1",
		  "agent_port": 42699,
		  "max_buffered_spans": 1000
		}

10. Modificar el archivo de configuracion de nginx agregando los modulos y variables para habilitar el tracing.

	Comando:

		cd /etc/nginx
		vi nginx.conf

	Agregar las siguientes lineas de configuracion:

		load_module modules/ngx_http_opentracing_module.so; 
			
		env INSTANA_SERVICE_NAME;
		env INSTANA_AGENT_HOST;
		env INSTANA_AGENT_PORT;
		env INSTANA_MAX_BUFFERED_SPANS;
		env INSTANA_DEV;

		#Dentro de la sección http
		
		opentracing_load_tracer /usr/lib/nginx/modules/libinstana_sensor.so /etc/instana-config.json;
		opentracing_propagate_context;


	Ejemplo del archivo de configuracion modificado:



	load_module modules/ngx_http_opentracing_module.so; 
	
	env INSTANA_SERVICE_NAME;
	env INSTANA_AGENT_HOST;
	env INSTANA_AGENT_PORT;
	env INSTANA_MAX_BUFFERED_SPANS;
	env INSTANA_DEV;
	
---> dentro de http
	
	opentracing_load_tracer /usr/lib/nginx/modules/libinstana_sensor.so /etc/instana-config.json;
	
	opentracing_propagate_context;


