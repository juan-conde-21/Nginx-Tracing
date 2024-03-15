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

   	Para autenticarse en el repositorio de Instana utilizar como usuario "_" y contraseña el agent key de Instana (utilizado para la descarga del agente) .
	

Instalar unzip o descomprimir localmente y luego subir al servidor

Agregar permisos de ejecucion sobre los binarios descargados

	chmod 775 glibc-*


Identificar el path de modulos

	nginx -V

	Buscar por la ubicacion de la ruta para los modulos
	
	--modules-path

Modificar los nombres de las librerias descargadas


	mv glibc-nginx-1.22.1-ngx_http_ot_module.so ngx_http_opentracing_module.so
	mv glibc-libinstana_sensor.so libinstana_sensor.so

Crear la carpeta de modulos en caso no existir

	mkdir /usr/lib/nginx
	mkdir /usr/lib/nginx/modules


Copiar los binarios en la carpeta de modulos de nginx

	mv ngx_http_opentracing_module.so libinstana_sensor.so /usr/lib/nginx/modules


Crear el archivo instana-config.json con los siguientes datos

	vi /etc/instana-config.json
	
	{
	  "service": "nginxtracing_nginx",
	  "agent_host": "127.0.0.1",
	  "agent_port": 42699,
	  "max_buffered_spans": 1000
	}



Agregar al archivo de configuracion

cd /etc/nginx

vi nginx.conf

	load_module modules/ngx_http_opentracing_module.so; 
	
	env INSTANA_SERVICE_NAME;
	env INSTANA_AGENT_HOST;
	env INSTANA_AGENT_PORT;
	env INSTANA_MAX_BUFFERED_SPANS;
	env INSTANA_DEV;
	
---> dentro de http
	
	opentracing_load_tracer /usr/lib/nginx/modules/libinstana_sensor.so /etc/instana-config.json;
	
	opentracing_propagate_context;


