# Angie Pro
	user@angie01:~$ curl -I http://localhost
	HTTP/1.1 200 OK
	Server: Angie/1.11.5
	Date: Sat, 16 May 2026 19:07:46 GMT
	Content-Type: text/html
	Content-Length: 546
	Last-Modified: Thu, 14 May 2026 16:27:31 GMT
	Connection: keep-alive
	ETag: "6a05f7f3-222"
	Accept-Ranges: bytes

	1. Установить модуль Brotli
	apt install angie-pro-module-brotli
	2. Установить модуль Auth LDAP
	apt install angie-pro-module-auth-ldap
	3. Добавить в /etc/angie/angie.conf
	load_module modules/ngx_http_brotli_filter_module.so;
	load_module modules/ngx_http_auth_ldap_module.so;
	4. user@angie01:~$ sudo angie -t
	angie:
	angie: Valid license found:
	angie:   - owner: CN=Angie Client License / Лицензия Angie PRO
	angie:   - period: May 11 21:00:00 2026 GMT .. Oct 12 20:59:59 2026 GMT
	angie:
	angie: Limitations:
	angie:   - worker_processes: 2
	angie:   - worker_connections: 256
	angie:
	angie: the configuration file /etc/angie/angie.conf syntax is ok
	angie: configuration file /etc/angie/angie.conf test is successful
	5. systemctl reload angie.service

# Docker
1. sudo docker run --name angie_docker -v /var/www/html:/usr/share/angie/html:ro -v /home/db/angie:/etc/angie:ro -p 8080:80 -d docker.angie.software/angie:1.11.5-ubuntu
2. root@angie01:~# docker ps -a
   
CONTAINER ID   IMAGE                                       COMMAND                  CREATED          STATUS         PORTS                                     NAMES

c2d2f53ed888   docker.angie.software/angie:1.11.5-ubuntu   "angie -g 'daemon of…"   20 minutes ago   Up 3 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   angie_docker

4. root@angie01:~# curl -I http://localhost:8080
	HTTP/1.1 200 OK
	Server: Angie/1.11.5
	Date: Sun, 17 May 2026 10:16:14 GMT
	Content-Type: text/html
	Content-Length: 568
	Last-Modified: Sun, 17 May 2026 10:15:33 GMT
	Connection: keep-alive
	ETag: "6a099545-238"
	Accept-Ranges: bytes
5. root@angie01:~# curl -I http://localhost:8080/status
	HTTP/1.1 301 Moved Permanently
	Server: Angie/1.11.5
	Date: Sat, 16 May 2026 19:53:47 GMT
	Content-Type: text/html
	Content-Length: 169
	Location: http://localhost/status/
	Connection: keep-alive

