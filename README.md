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
 


