## Habilitar tracing nginx

1. Determinar version de nginx ejecutando el siguiente comando.

	nginx -v

   	Ejemplo:

	   	root@ubuntu-server:~# nginx -v
		nginx version: nginx/1.18.0 (Ubuntu)

Descargar el binario

	wget https://_:{api_key}@artifact-public.instana.io/artifactory/shared/com/instana/nginx_tracing/1.8.3/linux-amd64-glibc-nginx-1.22.1.zip


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