# Миграция Nginx в Angie

	1. root@angie01:/etc/nginx/sites-available# grep -rn '/nginx' /etc/angie
	   /etc/angie/default:19:  access_log /var/log/nginx/default.log;
	   /etc/angie/default:39:        include /etc/nginx/static-avif.conf;
	   /etc/angie/default:45:        include /etc/nginx/static.conf;

	2. root@angie01:/etc/angie# find . -type f -name '*.conf' -exec sed --follow-symlinks -i 's|/nginx|/angie|g' {} \;

	3. root@angie01:/etc/angie# cat script.sh
	ln -nsf "$(readlink "/etc/angie/sites-enabled/default" | sed s!/etc/nginx/sites-available!/etc/angie/sites-available!)" 
	"$(echo "/etc/angie/sites-enabled/default" | sed s!/etc/nginx/sites-available!/etc/angie/sites-available!)"
	ln -nsf "$(readlink "/etc/angie/sites-enabled/test1" | sed s!/etc/nginx/sites-available!/etc/angie/sites-available!)" 
	"$(echo "/etc/angie/sites-enabled/test1" | sed s!/etc/nginx/sites-available!/etc/angie/sites-available!)"
	ln -nsf "$(readlink "/etc/angie/sites-enabled/test2" | sed s!/etc/nginx/sites-available!/etc/angie/sites-available!)" 
	"$(echo "/etc/angie/sites-enabled/test2" | sed s!/etc/nginx/sites-available!/etc/angie/sites-available!)"
   
	4. root@angie01:/etc/angie# ./script.sh
   	root@angie01:/etc/angie# ll sites-enabled/
   	total 8
   	drwxr-xr-x 2 root root 4096 May 23 13:10 ./
   	drwxr-xr-x 7 root root 4096 May 23 13:05 ../
   	lrwxrwxrwx 1 root root   34 May 23 13:10 default -> /etc/angie/sites-available/default
   	lrwxrwxrwx 1 root root   32 May 23 13:10 test1 -> /etc/angie/sites-available/test1
   	lrwxrwxrwx 1 root root   32 May 23 13:10 test2 -> /etc/angie/sites-available/test2

	5. root@angie01:/etc/angie/sites-available# angie -t
	angie:
	angie: Valid license found:
	angie:   - owner: CN=Angie Client License / Лицензия Angie PRO
	angie:   - period: May 11 21:00:00 2026 GMT .. Oct 12 20:59:59 2026 GMT
	angie:
	angie: Limitations:
	angie:   - worker_processes: 2
	angie:   - worker_connections: 256
	angie:
	angie: the configuration file /etc/angie/angie.conf syntax is ok
	angie: configuration file /etc/angie/angie.conf test is successful
 
	6. Остановили nginx и отключили автозапуск
   	root@angie01:/etc/angie/sites-available# service nginx status
	○ nginx.service - A high performance web server and a reverse proxy server
    	 Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: enabled)
    	 Active: inactive (dead)
    	   Docs: man:nginx(8)

	7. Запуск angie и включили автозапуск
   	root@angie01:/etc/angie/sites-available# service angie status
	● angie.service - Angie - high performance web server
    	 Loaded: loaded (/usr/lib/systemd/system/angie.service; enabled; preset: enabled)
    	 Active: active (running) since Sat 2026-05-23 09:41:31 UTC; 3h 51min ago
    	   Docs: https://en.angie.software/angie/docs/
    	Process: 755 ExecStart=/usr/sbin/angie -c /etc/angie/angie.conf (code=exited, status=0/SUCCESS)
   	Main PID: 791 (angie)
    	  Tasks: 3 (limit: 2313)
     	Memory: 9.0M (peak: 9.5M)
        	CPU: 59ms
     	CGroup: /system.slice/angie.service
             ├─791 "angie: master process v1.11.5 PRO #1 [/usr/sbin/angie -c /etc/angie/angie.conf]"
             ├─792 "angie: worker process #1"
             └─793 "angie: worker process #1"

	May 23 09:41:31 angie01 systemd[1]: Starting angie.service - Angie - high performance web server...
	May 23 09:41:31 angie01 systemd[1]: Started angie.service - Angie - high performance web server.

	8. root@angie01:/etc/angie/sites-available# curl -I localhost:80
	HTTP/1.1 200 OK
	Server: Angie/1.11.5
	Date: Sat, 23 May 2026 13:37:04 GMT
	Content-Type: text/html
	Content-Length: 546
	Last-Modified: Thu, 14 May 2026 16:27:31 GMT
	Connection: keep-alive
	ETag: "6a05f7f3-222"
	Accept-Ranges: bytes

	9. root@angie01:/etc/angie/sites-available# curl localhost
	<!DOCTYPE html>
	<html>
	<head>
	<title>Welcome to nginx!</title>
	<style>
	html { color-scheme: light dark; }
	body { width: 35em; margin: 0 auto;
	font-family: Tahoma, Verdana, Arial, sans-serif; }
	</style>
	</head>
	<body>
	<h1>Welcome to nginx!</h1>
	<p>If you see this page, the nginx web server is successfully installed and
	working. Further configuration is required.</p>

	<p>For online documentation and support please refer to
	<a href="http://nginx.org/">nginx.org</a>.<br/>
	Commercial support is available at
	<a href="http://nginx.com/">nginx.com</a>.</p>

	<p><em>Thank you for using nginx.</em></p>
	</body>
	</html>

