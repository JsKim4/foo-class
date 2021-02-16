==================== nginx-load-balancing ====================

 install cli
 
        sudo yum install nginx
        
  
 jenkins-instance 의 설정탭에서 SSH Servers 추가
 
 이전에 등록한 ITEM 에서 방금 등록한 SSH Server 추가
 
 - docker 오류 발생시 => (cpu-instance)도커 명령어를 못찾는 경우 도커 설치 확인 및 실행
 
 nginx-instance 에서 /etc/nginx/nginx.config 파일을 다음과 같이 수정
 
        user nginx;
        worker_processes auto;
        error_log /var/log/nginx/error.log;
        pid /run/nginx.pid;
        # Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
        include /usr/share/nginx/modules/*.conf;
        events {
            worker_connections 1024;
        }
        http {
            log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                              '$status $body_bytes_sent "$http_referer" '
                              '"$http_user_agent" "$http_x_forwarded_for"';
            access_log  /var/log/nginx/access.log  main;
            sendfile            on;
            tcp_nopush          on;
            tcp_nodelay         on;
            keepalive_timeout   65;
            types_hash_max_size 2048;
            include             /etc/nginx/mime.types;
            default_type        application/octet-stream;
            # Load modular configuration files from the /etc/nginx/conf.d directory.
            # See http://nginx.org/en/docs/ngx_core_module.html#include
            # for more information.
            include /etc/nginx/conf.d/*.conf;
            upstream cpu-bound-app {
                server {cpu-instance-host-1}:8080 weight=100 max_fails=3 fail_timeout=3s;
                server {cpu-instance-host-2}:8080 weight=100 max_fails=3 fail_timeout=3s;
                server {cpu-instance-host-3}:8080 weight=100 max_fails=3 fail_timeout=3s;
            }
            server {
                listen       80 default_server;
                listen       [::]:80 default_server;
                server_name  _;
                root         /usr/share/nginx/html;
                # Load configuration files for the default server block.
                include /etc/nginx/default.d/*.conf;
                location / {
                    proxy_pass http://cpu-bound-app;
                    proxy_http_version 1.1;
                    proxy_set_header Upgrade $http_upgrade;
                    proxy_set_header Connection 'upgrade';
                    proxy_set_header Host $host;
                    proxy_cache_bypass $http_upgrade;
                }
                error_page 404 /404.html;
                location = /404.html {
                }
                error_page 500 502 503 504 /50x.html;
                location = /50x.html {
                }
            }
        # Settings for a TLS enabled server.
        #
        #    server {
        #        listen       443 ssl http2 default_server;
        #        listen       [::]:443 ssl http2 default_server;
        #        server_name  _;
        #        root         /usr/share/nginx/html;
        #
        #        ssl_certificate "/etc/pki/nginx/server.crt";
        #        ssl_certificate_key "/etc/pki/nginx/private/server.key";
        #        ssl_session_cache shared:SSL:1m;
        #        ssl_session_timeout  10m;
        #        ssl_ciphers HIGH:!aNULL:!MD5;
        #        ssl_prefer_server_ciphers on;
        #
        #        # Load configuration files for the default server block.
        #        include /etc/nginx/default.d/*.conf;
        #
        #        location / {
        #        }
        #
        #        error_page 404 /404.html;
        #        location = /404.html {
        #        }
        #
        #        error_page 500 502 503 504 /50x.html;
        #        location = /50x.html {
        #        }
        #    }
        }
        
아래 명령어로 nginx 에러 로그 확인

        sudo tail -f /var/log/nginx/error.log
        
아래의 명령어로 해결함. 

        sudo setsebool -P httpd_can_network_connect on
        
        CentOs 5.6 이상 버전 부터 외부 네트워크를 사용하는것을 막아 놓는것이 기본 설정인데
        해당 옵션을 변경해 주는 명령어
        

   
        
     