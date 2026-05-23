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

 