# Статический сайт
	/etc/angie/angie.conf
	
	user angie;
	worker_processes 2;
	worker_rlimit_nofile 65536;
	
	error_log  /var/log/angie/error.log notice;
	pid        /run/angie.pid;
	
	load_module modules/ngx_http_brotli_filter_module.so;
	load_module modules/ngx_http_brotli_static_module.so;
	
	events {
	    worker_connections  256;
	}
		
	http {
	    include       /etc/angie/mime.types;
	    default_type  application/octet-stream;
	
	    access_log  /var/log/angie/access.log  main;
	
	    sendfile        on;
	    keepalive_timeout  65;
	
	    gzip on;
	
	    include /etc/angie/http.d/*.conf;
	    include /etc/angie/sites-enabled/*;
	
	    map $http_user_agent $limit_search_bots {
	        default 0;
	        ~*(google|yandex|bing|wget|msnbot|apachebench|curl) 1;
	    }
	}

	/etc/angie/http.d/default.conf
	
	server {
	        listen 80 reuseport default_server;
	
	        root /var/www/html;
	
	        index index.html index.htm index.php;
	
	        server_name _;
	
	        return 301 $scheme://$host:8080$request_uri;
	
	        if ($limit_search_bots = 1) {
	                return 401;
	        }
	
	 		# Static files location
	   		location /images/ {
	     		root /var/www/html/images/;
	     		include /etc/angie/static.conf;
	   		}
	
	
	  		location ~* \.(ttf|eot|svg|woff|woff2|css|js|json|ico|zip|tgz|gz|rar|bz2|doc|docx|xlsx|pptx|xls|exe|pdf|ppt|txt|tar|mid|midi|wav|bmp|rtf|avi|swf|flv|mp3|mp4|fla)$ {
	        	expires max;
	        	include /etc/angie/static.conf;
	  		}
	
	  		location ~* \.(jpg|jpeg|gif|png|ico)$ {
	        	root /var/www/html/images/;
	        	include /etc/angie/static.conf;
	  		}
	
	  		location /old {
	        	rewrite ^/old /new permanent;
	  		}
	
	  		location /newsite {
	        	proxy_pass http://127.0.0.1:8080;
	        }
		}
	
	/etc/angie/static.conf
		add_header Cache-Control "max-age=31536000, public, no-transform, immutable";
 
# Reverse proxy (Angie + Wordpress + MySQL)
1. Создаем директории для MySQL и Wordpress:
   
   		mkdir -p /home/mysql #volume for docker mysql
		mkdir -p /var/www/html/wordpress #volume for docker wordpress = root path
		ln -s /etc/angie/sites-available/wordpress /etc/angie/sites-enabled/wordpress #config for wordpress
2. Используем Angie на хостовой машине как reverse proxy:
   
		/etc/angie/sites-available/wordpress
			server {
	        		listen 80;
	        		root /var/www/html/wordpress;
	
	        index index.php;
	
	        server_name www.wordpress.ru;
		
	        if ($limit_search_bots = 1) {
	                return 401;
	        }
	
	        access_log /var/log/angie/wordpress-access.log;
	        error_log /var/log/angie/wordpress-error.log;
	
	        location / {
		            try_files $uri $uri/ /index.php?$args;
		    }
	
		# Static files location
			location ~* \.(ttf|eot|svg|woff|woff2|css|js|json|ico|zip|tgz|gz|rar|bz2|doc|docx|xlsx|pptx|xls|exe|pdf|ppt|txt|tar|mid|midi|wav|bmp|rtf|avi|swf|flv|mp3|mp4|fla)$ {
				expires max;
	        	include /etc/angie/static.conf;
	 	 }
	
	  		location ~* \.(jpg|jpeg|gif|png|ico)$ {
	        	include /etc/angie/static.conf;
	  	}
	  
	  		location =/info.php {
	        	allow 127.0.0.1;
	        	allow 87.117.185.72;
	        	deny all;
	
	        	include fastcgi_params;
	        	fastcgi_param SCRIPT_FILENAME /var/www/html/$fastcgi_script_name;
	        	fastcgi_pass 127.0.0.1:9000;
	  	}
	
	  		location ~ \.php$ {
	        	include fastcgi_params;
	        	fastcgi_param  SCRIPT_FILENAME /var/www/html/$fastcgi_script_name;
	        	fastcgi_pass 127.0.0.1:9000;
	        }
		}
3. Используем Docker compose для контейнеров Mysql и Wordpress
   
		/home/user/docker-compose.yml
			version: '3'
			services:
	  			db:
	    			image: mysql:8.0
	    			container_name: db
	    			restart: always
	    			env_file: .env
	    			environment:
	      			- MYSQL_DATABASE=wordpress
		    		volumes:
	    			- /home/mysql:/var/lib/mysql
	    			command: '--default-authentication-plugin=mysql_native_password'
	    			networks:
	  				- app-network
	
	  			wordpress:
	    		depends_on:
	      		- db
	    			image: wordpress:fpm
	    			container_name: wordpress
	    			restart: always
	    			env_file: .env
	    			environment:
	      			- WORDPRESS_DB_HOST=db:3306
	      			- WORDPRESS_DB_USER=$MYSQL_USER
	      			- WORDPRESS_DB_PASSWORD=$MYSQL_PASSWORD
	      			- WORDPRESS_DB_NAME=wordpress
	    			volumes:
	      			- /var/www/html/wordpress:/var/www/html
	    			ports:
	      			- "127.0.0.1:9000:9000"
	    			networks:
	      			- app-network

		networks:
	  		app-network:
	    		driver: bridge
4. 	/home/user/.env
   
    	MYSQL_ROOT_PASSWORD=dfkljs_d324fD_klj
		MYSQL_USER=wp
		MYSQL_PASSWORD=KMJ23f9sad_80ds
5. /home/user/.dockerignore
    
   		.env
6. Загружаем и запускаем контейнеры
    
   		/home/user/docker compose up -d
7. Проверка
    
   		root@angie01:/home/user# docker compose ps
		NAME        IMAGE           COMMAND                  SERVICE     CREATED        STATUS          PORTS
		db          mysql:8.0       "docker-entrypoint.s…"   db          20 hours ago   Up 17 minutes   3306/tcp, 33060/tcp
		wordpress   wordpress:fpm   "docker-entrypoint.s…"   wordpress   20 hours ago   Up 17 minutes   127.0.0.1:9000->9000/tcp
8. Проверка конфигурации и перезапуск Angie
    
   		angie -t && service angie reload
9. Проверка сайта до настройки
    
   		root@angie01:/etc/angie/sites-available# curl -I www.wordpress.ru
		HTTP/1.1 302 Found
		Server: Angie/1.11.5
		Date: Thu, 11 Jun 2026 16:36:40 GMT
		Content-Type: text/html; charset=UTF-8
		Connection: keep-alive
		X-Powered-By: PHP/8.3.31
		Expires: Wed, 11 Jan 1984 05:00:00 GMT
		Cache-Control: no-cache, must-revalidate, max-age=0, no-store, private
		X-Redirect-By: WordPress
		Location: http://www.wordpress.ru/wp-admin/install.php
10. Проверка сайта после настройки
    
   		root@angie01:/home/mysql# curl -I www.wordpress.ru
		HTTP/1.1 200 OK
		Server: Angie/1.11.5
		Date: Thu, 11 Jun 2026 16:42:46 GMT
		Content-Type: text/html; charset=UTF-8
		Connection: keep-alive
		X-Powered-By: PHP/8.3.31
		Link: <http://www.wordpress.ru/wp-json/>; rel="https://api.w.org/"

# Клиентская оптимизация и серверное кэширование
1. /etc/angie/angie.conf:

		user www-data;
		worker_processes auto;
		worker_cpu_affinity auto;
		worker_rlimit_nofile 65536;

		error_log  /var/log/angie/error.log notice;
		pid        /run/angie.pid;

		load_module modules/ngx_http_brotli_filter_module.so;
		load_module modules/ngx_http_brotli_static_module.so;

		load_module modules/ngx_http_zstd_filter_module.so;
		load_module modules/ngx_http_zstd_static_module.so;

		events {
    		worker_connections  256;
		}

		pcre_jit on;

		http {
    		include       /etc/angie/mime.types;
    		default_type  application/octet-stream;

    		log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    		log_format extended '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" rt="$request_time" '
                        '"$http_user_agent" "$http_x_forwarded_for" '
                        'h="$host" sn="$server_name" ru="$request_uri" u="$uri" '
                        'ucs="$upstream_cache_status" ua="$upstream_addr" us="$upstream_status" '
                        'uct="$upstream_connect_time" urt="$upstream_response_time"';

    		access_log  /var/log/angie/access.log  main;

    		sendfile        on;		# отдача файлов с диска в сетевой сокет по http
    		tcp_nopush      on;		# накопить данные перед отправкой и передавать полными пакетами
    		tcp_nodelay     on;		# быстрая отправка данных без задержек

    		reset_timedout_connection on;	# сброс висячих коннектов после таймаута

    		keepalive_timeout  120;		# поддерживать клиентские соединения 120 сек
    		keepalive_requests 10000;	# максимальное кол-во запросов в одном клиентском соединении

    		send_timeout 10;			# таймаут между запросами для медленных/висячих клиентов
    		client_body_timeout 10;		# таймаут для тела запросов
    		client_header_timeout 10;	# таймаут для заголовков запросов

			# серверное кэширование
    		proxy_cache_valid 1m;
    		proxy_cache_key $scheme$host$uri;
    		proxy_cache_path /var/www/cache levels=1:2 keys_zone=one:10m:file=/etc/angie/cache.state inactive=4h max_size=800m;

			# работа с бэкендом
    		proxy_connect_timeout 5;
    		proxy_send_timeout 10;
    		proxy_read_timeout 10;

    		proxy_buffers 64 4k;

			# работа с файлами
			open_file_cache max=10000 inactive=60s;
    		open_file_cache_valid 30s;
   			open_file_cache_min_uses 2;

    		open_log_file_cache max=100 inactive=60s min_uses=2;

			include /etc/angie/sites-enabled/*;

    		map $http_user_agent $limit_search_bots {
        		default 0;
        		~*(google|yandex|bing|wget|msnbot|apachebench) 1;
    		}

			map $http_user_agent $rate {
				default 0;
				~*(bot|crawl|wget|python|apachebench|curl) 500k;
			}

    		map $msie $cache_control {
        		default "max-age=31536000, public, no-transform, immutable";
        		"1" "max-age=31536000, private, no-transform, immutable";
    		}

    		map $msie $vary_header {
        		default "Accept";
        		"1" "";
    		}

    		map $http_accept $webp_suffix {
        		"~*webp" ".webp";
    		}

    		map $http_accept $avif_suffix {
        		"~*avif" ".avif";
        		"~*webp" ".webp";
    		}

		}
 
3. /etc/angie/sites-available/wordpress:

		server {
        listen 80 default_server reuseport;
        
        root /var/www/html/wordpress;

        index index.php;

        server_name _;

        if ($limit_search_bots = 1) {
                return 401;
        }

		# сжатие текстового контента
        gzip on;
        gzip_static on;
        gzip_types text/plain text/css text/xml application/javascript application/json application/msword application/font-ttf;
        gzip_comp_level 4;
        gzip_proxied any;
        gzip_min_length 1000;
        gzip_disable "msie6";
        gzip_vary on;
        gzip_http_version 1.0;

        brotli_static on;
        brotli on;
        brotli_comp_level 5;
        brotli_types text/plain text/css text/xml application/javascript application/json image/x-icon image/svg+xml;

        zstd on;
        zstd_min_length 256;
        zstd_comp_level 5;
        #zstd_static on;
        zstd_types text/plain text/css text/xml application/javascript application/json image/x-icon image/svg+xml;

        access_log /var/log/angie/wordpress-access.log;
        error_log /var/log/angie/wordpress-error.log;

        location / {
            proxy_cache one;
            proxy_cache_valid 200 1h;
            proxy_cache_lock on;
            proxy_cache_min_uses 2;
            proxy_ignore_headers "X-Accel-Expires" "Cache-Control" "Expires";
            proxy_cache_use_stale updating error timeout invalid_header http_500 http_502 http_504;
            proxy_cache_background_update on;

            try_files $uri $uri/ /index.php?$args;

        }

  		# Static files location
  		location ~* \.(ttf|eot|svg|woff|woff2|css|js|json|ico|zip|tgz|gz|rar|bz2|doc|docx|xlsx|pptx|xls|exe|pdf|ppt|txt|tar|mid|midi|wav|bmp|rtf|avi|swf|flv|mp3|mp4|fla)$ {
                expires max;
        	include /etc/angie/static.conf;	
  		}

  		location ~* \.(jpg|jpeg|gif|png|ico)$ {
	        include /etc/angie/static.conf;
  		}

		location ~ \.php$ {
	        fastcgi_buffering off;
        	include fastcgi_params;
        	fastcgi_param SCRIPT_FILENAME /var/www/html/$fastcgi_script_name;
        	fastcgi_pass 127.0.0.1:9000;
        }

   		location /img {
	        include /etc/angie/static-avif.conf;
   		 }
		}

4. /etc/angie/static.conf
   
   		add_header Cache-Control "max-age=31536000, public, no-transform, immutable";
   
5. /etc/angie/static-avif.conf
   
   		add_header Vary $vary_header;
		add_header Cache-Control $cache_control;
		try_files $uri$avif_suffix $uri$webp_suffix $uri =404;

6. После оптимизации сайта Lighthouse index: Performance/Accessibility/Best Ptactices/SEO - 100/97/78/54

# Настройка HTTPS
1. Создание сертификата Let'sEncrypt через certbot

   		sudo apt install certbot;

   		в angie.conf добавить строку:
   		location /.well-known/ {
			root /var/www/html/wordpress;
   		}

   		certbot certonly --dry-run --webroot -w /var/www/site -d site.ru -d www.site.ru  #тестовый запуск

   		root@angie01:/etc/angie/sites-available# certbot certonly --webroot -w /var/www/html/wordpress/ -d evg.mtdlb.ru
		Saving debug log to /var/log/letsencrypt/letsencrypt.log
		Successfully received certificate.
		Certificate is saved at: /etc/letsencrypt/live/evg.mtdlb.ru/fullchain.pem
		Key is saved at:         /etc/letsencrypt/live/evg.mtdlb.ru/privkey.pem
		This certificate expires on 2026-10-12.

2. /etc/angie/sites-available/wordpress:

		# Redirect HTTP (80) на HTTPS (443) 
		server { 
		listen 80; 
	
		server_name evg.mtdlb.ru; 
		
		return 302 https://$host$request_uri; 
		}

		server {
        listen 443 ssl;
   		http2 on;
   		#http3 не будет работать из-за openssl 3.0.13
        
        root /var/www/html/wordpress;

        index index.php;

        server_name evg.mtdlb.ru; 

        if ($limit_search_bots = 1) {
                return 401;
        }

		# HSTS
		add_header Strict-Transport-Security max-age=31536000;

	    ssl_certificate /etc/letsencrypt/live/evg.mtdlb.ru/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/evg.mtdlb.ru/privkey.pem;

        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;
        ssl_ciphers HIGH:!aNULL:!MD5;

		# Восстановление сессий
		ssl_session_cache shared:SSL:10m;
		ssl_session_timeout 1h;
		ssl_session_tickets on;

		# сжатие текстового контента
        gzip on;
        gzip_static on;
        gzip_types text/plain text/css text/xml application/javascript application/json application/msword application/font-ttf;
        gzip_comp_level 4;
        gzip_proxied any;
        gzip_min_length 1000;
        gzip_disable "msie6";
        gzip_vary on;
        gzip_http_version 1.0;

        brotli_static on;
        brotli on;
        brotli_comp_level 5;
        brotli_types text/plain text/css text/xml application/javascript application/json image/x-icon image/svg+xml;

        zstd on;
        zstd_min_length 256;
        zstd_comp_level 5;
        #zstd_static on;
        zstd_types text/plain text/css text/xml application/javascript application/json image/x-icon image/svg+xml;

        access_log /var/log/angie/wordpress-access.log;
        error_log /var/log/angie/wordpress-error.log;

        location / {
            proxy_cache one;
            proxy_cache_valid 200 1h;
            proxy_cache_lock on;
            proxy_cache_min_uses 2;
            proxy_ignore_headers "X-Accel-Expires" "Cache-Control" "Expires";
            proxy_cache_use_stale updating error timeout invalid_header http_500 http_502 http_504;
            proxy_cache_background_update on;

            try_files $uri $uri/ /index.php?$args;

        }

  		# Static files location
  		location ~* \.(ttf|eot|svg|woff|woff2|css|js|json|ico|zip|tgz|gz|rar|bz2|doc|docx|xlsx|pptx|xls|exe|pdf|ppt|txt|tar|mid|midi|wav|bmp|rtf|avi|swf|flv|mp3|mp4|fla)$ {
                expires max;
        	include /etc/angie/static.conf;	
  		}

  		location ~* \.(jpg|jpeg|gif|png|ico)$ {
	        include /etc/angie/static.conf;
  		}

		location ~ \.php$ {
	        fastcgi_buffering off;
        	include fastcgi_params;
        	fastcgi_param SCRIPT_FILENAME /var/www/html/$fastcgi_script_name;
        	fastcgi_pass 127.0.0.1:9000;
        }

   		location /img {
	        include /etc/angie/static-avif.conf;
   		 }
		}

3. angie -t && service angie reload
   
4. проверка

   			root@angie01:~# curl -vI https://evg.mtdlb.ru
			* Host evg.mtdlb.ru:443 was resolved.
			* IPv6: (none)
			* IPv4: 51.250.106.241
			*   Trying 51.250.106.241:443...
			* Connected to evg.mtdlb.ru (51.250.106.241) port 443
			* ALPN: curl offers h2,http/1.1
			* TLSv1.3 (OUT), TLS handshake, Client hello (1):
			*  CAfile: /etc/ssl/certs/ca-certificates.crt
			*  CApath: /etc/ssl/certs
			* TLSv1.3 (IN), TLS handshake, Server hello (2):
			* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
			* TLSv1.3 (IN), TLS handshake, Certificate (11):
			* TLSv1.3 (IN), TLS handshake, CERT verify (15):
			* TLSv1.3 (IN), TLS handshake, Finished (20):
			* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
			* TLSv1.3 (OUT), TLS handshake, Finished (20):
			* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519 / id-ecPublicKey
			* ALPN: server accepted h2
			* Server certificate:
			*  subject: CN=evg.mtdlb.ru
			*  start date: Jul 14 15:46:45 2026 GMT
			*  expire date: Oct 12 15:46:44 2026 GMT
			*  subjectAltName: host "evg.mtdlb.ru" matched cert's "evg.mtdlb.ru"
			*  issuer: C=US; O=Let's Encrypt; CN=YE1
			*  SSL certificate verify ok.
			*   Certificate level 0: Public key type EC/prime256v1 (256/128 Bits/secBits), signed using ecdsa-with-SHA384
			*   Certificate level 1: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using ecdsa-with-SHA384
			*   Certificate level 2: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using ecdsa-with-SHA384
			*   Certificate level 3: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using ecdsa-with-SHA384
			* using HTTP/2
			* [HTTP/2] [1] OPENED stream for https://evg.mtdlb.ru/

# HTTP балансировка
1. запустим 4 контейнера webdebugger
      
			docker.yml:
			version: '3'
	
			services:
	  			debug-white:
	    			image: vscoder/webdebugger
	    			container_name: debug-white
	    			restart: unless-stopped
	    			environment:
	      				APP_DELAY: 0
	      				APP_PORT: 8080
	      				APP_BGCOLOR: white
	    			ports:
	      				- "9000:8080"
	  			debug-blue:
	    			image: vscoder/webdebugger
	    			container_name: debug-blue
	    			restart: unless-stopped
	    			environment:
	      				APP_DELAY: 0
	      				APP_PORT: 8080
	      				APP_BGCOLOR: skyblue
	    			ports:
	      				- "9001:8080"
	  			debug-green:
	    			image: vscoder/webdebugger
	    			container_name: debug-green
	    			restart: unless-stopped
	    			environment:
	      				APP_DELAY: 0
	      				APP_PORT: 8080
	      				APP_BGCOLOR: limegreen
	    			ports:
	      				- "9002:8080"      
	  			debug-gold:
	    			image: vscoder/webdebugger
	    			container_name: debug-gold
	    			restart: unless-stopped
	    			environment:
	      				APP_DELAY: 0
	      				APP_PORT: 8080
	      				APP_BGCOLOR: gold
	    			ports:
	      				- "9003:8080"
	
			docker compose -f docker.yml up -d
2. Балансировка round-robin (by default)

				/etc/angie/sites-available/balance
					upstream backend {
				    zone upstream-backend 256k;
	    			server 127.0.0.1:9000 sid=white slow_start=120s;
	    			server 127.0.0.1:9001 sid=blue slow_start=120s;
	    			server 127.0.0.1:9002 sid=green slow_start=120s;
	    			server 127.0.0.1:9003 sid=gold slow_start=120s;
					}
	
					server {
	        			listen 80 default_server;
	        			server_name _;
	
	        			location / {
			                proxy_pass http://backend;
				        }
	
	        			location /favicon {
	        			}
					}
3. Балансировка по хэшу $request_uri

	  			/etc/angie/sites-available/balance
					upstream backend {
				    zone upstream-backend 256k;
      				hash $request_uri consistent; 
	    			server 127.0.0.1:9000 sid=white;
	    			server 127.0.0.1:9001 sid=blue;
	    			server 127.0.0.1:9002 sid=green;
	    			server 127.0.0.1:9003 sid=gold;
					}
	
					server {
	        			listen 80 default_server;
	        			server_name _;

						add_header X-Backend-Server $upstream_addr always;
						
	        			location / {
			                proxy_pass http://backend;
				        }
	
	        			location /favicon {
	        			}
					}
4. Проверка

      			curl -sI localhost/asdfg | grep "X-Backend-Server"
				X-Backend-Server: 127.0.0.1:9002

				curl -sI localhost/asdert34 | grep "X-Backend-Server"
				X-Backend-Server: 127.0.0.1:9000

				curl -sI localhost/wdxcvb123 | grep "X-Backend-Server"
				X-Backend-Server: 127.0.0.1:9001

				curl -sI localhost/qsxc23fg | grep "X-Backend-Server"
				X-Backend-Server: 127.0.0.1:9003

5. Произвольная балансировка

				/etc/angie/sites-available/balance
					upstream backend {
				    zone upstream-backend 256k;
    				random two least_conn;
	    			server 127.0.0.1:9000 sid=white;
	    			server 127.0.0.1:9001 sid=blue down;
	    			server 127.0.0.1:9002 sid=green down;
	    			server 127.0.0.1:9003 sid=gold;
					}
	
					server {
	        			listen 80 default_server;
	        			server_name _;

						add_header X-Backend-Server $upstream_addr always;
						
	        			location / {
			                proxy_pass http://backend;
				        }
	
	        			location /favicon {
	        			}
					}    			

6. Проверка

    		curl -sI localhost | grep "X-Backend-Server"
			X-Backend-Server: 127.0.0.1:9003
			curl -sI localhost | grep "X-Backend-Server"
			X-Backend-Server: 127.0.0.1:9003
			curl -sI localhost | grep "X-Backend-Server"
			X-Backend-Server: 127.0.0.1:9003
			curl -sI localhost | grep "X-Backend-Server"
			X-Backend-Server: 127.0.0.1:9000
			curl -sI localhost | grep "X-Backend-Server"
			X-Backend-Server: 127.0.0.1:9003
			curl -sI localhost | grep "X-Backend-Server"
			X-Backend-Server: 127.0.0.1:9000
   			curl -sI localhost | grep "X-Backend-Server"
			X-Backend-Server: 127.0.0.1:9000
			curl -sI localhost | grep "X-Backend-Server"
			X-Backend-Server: 127.0.0.1:9003
   			curl -sI localhost | grep "X-Backend-Server"
			X-Backend-Server: 127.0.0.1:9000

7. Резервный бэкенд

			/etc/angie/sites-available/balance
   			upstream backend {
    			zone upstream-backend 256k;
			    server 127.0.0.1:9000 weight=4 sid=white;
    			server 127.0.0.1:9001 sid=blue backup;
    			server 127.0.0.1:9002 sid=green down;
    			server 127.0.0.1:9003 weight=2 fail_timeout=1s max_fails=1 sid=gold;
    			sticky route $arg_route;
    			sticky_strict on;
			}

			server {
        		listen 80 default_server;
		        server_name _;

        		location / {
                add_header X-Backend-Server "$upstream_addr" always;

                proxy_pass http://backend;

		    	}

        		location /favicon {
        		}

        		location /status/ {
          			api /status/;
     	   		}
			}

8. Проверка

			curl -sI localhost/?route=white | grep "X-Backend-Server"
			X-Backend-Server: 127.0.0.1:9000
			curl -sI localhost/?route=gold | grep "X-Backend-Server"
			X-Backend-Server: 127.0.0.1:9003

			curl -I localhost/?route=green
			HTTP/1.1 502 Bad Gateway
			Server: Angie/1.11.8
			Date: Fri, 24 Jul 2026 20:31:38 GMT
			Content-Type: text/html
			Content-Length: 157
			Connection: keep-alive
			X-Backend-Server: backend

# Защита от DoS
1. Настройка ограничений частоты запросов
  
      		В секции http добавить:
      		limit_req_zone $binary_remote_addr zone=lone:10m rate=10r/s;
      		В секции server добавить:
      		limit_req zone=lone burst=50 nodelay;
   			limit_req_log_level error;
			limit_req_status 503;

2. Проверка ограничений

    		wrk -d20s -c 10 http://localhost

			tail /var/log/angie/error.log
				2026/08/02 14:42:21 [error] 13957#13957: *8235 limiting requests, excess: 50.620 by zone "lone", client: 127.0.0.1, server: _, request: "GET / HTTP/1.1", host: "localhost"
				2026/08/02 14:42:21 [error] 13956#13956: *8227 limiting requests, excess: 50.620 by zone "lone", client: 127.0.0.1, server: _, request: "GET / HTTP/1.1", host: "localhost"
				2026/08/02 14:42:21 [error] 13956#13956: *8229 limiting requests, excess: 50.620 by zone "lone", client: 127.0.0.1, server: _, request: "GET / HTTP/1.1", host: "localhost"
				2026/08/02 14:42:21 [error] 13957#13957: *8231 limiting requests, excess: 50.620 by zone "lone", client: 127.0.0.1, server: _, request: "GET / HTTP/1.1", host: "localhost"
				2026/08/02 14:42:21 [error] 13956#13956: *8215 limiting requests, excess: 50.620 by zone "lone", client: 127.0.0.1, server: _, request: "GET / HTTP/1.1", host: "localhost"

3. Basic HTTP аутентификация

			Установить htpasswd
   			apt install apache2-utils

   			Создание пользователя
			htpasswd -c /etc/angie/htpasswd user

   			Добавить в секцию server:
			auth_basic "Identify yourself";
			auth_basic_user_file /etc/angie/htpasswd;

   			Проверка 
			curl -v http://locahost
			* Connected to localhost (127.0.0.1) port 80
			> GET / HTTP/1.1
			> Host: localhost
			> User-Agent: curl/8.5.0
			> Accept: */*
			>
			< HTTP/1.1 401 Unauthorized
			< Server: Angie/1.11.8
			< Date: Sun, 02 Aug 2026 11:55:33 GMT
			< Content-Type: text/html
			< Content-Length: 159
			< Connection: keep-alive
			< WWW-Authenticate: Basic realm="Identify yourself"

4. Ограничение по ip и логину
   
			Добавить в секцию server:
 			satisfy all; # (включить оба варианта)
			allow 127.0.0.1
			deny all;

6. Проверка
   
			tail /var/log/angie/error.log
			2026/08/02 14:54:54 [error] 19245#19245: *9456 access forbidden by rule, client: 87.117.189.60, server: _, request: "GET / HTTP/1.1", host: "evg.mtdlb.ru"
			2026/08/02 14:54:56 [error] 19245#19245: *9456 access forbidden by rule, client: 87.117.189.60, server: _, request: "GET / HTTP/1.1", host: "evg.mtdlb.ru"
			2026/08/02 14:54:56 [error] 19245#19245: *9456 access forbidden by rule, client: 87.117.189.60, server: _, request: "GET / HTTP/1.1", host: "evg.mtdlb.ru"
			2026/08/02 14:54:57 [error] 19245#19245: *9456 access forbidden by rule, client: 87.117.189.60, server: _, request: "GET / HTTP/1.1", host: "evg.mtdlb.ru"
			2026/08/02 14:55:03 [error] 19245#19245: *9456 access forbidden by rule, client: 87.117.189.60, server: _, request: "GET /console/ HTTP/1.1", host: "evg.mtdlb.ru"
   
7. Fail2ban

			apt install fail2ban
   			cd /etc/fail2ban
  			cp jail.conf jail.local
 			nano /etc/fail2ban/jail.local
			[nginx-limit-req]
				port = http, https
				enabled = true
				filter = nginx-limit-req
				action = iptabled-multiport[name=RegLimit, port="http,https", protocol=tcp]  // создается цепочка правил f2b-RegLimit
				logpath = /var/log/angie/*error.log
				findtime = 600 
				bantime = 7200 
				maxretry =4 
   			
			/etc/fail2ban/filter.d/nginx-limit-req.conf
   			failregex = ^\s*\[[a-z]+\] \d+#\d+: \*\d+ limiting requests, excess: [\d\.]+ by zone "(?:%(ngx_limit_req_zones)s)", client: "<HOST>",

			service fail2ban reload

   			Проверка:
			root@angie01:/etc/fail2ban# fail2ban-client status
			Status
				|- Number of jail:      2
				`- Jail list:   nginx-limit-req, sshd
	
			root@angie01:/etc/fail2ban# fail2ban-client status nginx-limit-req
			Status for the jail: nginx-limit-req
				|- Filter
				|  |- Currently failed: 0
				|  |- Total failed:     0
				|  `- File list:        /var/log/angie/wordpress-error.log /var/log/angie/error.log
				`- Actions
	   			|- Currently banned: 0
	   			|- Total banned:     0
	   			`- Banned IP list:

8. 	Блокировка ботов по User-Agent

			Добавить в секцию http:
			map $http_user_agent $limit_search_bots {
				default 0;
				~*(BlackWidow|ChinaClaw|Custo|DISCo|Download|Demon|eCatch|EirGrabber|ClaudeBot) 1;
				~*(Express|WebPictures|ExtractorPro|EyeNetIE|FlashGet|GetRight|GetWeb!) 1;
				~*(rafula|HMView|HTTrack|Stripper|Sucker|Indy|InterGET|Ninja|JetCar|Spider) 1;
				~*(GrabNet|NetSpider|Vampire|NetZIP|Octopus|Offlie|PageGrabber|Foto|pavuk) 1;
				~*(Teleport|VoidEYE|Collector|WebAuto|WebCopier|WebFetch|WebGo|WebLecher|WebReaper) 1;
				~*(Twengabot|htmlparser|libwww|Python|perl|urllib|scan|email|PycURL|Pyth|PyQ) 1;
			}
	
			Добавить в секцию server:
			If ($limit_search_bots = 1) {
				return 421; 
			}

9. Окончательный server конфиг

			limit_req_zone $binary_remote_addr zone=lone:10m rate=10r/s;

			upstream backend {
    		zone upstream-backend 256k;
		    server 127.0.0.1:9000 sid=white;
    		server 127.0.0.1:9001 sid=blue;
    		server 127.0.0.1:9002 sid=green;
    		server 127.0.0.1:9003 sid=gold;
			}

			server {
        		listen 80 default_server;
        		server_name evg.mtdlb.ru;
        		return 302 https://$host$request_uri;
			}

			server {
        		listen 443 ssl;

        		http2 on;

        		status_zone balance;
		        server_name evg.mtdlb.ru;

        		auth_basic "Identify yourself";
        		auth_basic_user_file /etc/angie/htpasswd;

				satisfy any;
        		allow 127.0.0.1;
		        allow 87.117.189.60;
        		deny all;

        		add_header Strict-Transport-Security max-age=300;

        		ssl_session_cache shared:SSL:10m;
        		ssl_session_timeout 28h;

        		ssl_session_tickets on;

        		ssl_certificate /etc/letsencrypt/live/evg.mtdlb.ru/fullchain.pem;
        		ssl_certificate_key /etc/letsencrypt/live/evg.mtdlb.ru/privkey.pem;

        		ssl_protocols TLSv1.2 TLSv1.3;
        		ssl_prefer_server_ciphers on;
        		ssl_ciphers HIGH:!aNULL:!MD5;

        		location / {
                add_header X-Backend-Server "$upstream_addr" always;

                proxy_pass http://backend;

                upstream_probe backend_probe
                        uri=/probe
                        interval=5s
                        test=$good
                        essential
                        persistent
                        fails=3
                        passes=3
                        max_body=10m
                        mode=idle;

		        }

				location /console/ {

		        auto_redirect on;

        		alias /usr/share/angie-console-light/html/;
        		index index.html;

        		location /console/api/ {
            		api /status/;
        		}

        		}

		        location /favicon {
        		}

        		location /status/ {
          		api /status/;
        		}

		       limit_req zone=lone burst=50 nodelay;
   			    limit_req_log_level error;
   			    limit_req_status 503;

        		if ($limit_search_bots = 1) {
                	return 421;
        		}

			}




   			



	










   
			
			
			
			
				
 


   		 

   

