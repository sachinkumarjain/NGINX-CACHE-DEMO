# NGINX-CACHE-DEMO
NGINX-CACHE-DEMO with PCF for static file and caching

cf login -a pcf-url -o org -s space -u user
cf push

* https://nginx-demo.ausvdc02.pcf.dell.com/var/www/locale.xml - caching
* https://nginx-demo.ausvdc02.pcf.dell.com/index.html - no-cache

----------------------------------------------------------------------
nginx.conf [https://www.nginx.com/resources/wiki/start/topics/examples/full/#nginx-conf]

worker_processes 2;
daemon off;
error_log  logs/error.log;
pid        logs/nginx.pid;
worker_rlimit_nofile 8192;

events { worker_connections 1024; }

http {
  log_format cloudfoundry '$http_x_forwarded_for - $http_referer - [$time_local] "$request" $status $body_bytes_sent';
  default_type application/octet-stream;
  include mime.types;
  sendfile on;
  gzip on;
  tcp_nopush on;
  keepalive_timeout 60;

  server {
    listen {{port}};
    server_name ausvdc02.pcf.dell.com .ausvdc02.pcf.dell.com;
	access_log   logs/ausvdc02.access.log;
	
    location / {
      root /app;
      index index.html;
      proxy_redirect off;

      proxy_set_header   Host              $http_host;
      proxy_set_header   X-Real-IP         $remote_addr;
      proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
      proxy_set_header   X-Forwarded-Proto https;

    }
  }
}

buildpack.yml

---
nginx:
  version: 1.18.0
  plaintext_env_vars:
    - "OVERRIDE"

mime.types

types {
  text/html                             html htm shtml;
  text/css                              css;
  text/xml                              xml rss;
  image/gif                             gif;
  image/jpeg                            jpeg jpg;
  application/x-javascript              js;
  text/plain                            txt;
 }


manifest.yml

---
applications:
- name: nginx-demo
  path: app
  no-route: false
  no-start: false
  memory: 1G
  disk_quota: 1G
  instances: 1
  stack: cflinuxfs3
  buildpacks: 
  - staticfile_buildpack
  - nginx_buildpack
  health-check-type: process
  env:
    Service_Environment: dev
    CACHE_NUGET_PACKAGES: false


https://www.digitalocean.com/community/tutorials/how-to-implement-browser-caching-with-nginx-s-header-module-on-centos-7

curl -i -H 'If-None-Match:"5f6c2ad3-6c"' https://nginx-demo.ausvdc02.pcf.dell.com/index.html

curl -I -H 'If-None-Match:"5f6c3f2b-88553"' https://nginx-demo.ausvdc02.pcf.dell.com/var/www/locale.xml -H content-type:text/xml

