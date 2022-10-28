# Server_NgninxFundamentalsHighPerformanceServersFromScratch

## NgninX vs Apache
### NginX
 - URI를 기준으로 파일을 찾는다
### Apache
 - File system을 기준으로 파일을 찾는다

### Initial setting(Ubuntu)
```
# apt-get update
# apg-get install -y nginx
# ps aux | grep nginx # au: all users, x: including boot section
# ll /etc/nginx # nginx 폴더 확인

```

#### how to change security key
 - 동일 아이피에 다른 pem키를 사용하여 접속하려고 하면 known ssh 파일에서 오류를 띄운다. 기존꺼 삭제해야 함
```
# ssh-keygen -R 주소 # 기존ip에 키가 있을때 삭제하는 법
```

### Initial setting(Centos7)
```
# yum install epel-release
# yum install nginx
# ll /etc/nginx # nginx 폴더 확인
# ps aux | grep nginx # au: all users, x: including boot section, centos에서는 자동으로 nginx를 등록하지 않으므로 수동으로 해줘야 한다
# service nginx start
```


## 소스를 받아서 빌드하는 방법
 - nginx.org에서 download탭에서 소스 주소를 복사한다

![image](https://user-images.githubusercontent.com/22423285/195734411-49a9ff69-6569-4a11-9841-c0664b2c300b.png)

 - ubuntu에서 wget으로 소스를 다운받고 그 폴더로 들어간다
```
# cd
# wget https://nginx.org/download/nginx-1.23.1.tar.gz
# tar - zxvf nginx-1.23.1.tar.gz
# cd nginx-1.23.1
```
 - 소스를 다운 받았으므로 컴파일한다. 소스 폴더 내에서 설정 실행
```
# ./configure
```
 - 사용하려고 할때에 C 컴파일러가 없다면 인스톨 필요
    + ubuntu
```
# apt-get install build-essential
```
    + centos 
```
# yum groupinstall "Development Tools"
```

 - 하다가 안될때에 dependency package를 설치해야 한다
    + ubuntu
```
# apt-get install libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev
```
    + centos
```
# yum install pcre pcre-devel zlib zlib-devel openssl openssl-devel
```
 - nginx 세부설정하여 다시 설정하기
 - 하기전에 "./configure --help"로 사용가능 확인 가능
    + https://nginx.org/en/docs/configure.html
```
# ./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module
```
 - 컴파일하기
```
# make
```
 - 컴파일 된 파일을 설치하고 기동하기
```
# make install
```
 - 폴더 확인
```
# ll /etc/nginx
```
 - 버전 확인
```
# nginx -V
```
 - nginx 기동
```
# nginx
```
 - check process
```
# ps aux | grep nginx
```

### nginx 종료
```
# nginx -s stop
```

#### nginx 스크립트
 - https://www.nginx.com/nginx-wiki/build/dirhtml/start/topics/examples/initscripts/

### systemd를 활용한 nginx 시스템 실행/중지
 - ubuntu17.0 or centos7 이상
 - 설정 안내 사이트: https://www.nginx.com/nginx-wiki/build/dirhtml/start/topics/examples/systemd/
 - 먼저 /lib/systemd/system/nginx.service 경로에 파일을 만든다
```
# touch /lib/systemd/system/nginx.service
```
 - vim으로 스크립트를 붙여 넣고 저장(페이지에서 가져온거 경로 수정)
```
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/bin/nginx -t
ExecStart=/usr/bin/nginx
ExecReload=/usr/bin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

 - systemd로 실행
```
# systemctl start nginx
```
 - systemd로 상태 확인
```
# systemctl status nginx
```
 - systemd로 종료
```
# systemctl stop nginx
```
 - 리눅스 재부팅시 자동 실행되도록 설정
```
 - systemctl enable nginx
```

## 설정
 - 설정파일: nginx.conf
 - 위치: /etc/nginx/nginx.conf
### 용어 정리
 - Directive
    + nginx의 설정은 directive의 모음으로 이루어져 있다.
    + name value의 형식을 가진다. value는 {}로 묶여진 block이 될 수도 있다.
    + 상위에 설정된 directive는 하위의 context에 영향을 미친다.
    + context에서 재정의 가능
    + 하위에 
 - Context
    + directive value의 block으로 묶여진 부분을 뜻한다.


### Virtual Host
 - nginx기동 후 기본 페이지에 보여주는 것에서 내가 원하는 사이트를 기등하는 것
 - 폴더를 하나 만든다
```
# ll /sites/demo
```
 - 그리고 설정 파일을 수정한다 (/etc/nginx/nginx.conf)
 - 기존에것은 다 지우고 일단은 아래꺼 적어서 동작하는지 확인
```
events {}


http {
	server {
		listen 80;
		server_name 13.125.215.175;

		# connect file system to uri from static requests
		root /sites/demo;
	}
}
```
 - nginx의 바뀐 설정파일이 구동가능한지 확인
```
# nginx -t
```
 - nginx 재시작
```
# systemctl reload nginx
```
    
### CSS 잘 안될때
 - mime타입이 제대로 안 되서 그럴 수 있음
 - 아래의 명령어를 해보면 Content-Type이 text/plaind으로 되어 있음 
```
# curl -I http://11.1.1.1/style.css # header 요청만 받음
```
 - nginx.conf파일을 수정해야 함
```
events {}


http {
        types {
                text/html html; # 이렇게 수동으로 설정해 줘도 된다
                text/css css;
        }

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;
        }
}
```
 - 하지만 /etc/nginx/mime.types폴더 안에 이미 대부분의 것들이 정의되어 있다. 따라서 include하면 된다
```
events {}


http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;
        }
}

```
 - 설정 하고 리로드 해줘야 한다
```
# systemctl reload nginx
```

### Location Blocks
 - 특정 URI 요청에 대한 리턴값을 지정할 수 있다
 - nginx.conf의 http-server context내에 설정한다
    + Prefix match: 해당하는 것으로 시작하는 것은 모두 리턴한다(아래에서 /greet /greeting /greet1)
```
events {}


http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                location /greet {
                        return 200 'Hello from Nginx "/greet" location';

                }
        }
}
```
    + Exact match: =을 붙여준다
    + Regex match: ~를 붙여준다
    + Regex match insensitive: ~*를 붙여준다
```
events {}


http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                # Prefix match
                # location /greet {
                #       return 200 'Hello from Nginx "/greet" location';
                # }

                # Exact match
                # location = /greet {
                #       return 200 'Hello from Nginx "/greet" location - Exact';
                # }

                # REGEX match case sensitive /greet1(O), /Greet1(X)
                # location ~ /greet[0-9] {
                #       return 200 'Hello from Nginx "/greet" location - Regex';
                # }

                # REGEX match case insensitive /greet1(O), /Greet1(O)
                location ~* /greet[0-9] {
                        return 200 'Hello from Nginx "/greet" location - Regex Insensitive';
                }
        }
}
```
 - 표시순서
    + 중복되는 규칙에서 정규식이 우선순위를 가진다.
    + 중복되는 규칙에서 정규식보다 높은 우선순위를 가지고 싶다면 ^~를 붙여주면 된다
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                # Prefix match
                # location /greet {
                #       return 200 'Hello from Nginx "/greet" location';
                # }

                # Exact match
                # location = /greet {
                #       return 200 'Hello from Nginx "/greet" location - Exact';
                # }

                # REGEX match case sensitive /greet1(O), /Greet1(X)
                # location ~ /greet[0-9] {
                #       return 200 'Hello from Nginx "/greet" location - Regex';
                # }

                # REGEX match case insensitive /greet1(O), /Greet1(O)
                location ~* /greet[0-9] {
                        return 200 'Hello from Nginx "/greet" location - Regex Insensitive';
                }

                # Prefix match
                location ^~ /greet2 {
                        return 200 'Hello from Nginx "/greet" location';
                }
        }
}

```
 - 표시순서 우선순위 순서
    + Exact Match: =
    + Preferential Prefix Match ^~
    + REGEX Match ~*
    + Prefix Match
    
### Variables
 - Build in variables
   + https://nginx.org/en/docs/varindex.html
 - 사용자 커스텀 변수
```
set $var 'something';
```
    
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                location /inspect {
                        return 200 "$host\n$uri\n$args";
                }
        }
}
```
 - 파라미터 변수를 활용하고 싶다면 built-in변수를 쓴다
    + $arg_파라미터명
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                location /inspect {
                        return 200 "Name: $arg_name";
                }
        }
}
```
### if statement 사용
 - static apikey 예시
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                # Check static api key
                if ( $arg_apikey != 1234) {
                        return 401 "Incorrect API Key";
                }

                location /inspect {
                        return 200 "Name: $arg_name";
                }
        }
}
```
 - 화요일인지 체크
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                # Check static api key
                set $weekend 'No';

                # check if weekend
                if ( $date_local ~ 'Tuesday' ) {
                        set $weekend 'Yes';
                }

                location /is_weekend {
                        return 200 $weekend;
                }
        }
}
```
 
### 경로 분기
 - rewrite
    + URI 경로는 바뀌지 않는다.
    + user이라는 경로로 시작하면 메시지 반환 예제
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                rewrite ^/user/\w+ /greet;

                location /greet{
                        return 200 "Hello User";
                }
        }
}
```
     + John만 특별한 경로 표시
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                rewrite ^/user/(\w+) /greet/$1;

                location /greet{
                        return 200 "Hello User";
                }

                location = /greet/john {
                        return 200 "Hello John";
                }
        }
}
```
     + 2개의 rewrite가 있을 때에는 둘다 일어난다
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                rewrite ^/user/(\w+) /greet/$1; # first move to /greet/john
                rewrite ^/greet/john /thumb.png; # and then move to thumb.png again. Two rewrites will occure step by step.

                location /greet{
                        return 200 "Hello User";
                }

                location = /greet/john {
                        return 200 "Hello John";
                }
        }
}                                                                                                                                    
```
     + If I put the "last keyword" at the end of the rewrite, then rewrite chain will stop to proceed.
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                rewrite ^/user/(\w+) /greet/$1 last; # break the rewrite chain
                rewrite ^/greet/john /thumb.png;

                location /greet{
                        return 200 "Hello User";
                }

                location = /greet/john {
                        return 200 "Hello John";
                }
        }
}
```

 - return
    + return state 가 300번때이면 경로를 바꿔버린다. 경로 자체가 바뀐다(/logo -> /thumb.png)
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;
                
                location /logo {
                        return 307 /thumb.png;
                }
        }
}

```
    
### try_files path1 path2 final;
 - 먼저 path1을 호출하고 없으면 path2를 호출한다
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                try_files /thumb.png /greet;


                location /greet{
                        return 200 "Hello User";
                }
        }
}
```
 - $uri변수를 붙이면 있는것은 그냥 사용한다. 없으면 순차적으로 다음것을 호출한다
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                try_files $uri /cat.png /greet;


                location /greet{
                        return 200 "Hello User";
                }
        }
}
```
 - 없으면 404에러를 rewrite가능하다
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                try_files $uri /cat.png /greet /friendly_404;

                location /friendly_404 {
                        return 404 "Sorry, that file could not be found";
                }

                location /greet{
                        return 200 "Hello User";
                }
        }
}
```

## Logging
 - 서버 생성할 때에 로그 폴더를 따로 지장했다
    + --error-log-path=/var/log/nginx/error.log -- http-log-path=/var/log/nginx/access.log
    + 로그 저장 폴더: /var/log/nginx
 - Error log
    + 보통 404를 에러로그라고 생각하는데 에러로그 아니다. 안찍힘 
 - Access log
 - 테스트 전에 먼저 로그를 날려준다
```
# cd /var/log/nginx
# echo '' > access.log
# echo '' > error.log
```
 - 만약 nginx.conf파일에서 syntex 오류가 있고, systemctl reload nginx 했을때 오류가 있다면 error.log에 찍힌다
 - 특정 로그를 만들려면 nginx.conf파일에 기록해야 한다
    + 시스템을 재시작하면 자동으로 secure.access.log라는 파일을 만든다
    + 2개 이상도 가능하다
    + 아래 예제는 access_log를 2개 찍는것
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                location /secure {
                        access_log /var/log/nginx/secure.access.log;
			access_log /var/log/nginx/access.log;
                        return 200 "Welcome to secure area.";
                }

        }
}
```

 - 로그를 안찍을 수도 있다
    + 만약 트래픽이 많으면 끈다
    + off 키워드 사용
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                location /secure {
                        access_log off;
                        return 200 "Welcome to secure area.";
                }

        }
}
```
 - 압축 등 옵션이 많으니 찾아서 하면 된다


## Directive
 - 3가지 구별 예제
```
events {}

######################
# (1) Array Directive
######################
# Can be specified multiple times without overriding a previous setting
# Gets inherited by all child contexts
# Child context can override inheritance by re-declaring directive
access_log /var/log/nginx/access.log;
access_log /var/log/nginx/custom.log.gz custom_format;

http {

  # Include statement - non directive
  include mime.types;

  server {
    listen 80;
    server_name site1.com;

    # Inherits access_log from parent context (1)
  }

  server {
    listen 80;
    server_name site2.com;

    #########################
    # (2) Standard Directive
    #########################
    # Can only be declared once. A second declaration overrides the first
    # Gets inherited by all child contexts
    # Child context can override inheritance by re-declaring directive
    root /sites/site2;

    # Completely overrides inheritance from (1)
    access_log off;

    location /images {

      # Uses root directive inherited from (2)
      try_files $uri /stock.png;
    }

    location /secret {
      #######################
      # (3) Action Directive
      #######################
      # Invokes an action such as a rewrite or redirect
      # Inheritance does not apply as the request is either stopped (redirect/response) or re-evaluated (rewrite)
      return 403 "You do not have permission to view this.";
    }
  }
}
```

## PHP Processing
 - installing php
```
# apt-get update
# apt-get install php-fpm # fpm keyword installs move latest stable release of php- fastcgi protocol
# systemctl list-units | grep php
# systemctl status php8.1-fpm
```

### Setting index directive
 - 루트경로로 요청이 왔을 때 보여줄 페이지의 위치를 결정한다.
 - fastcgi를 사용하여 설정. 해당 설정은 존재해서 include해서 사용
 - fastcgi에 대한 nginx설정은 /etc/nginx/fastcgi.conf
 - 먼저 php의 fpm소켓에 대한 파일을 찾는것이 필요하다
```
# find / -name *fpm.sock
```
 - 설정하기
```
http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                index index.php index.html;

                location / {
                        try_files $uri $uri/ =404;
                }

                location ~\.php$ {
                        # Pass php requests to the php-fpm service (fastcgi)
                        include fastcgi.conf;
                        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
                }

        }
}
```
 - 이후 php파일을 임시로 만들어 준다
```
# echo '<?php phpinfo(); ?>' > /sites/demo/info.php
```
 - 브라우저로 가서 ip를 치면 그냥 기존의 static 페이지가 표시된다.
    + 이는 현재 "index index.php index.html" directive를 통하여 index.php를 찾았지만 아직 안만들어서 index.html을 표시해서 그렇다
 - 13.125.215.xxx/info.php를 친다
    + 502 bad gateway가 뜬다
    + 에러 로그를 통하여 문제가 먼지 파악한다
```
# tail -n 1 /var/log/nginx/error.log
```
   + 오류 이유: Permission error
```
2022/10/22 00:21:00 [crit] 43115#0: *5 connect() to unix:/run/php/php8.1-fpm.sock failed (13: Permission denied) while connecting to upstream, client: 106.102.128.150, server: 13.125.215.175, request: "GET /info.php HTTP/1.1", upstream: "fastcgi://unix:/run/php/php8.1-fpm.sock:", host: "13.125.215.175"
```
 - process 확인
    + 확인을 해보면 nginx의 user와 php의 user가 달라서 그렇다
```
# ps aux | grep nginx
# ps aux | grep php
```
   + 결과
```
root@ip-172-31-7-40:/home/ubuntu# ps aux | grep nginx
root       42932  0.0  0.3   9904  3060 ?        Ss   00:01   0:00 nginx: master process /usr/bin/nginx
nobody     43115  0.0  0.3  10440  3428 ?        S    00:18   0:00 nginx: worker process
root       43129  0.0  0.2   7004  2196 pts/1    S+   00:25   0:00 grep --color=auto nginx
root@ip-172-31-7-40:/home/ubuntu# ps aux | grep php
root       42780  0.0  1.9 202684 18884 ?        Ss   00:00   0:00 php-fpm: master process (/etc/php/8.1/fpm/php-fpm.conf)
www-data   42781  0.0  0.7 203156  7036 ?        S    00:00   0:00 php-fpm: pool www
www-data   42782  0.0  0.7 203156  7036 ?        S    00:00   0:00 php-fpm: pool www
root       43131  0.0  0.2   7004  2192 pts/1    S+   00:26   0:00 grep --color=auto php
```

 - user를 동일하게 설정
    + nginx.conf에서 user를 php에 맞춰준다.
```
user www-data;

events {}


http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                index index.php index.html;

                location / {
                        try_files $uri $uri/ =404;
                }

                location ~\.php$ {
                        # Pass php requests to the php-fpm service (fastcgi)
                        include fastcgi.conf;
                        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
                }

        }
}
```
 - index.php 만들기
```
# echo '<h1>Date: <?php echo date("l jS F"); ?></h1>' > /sites/demo/index.php
```

#### ※ FastCGI: html같은 프로토콜인데 동적인 데이터를 전달할때 스레드풀을 사용하여 만들어져 있는 스레드를 사용하여 cgi보다 빠르게 동작한다.


### Worker Porcess
 - "systemctl status nginx"해보면 프로세스가 master, worker2개가 나온다
 - master: acutual nginx server. spawn worker processes
 - worker: listen for and respond to client request
    + default number: 1
    + worker_process 다이렉티브로 개수를 바꿀 수 있다
    + 하나의 cpu core수만큼 worker process 개수를 맞춰주는 것이 좋다
    + cpu가 하나의 코어인데 2개의 worker processes를 기동하면 각각 50%만큼의 퍼포먼스를 보이고, 그만큼 더 자원을 소모한다
    + nginx에는 비동기로 동작하기 때문에 하나의 cpu에 하나의 worker만 있으면 된다
    + cpu수는 lscpu 또는 nproc 리눅스 명령어로 확인 가능하다
```
 user www-data;

worker_processes 2;

events {}


http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                index index.php index.html;

                location / {
                        try_files $uri $uri/ =404;
                }

                location ~\.php$ {
                        # Pass php requests to the php-fpm service (fastcgi)
                        include fastcgi.conf;
                        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
                }

        }
}
```
 - auto로 하면 알아서 해준다
```
user www-data;

worker_processes auto;

events {}


http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                index index.php index.html;

                location / {
                        try_files $uri $uri/ =404;
                }

                location ~\.php$ {
                        # Pass php requests to the php-fpm service (fastcgi)
                        include fastcgi.conf;
                        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
                }

        }
}
```

#### Event directive
 - worker_connections를 통해서 동시에 파일에 접근 가능한 수를 설정한다
 - 리눅스 명렁어 ulimit -n의 숫자만큼 동시접근 가능하다
```
user www-data;

worker_processes auto;

events {
        worker_connections 1024;
}


http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                index index.php index.html;

                location / {
                        try_files $uri $uri/ =404;
                }

                location ~\.php$ {
                        # Pass php requests to the php-fpm service (fastcgi)
                        include fastcgi.conf;
                        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
                }

        }
}
```

#### ❉ 동시접근 커넥션 가능 수
 - worker_processes * worker_connections


### pid 교체
```
user www-data;

pid /var/run/new_nginx.pid;

worker_processes auto;

events {
        worker_connections 1024;
}


http {
        include mime.types;

        server {
                listen 80;
                server_name 13.125.215.175;

                # connect file system to uri from static requests
                root /sites/demo;

                index index.php index.html;

                location / {
                        try_files $uri $uri/ =404;
                }

                location ~\.php$ {
                        # Pass php requests to the php-fpm service (fastcgi)
                        include fastcgi.conf;
                        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
                }

        }
}
```

### Buffer and Timeout
 - sample configuration
 - buffer directives
    + 100: 100 bytes
    + 10k: 10k bytes
    + 10m: 10m bytes
 - timeout directives
    + 30: millisecondes
    + 30s: 30 seconds
    + 30m: 30 minutes
    + 30h: 30 hours
    + 30d: 30 days
```
user www-data;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  # Buffer size for POST submissions
  client_body_buffer_size 10K; # POST 데이터 처리 버퍼 사이즈
  client_max_body_size 8m; # 8메가를 넘는 POST request를 받지 마라, 413에러 반환

  # Buffer size for Headers
  client_header_buffer_size 1k; # 1k를 넘는 http header requeest를 받지 마라

  # Max time to receive client headers/body
  client_body_timeout 12; # 클라이언트 바디 타임아웃
  client_header_timeout 12; # 클라이언트 헤더 타임아웃

  # Max time to keep a connection open for
  keepalive_timeout 15;

  # Max time for the client accept/receive a response
  send_timeout 10; # client가 응답을 받지 않을 때 타임아웃

  # Skip buffering for static files
  sendfile on; # image css 등의 static file을 읽을 때에는 buffer를 거치지 말고 다이렉트로 반환하라. static 설정으로 중요

  # Optimise sendfile packets
  tcp_nopush on; # 클라이언트에 데이타 보낼때 데이터 패킷의 사이즈를 조절하라. static 설정으로 중요

  server {

    listen 80;
    server_name 167.99.93.26;

    root /sites/demo;

    index index.php index.html;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php7.1-fpm.sock;
    }

  }
}
```

### Adding Dynamic Modules
 - 현재 설치된 nginx의 버전 및 상세 설정은 아래의 커맨드로 확인 가능하다
 - 이미지 필터 설치
    + https://nginx.org/en/docs/http/ngx_http_image_filter_module.html
    + 이미지 변환해서 
```
# nginx -V
```
 - nginx가 설치된 폴더로 간다
```
# cd /home/ubuntu/nginx-1.23.1
```
 - configure help를 통하여 현재 사용가능한 모듈을 확인한다
```
# ./configure --help
```
 - 몇몇 모듈에 dynamic이라고 붙은것이 확인 가능하다
```
--with-http_perl_module            enable ngx_http_perl_module
  --with-http_perl_module=dynamic    enable dynamic ngx_http_perl_module
  --with-perl_modules_path=PATH      set Perl modules path
  --with-perl=PATH                   set perl binary pathname

  --http-log-path=PATH               set http access log pathname
  --http-client-body-temp-path=PATH  set path to store
                                     http client request body temporary files
  --http-proxy-temp-path=PATH        set path to store
                                     http proxy temporary files
  --http-fastcgi-temp-path=PATH      set path to store
                                     http fastcgi temporary files
  --http-uwsgi-temp-path=PATH        set path to store
                                     http uwsgi temporary files
  --http-scgi-temp-path=PATH         set path to store
```
 - dynamic module만 필터링 하기
```
# ./configure --help | grep dynamic
```
 - 그런뒤 ./configure 설정에서 기존에꺼 복사하고 추가할 설정을 붙여 넣는다
```
# ./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module --with-http_image_filter_module=dynamic
```
 - 에러 남: the HTTP image filter module requires the GD library.
    + 리눅스 GD 모듈 인스톨
```
# apt-get install libgd-dev
```
    + 그 뒤 위의 ./configure 명령어를 다시 실행하면 된다
 - 컴파일
```
# make
```
 - 설치
```
# make install
```
 - 정상설치 확인
```
# nginx -V
```
 - 시스템 재실행
```
# systemctl reload nginx
```
 - 스테이터스 확인
```
# systemctl status nginx
```
 - 모듈 사용위한 conf파일 설정
    + dynamic 모듈이기 때문에 nginx.conf설정만으로 안된다. 오류남
```
  server {

    ...

    location = /thumb.png {
            image_filter rotate 180;
    }
  }

```
 - 모듈 확인
```
# ll /etc/nginx/modules
```
 - 모듀을 main context에 추가
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

load_module modules/ngx_http_image_filter_module.so;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  # Buffer size for POST submissions
  client_body_buffer_size 10K;
  client_max_body_size 8m;

  # Buffer size for Headers
  client_header_buffer_size 1k;

  # Max time to receive client headers/body
  client_body_timeout 12;
  client_header_timeout 12;

  # Max time to keep a connection open for
  keepalive_timeout 15;

  # Max time for the client accept/receive a response
  send_timeout 10;

  # Skip buffering for static files
  sendfile on;

  # Optimise sendfile packets
  tcp_nopush on;

  server {

    listen 80;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.php index.html;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    location = /thumb.png {
            image_filter rotate 180;
    }
  }
}
```
 - 재시작
```
# nginx -t
# systemctl reload nginx
```
 - 브라우저 가서 이미지 로드하면 거꾸로 보인다
    + http://ec2-13-125-215-175.ap-northeast-2.compute.amazonaws.com/thumb.png


## Performance

### Headers & Expires
 - 이미지등은 잘 바뀌지 않기 때문에 캐싱 기간을 반복하여 해당 기간 내에 반복되는 요청을 브라우저 캐싱으로 대체할 수 있다
 - nginx.conf에서 바꿈
 - 임의의 헤더 추가하기
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server {

    listen 80;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.php index.html;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    location = /thumb.png {
            add_header my_header "Hello World!"; # 헤더추가
    }
  }
}
```
 - 시스테 재시작
```
# systemctl reload nginx
```
 - curl로 헤더만 받아본다
```
# curl -I http://13.125.215.175/thumb.png
```
 - 결과값
```
HTTP/1.1 200 OK
Server: nginx/1.23.1
Date: Sun, 23 Oct 2022 04:34:22 GMT
Content-Type: image/png
Content-Length: 12627
Last-Modified: Sat, 15 Oct 2022 00:20:13 GMT
Connection: keep-alive
ETag: "6349fcbd-3153"
my_header: Hello World! # 추가된 헤더
Accept-Ranges: bytes
````
 - 캐시 컨트롤 헤더 추가하기
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server {

    listen 80;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.php index.html;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    location = /thumb.png {
            add_header Cache-Control public; # declare that this request will use cache control
            add_header Pragma public; # older version of cache control header
            add_header Vary Accept-Encoding; # except encoding response can very based on value of request header
            expires 1M; # 1 month

    }
  }
}
```
 - 재시작
```
# systemctl reload nginx
```
 - 호출
```
# curl -I http://13.125.215.175/thumb.png
```
 - 결과
```
HTTP/1.1 200 OK
Server: nginx/1.23.1
Date: Sun, 23 Oct 2022 04:41:26 GMT
Content-Type: image/png
Content-Length: 12627
Last-Modified: Sat, 15 Oct 2022 00:20:13 GMT
Connection: keep-alive
ETag: "6349fcbd-3153"
Expires: Tue, 22 Nov 2022 04:41:26 GMT # 만료기간 한달
Cache-Control: max-age=2592000 # 캐시컨트롤헤더 seconds of 1 month
Cache-Control: public # 캐시컨트롤헤더
Pragma: public # 옛날 캐시컨트롤 헤더
Vary: Accept-Encoding
Accept-Ranges: bytes
```

 - 주소 regex로 변경
    + css, js, jpg, png는 캐싱
    + 로그도 찍지 않는다
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server {

    listen 80;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.php index.html;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    location ~* \.(css|js|jpg|png)$ {
            access_log off;
            add_header Cache-Control public; # declare that this request will use cache control
            add_header Pragma public; # older version of cache control header
            add_header Vary Accept-Encoding; # except encoding response can very based on value of request header
            expires 1M; # 1 month

    }
  }
}

```
 - 재시작
```
# systemctl reload nginx
```
 - 호출
```
# curl -I http://13.125.215.175/style.css
```
 - 결과
```
HTTP/1.1 200 OK
Server: nginx/1.23.1
Date: Sun, 23 Oct 2022 05:01:31 GMT
Content-Type: text/css
Content-Length: 980
Last-Modified: Sat, 15 Oct 2022 00:20:13 GMT
Connection: keep-alive
ETag: "6349fcbd-3d4"
Expires: Tue, 22 Nov 2022 05:01:31 GMT
Cache-Control: max-age=2592000
Cache-Control: public
Pragma: public
Vary: Accept-Encoding
Accept-Ranges: bytes
```

### gzip
 - compress static resources
 - css같은 것을 압축해서 보낸다. 브라우저가 압축 풀어서 사용
 - 압축 레벨은 0~9까지 있다
 - 하지만 4이후로는 크게 용량이 줄어들지 않으므로, 보통 3~4로 설정한다
 - 설정
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  gzip on; # 압축 켜기
  gzip_comp_level 3; # 압축레벨 3

  gzip_types text/css; # css 압축
  gzip_types text/javascript; # javascirpt 

  server {

    listen 80;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.php index.html;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    location ~* \.(css|js|jpg|png)$ {
            access_log off;
            add_header Cache-Control public; # declare that this request will use cache control
            add_header Pragma public; # older version of cache control header
            add_header Vary Accept-Encoding; # except encoding response can very based on value of request header
            expires 1M; # 1 month

    }
  }
}

```

 - 시스템 재시작
```
# systemctl reload nginx
```
 - 헤더에 인코딩 받는다고 선언하고 헤더만 받기
```
# curl -I -H "Accept-Encoding: gzip, deflate" http://11.1.1.1/style.css
```
 - 내용 확인
```
HTTP/1.1 200 OK
Server: nginx/1.23.1
Date: Tue, 25 Oct 2022 14:29:47 GMT
Content-Type: text/css
Last-Modified: Sat, 15 Oct 2022 00:20:13 GMT
Connection: keep-alive
ETag: W/"6349fcbd-3d4"
Expires: Thu, 24 Nov 2022 14:29:47 GMT
Cache-Control: max-age=2592000
Cache-Control: public
Pragma: public
Vary: Accept-Encoding
Content-Encoding: gzip # 압축 허용
```
 - 헤더 + 바디 받기
```
# curl -H "Accept-Encoding: gzip, deflate" http://11.1.1.1/style.css
```
 - 바디 받은 내용
```
Warning: Binary output can mess up your terminal. Use "--output -" to tell 
Warning: curl to output it to your terminal anyway, or consider "--output 
Warning: <FILE>" to save to a file.
```
 - 용량 비교( 980 => 487)
```
root@ip-172-31-7-40:/home/ubuntu# curl http://13.125.215.175/style.css > style.css
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   980  100   980    0     0    97k      0 --:--:-- --:--:-- --:--:--  106k
root@ip-172-31-7-40:/home/ubuntu# curl -H "Accept-Encoding: gzip, deflate" http://13.125.215.175/style.css > style.min.css
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   487    0   487    0     0  47820      0 --:--:-- --:--:-- --:--:-- 48700
```

### Fast CGI Cache
 - micro cache:는 php통해서 db갔다온 결과를 캐싱해놓는것
 - 결과를 static 데이터 반환하듯이 캐시된 데이터를 반환
 - 예제에서는 fastcgi를 통해서 micro cache 구현
 - fastcgi_cache_path
    + levels: 하위 경로의 트리구조 수 
 - fastcgi_cache_key
    + 아래의 구조에 대하여 캐싱함
#### ※ 대부분의 주소 구조, cache entiry

$scheme		$request_method	$host		$request_uri
https://	GET		domain.com	/blog/article

 - 설정
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;


  # Configure micro cache (fastcgi), 캐싱 설정
  fastcgi_cache_path /tmp/nginx_cache levels=1:2 keys_zone=ZONE_1:100m inactive=60m;
  fastcgi_cache_key "$scheme$request_method$host$request_uri";


  server {

    listen 80;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.php index.html;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;

      # Enable cache, 캐싱 허용
      fastcgi_cache ZONE_1;
      fastcgi_cache_valid 200 60m;
      fastcgi_cache_valid 404 10m;
    }

  }
}
```
 - 캐싱성능 테스트
    + apache bench로 테스트 가능
    + 설치 ubuntu
```
# apt-get install apache2-utils
```
    + 설치 centos
```
# yum install httpd-tools
```
 - 벤치툴 동작하는지 확인
```
# ab
```
 - 접속 체크
```
curl http://13.125.215.175
```
 - 100개의 접속을 10개씩 동시에 진행
```
# ab -n 100 -c 10 http://13.125.215.175/
```
 - 결과
```
This is ApacheBench, Version 2.3 <$Revision: 1879490 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 13.125.215.175 (be patient).....done


Server Software:        nginx/1.23.1
Server Hostname:        13.125.215.175
Server Port:            80

Document Path:          /
Document Length:        36 bytes

Concurrency Level:      10
Time taken for tests:   0.078 seconds
Complete requests:      100
Failed requests:        0
Total transferred:      17300 bytes
HTML transferred:       3600 bytes
Requests per second:    1274.26 [#/sec] (mean) # 초당 몇번의 request를 했는지
Time per request:       7.848 [ms] (mean) # 하나의 request가 응답 받는데 얼마나 거렸는지
Time per request:       0.785 [ms] (mean, across all concurrent requests)
Transfer rate:          215.28 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    3   2.9      1       8
Processing:     1    4   3.6      2      16
Waiting:        0    4   3.5      2      16
Total:          1    8   5.2      8      22

Percentage of the requests served within a certain time (ms)
  50%      8
  66%      8
  75%      9
  80%     14
  90%     15
  95%     16
  98%     22
  99%     22
 100%     22 (longest request)


```
 - 인덱스 파일 변경. 호출될 때마다 1초
```
<?php sleep(1); ?>
<h1>Date: <?php echo date("l jS F"); ?></h1>
```
 - 다시 다중 호출
```
# ab -n 100 -c 10 http://13.125.215.175/
```
 - upstream_cache_status
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;


  # Configure micro cache (fastcgi)
  fastcgi_cache_path /tmp/nginx_cache levels=1:2 keys_zone=ZONE_1:100m inactive=60m;
  fastcgi_cache_key "$scheme$request_method$host$request_uri";
  add_header X-Cache $upstream_cache_status; # 헤더에 추가

  server {

    listen 80;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.php index.html;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;

      # Enable cache
      fastcgi_cache ZONE_1;
      fastcgi_cache_valid 200 60m;
      fastcgi_cache_valid 404 10m;
    }

  }
}

```

 - curl로 페이지 호출
```
root@ip-172-31-7-40:/home/ubuntu# curl -I  http://13.125.215.175/
HTTP/1.1 200 OK
Server: nginx/1.23.1
Date: Tue, 25 Oct 2022 15:05:51 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Cache: HIT # HIT이면 캐시된 결과
```
 - 캐시된 결과 아니면 헤더 바뀜
```
root@ip-172-31-7-40:/home/ubuntu# curl -I  http://13.125.215.175/index.php
HTTP/1.1 200 OK
Server: nginx/1.23.1
Date: Tue, 25 Oct 2022 15:07:04 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Cache: MISS # 캐시된 결과가 아님
```

#### Cache exception
 - 캐싱예외 등록
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;


  # Configure micro cache (fastcgi)
  fastcgi_cache_path /tmp/nginx_cache levels=1:2 keys_zone=ZONE_1:100m inactive=60m;
  fastcgi_cache_key "$scheme$request_method$host$request_uri";
  add_header X-Cache $upstream_cache_status;


  server {

    listen 80;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.php index.html;

    # Cache by defualt
    set $no_cache 0;

    # check for cache bypass, 파라미터에 skipcache가 1이면 캐싱 안함
    if ($arg_skipcache = 1) { 
            set $no_cache 1;
    }

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;

      # Enable cache
      fastcgi_cache ZONE_1;
      fastcgi_cache_valid 200 60m;
      fastcgi_cache_valid 404 10m;
      fastcgi_cache_bypass $no_cache; # 캐싱안함
      fastcgi_no_cache $no_cache; # 캐싱안함
    }

  }
}
```
 - 테스트
```
root@ip-172-31-7-40:/home/ubuntu# curl -I  http://13.125.215.175/?skipcache=1
HTTP/1.1 200 OK
Server: nginx/1.23.1
Date: Tue, 25 Oct 2022 15:13:00 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Cache: BYPASS
```
### HTTP2
 - binary protocal
 - comporessed headers
 - persistent connections
 - multiplex streaming
 - server push

#### HTTP2 설치
 - 현재 설치된 설정 확인
```
# nginx -V
```
 - http2 버전 모듈 확인
```
# ./configure --help | grep http_v2
```
 - 설정 변경
```
# ./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module --with-http_image_filter_module=dynamic --with-http_v2_module
```
 - 컴파일
```
# make 
```
 - 인스톨
```
# make install
```
 - 재시작
```
# systemctl restart nginx
```
 - 확인
```
# systemctl status nginx
```
#### 테스팅을 위하여 self ssl을 설치
 - 폴더 생성
```
# cd
# mkdir /etc/nginx/ssl
```
  - 테스트용 키 생성
```
# openssl req -x509 -days 100 -nodes -newkey rsa:2048 -keyout /etc/nginx/ssl/self.key -out /etc/nginx/ssl/self.crt
```
 - 추가정보 입력
```
Country Name (2 letter code) [AU]:KR
State or Province Name (full name) [Some-State]:BUSAN
Locality Name (eg, city) []:BUSAN
Organization Name (eg, company) [Internet Widgits Pty Ltd]:neighborpil.com
Organizational Unit Name (eg, section) []:dev
Common Name (e.g. server FQDN or YOUR name) []:John Doe
Email Address []:neighbor@gmail.com
```
 - 하고나면 self.crt(인증서), self.key(private key)가 생긴다
 - nginx ssl 설정을 한다
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server {

    listen 443 ssl; # listen port 설정
    server_name 13.125.215.175;

    root /sites/demo;

    index index.php index.html;

    ssl_certificate /etc/nginx/ssl/self.crt; # 인증서 
    ssl_certificate_key /etc/nginx/ssl/self.key; # 키

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

  }
}
```
 - 시스템 재시작
```
# systemctl reload nginx
```
 - 이상태로는 http1으로 동작한다
 - http2 옵션 넣기
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server {

    listen 443 ssl http2; # http2 옵션 넣기
    server_name 13.125.215.175;

    root /sites/demo;

    index index.php index.html;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

  }
}
```
 - 페이지를 다시 호출해보면 프로토콜이 http2로 바뀐다(지원하는 브라우저에서만)

### Server Push
 - 리눅스에서 테스트 하려면 라이브러리 설치 필요
```
# apt-get install nghttp2-client
```
 - 테스팅
    + n : 테스팅임. 데이터를 저장하지 않음
    + y : self-sertificate ssl를 무시함
    + s : print the response statistics
    + a : also requests style,image and etc 
```
# nghttp -nysa https://ec2-13-125-215-175.ap-northeast-2.compute.amazonaws.com/index.html
***** Statistics *****

Request timing:
  responseEnd: the  time  when  last  byte of  response  was  received
               relative to connectEnd
 requestStart: the time  just before  first byte  of request  was sent
               relative  to connectEnd.   If  '*' is  shown, this  was
               pushed by server.
      process: responseEnd - requestStart
         code: HTTP status code
         size: number  of  bytes  received as  response  body  without
               inflation.
          URI: request URI

see http://www.w3.org/TR/resource-timing/#processing-model

sorted by 'complete'

id  responseEnd requestStart  process code size request path
 13     +4.57ms        +54us   4.51ms  200   4K /index.html
 15     +6.92ms      +6.09ms    829us  200  980 /style.css
 17     +9.09ms      +6.09ms   3.00ms  200  12K /thumb.png
```
 - html요청이 왔을때 style.css, thumb.png를 푸시하도록 수정
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server {

    listen 443 ssl http2;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.php index.html;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    location = /index.html { # html 요청이 오면
            http2_push /style.css; # css 푸쉬
            http2_push /thumb.png; # png 푸쉬
    }

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

  }
}
```

## SSL
 - 설정 초기화
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server {

    listen 443 ssl http2;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.html;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

  }
}
```
 - 80포트 443으로 리다이렉트 하기
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server {
          listen 80;
          server_name 13.125.215.175;
          return 301 https://$host$request_uri;
  }

  server {

    listen 443 ssl http2;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.html;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

  }
}
```
 - 확인
```
# curl -Ik http://ec2-13-125-215-175.ap-northeast-2.compute.amazonaws.com/index.html
HTTP/1.1 301 Moved Permanently # 
Server: nginx/1.23.1
Date: Wed, 26 Oct 2022 04:31:07 GMT
Content-Type: text/html
Content-Length: 169
Connection: keep-alive
Location: https://ec2-13-125-215-175.ap-northeast-2.compute.amazonaws.com/index.html
```
 - cipher 설정
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server {
          listen 80;
          server_name 13.125.215.175;
          return 301 https://$host$request_uri;
  }

  server {

    listen 443 ssl http2;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.html;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    # Disable SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # ssl을 오래되었다. 그래서 tls로 바꿔줌

    # Optimize cipher suits
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5; # 서버 설정당시의 최신버전으로 맞춰주면 된다

    # Enalbe DH Params (Diffie Helmond Key Exchange)
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;



    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

  }
}

```
 - dhparam.pem 키 만들기
    + openssl로 cert를 만들때 만들었던 암호화(rsa 2048)을 그대로 사용해야 한다
```
# openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```
 - 시스템 재시작
```
# systemctl reload nginx
```
 - 테스트
```
# curl -Ik http://ec2-13-125-215-175.ap-northeast-2.compute.amazonaws.com
```
 - 추가 설정
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server {
          listen 80;
          server_name 13.125.215.175;
          return 301 https://$host$request_uri;
  }

  server {

    listen 443 ssl http2;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.html;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    # Disable SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    # Optimize cipher suits
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

    # Enalbe DH Params (Diffie Helmond Key Exchange)
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;

    # Enable HSTS
    add_header Strict-Transport-Security "max-age=31536000" always; # https 접속 브라우저 강제

    # SSL session
    ssl_session_cache shared:SSL:40m; # ssl 캐싱
    ssl_session_timeout 4h; # 타임아웃
    ssl_session_tickets on; # 서버발행 티켓이라 신뢰 가능


    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

  }
}

```
 - 접속 테스트
```
# curl -Ik https://ec2-13-125-215-175.ap-northeast-2.compute.amazonaws.com

HTTP/2 200 
server: nginx/1.23.1
date: Thu, 27 Oct 2022 00:15:46 GMT
content-type: text/html
content-length: 4490
last-modified: Sat, 15 Oct 2022 00:20:13 GMT
etag: "6349fcbd-118a"
strict-transport-security: max-age=31536000
accept-ranges: bytes
```
### Rate Limiting
 - ddos 공격등 접속 방어
 - 테스팅 위해 siege라는 새로운 툴 설치 필요
```
# apt-get install siege
```
 - brute force attack
    + -v: verbose
    + -r: repeat
    + -c: concurrent
 - 10번 공격
```
# siege -v -r 2 -c 5 https://ec2-13-125-215-175.ap-northeast-2.compute.amazonaws.com/thumb.png

{	"transactions":			          10,
	"availability":			      100.00,
	"elapsed_time":			        0.05,
	"data_transferred":		        0.12,
	"response_time":		        0.02,
	"transaction_rate":		      200.00,
	"throughput":			        2.41,
	"concurrency":			        4.80,
	"successful_transactions":	          10,
	"failed_transactions":		           0,
	"longest_transaction":		        0.04,
	"shortest_transaction":		        0.01
}

```
 - 설정: 접속제한
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  # Define limit zone
  # limit_req_zone $server_name;
  # limit_req_zone $binary_remote_addr;
  limit_req_zone $request_uri zone=MYZONE:10m rate=60r/m; # 분당 60 request로 제한한다. 즉 초당 한번만 요청가능 나머지는 리젝트(214) 이렇게 하면 접속 spike를 막아준다

  server {
          listen 80;
          server_name 13.125.215.175;
          return 301 https://$host$request_uri;
  }

  server {

    listen 443 ssl http2;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.html;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    # Disable SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    # Optimize cipher suits
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

    # Enalbe DH Params (Diffie Helmond Key Exchange)
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;

    # Enable HSTS
    add_header Strict-Transport-Security "max-age=31536000" always;

    # SSL session
    ssl_session_cache shared:SSL:40m;
    ssl_session_timeout 4h;
    ssl_session_tickets on;


    location / {
        limit_req zone=MYZONE; # 존 
        try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

  }
}

```
 - 테스트
```
root@ip-172-31-7-40:/home/ubuntu# siege -v -r 2 -c 5 https://ec2-13-125-215-175.ap-northeast-2.compute.amazonaws.com/thumb.png

{	"transactions":			           1, # 한번만 
	"availability":			       10.00,
	"elapsed_time":			        0.05,
	"data_transferred":		        0.01,
	"response_time":		        0.23,
	"transaction_rate":		       20.00,
	"throughput":			        0.27,
	"concurrency":			        4.60,
	"successful_transactions":	           1,
	"failed_transactions":		           9,
	"longest_transaction":		        0.03,
	"shortest_transaction":		        0.02
}


```

 - burst 추가: burst가 없으면 1회 요청 이외에 즉시 에러를 띄우지만, burst를 넣어주면 1초 리밋마다 리턴
    + burst에 숫자가 있는 이유는 1+5회 요청까지는 1초 리밋이 지날때마다 리터하지만, 1+5회를 넘어가는 요청에 대해서는 에러를 띄운다
    + burst뒤에 nodelay를 넣어주면 1+5회까지 빠르게 
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  # Define limit zone
  # limit_req_zone $server_name;
  # limit_req_zone $binary_remote_addr;
  limit_req_zone $request_uri zone=MYZONE:10m rate=60r/m;
  # limit_req_zone $request_uri zone=MYZONE:10m rate=60r/m burst=5; # 여기 적어도 된다

  server {
          listen 80;
          server_name 13.125.215.175;
          return 301 https://$host$request_uri;
  }

  server {

    listen 443 ssl http2;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.html;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    # Disable SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    # Optimize cipher suits
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

    # Enalbe DH Params (Diffie Helmond Key Exchange)
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;

    # Enable HSTS
    add_header Strict-Transport-Security "max-age=31536000" always;

    # SSL session
    ssl_session_cache shared:SSL:40m;
    ssl_session_timeout 4h;
    ssl_session_tickets on;


    location / {
        limit_req zone=MYZONE burst=5 nodelay; # 1r/s + 5회만큼 허용
        try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

  }
}

```
 - 재시작 후 테스트
```
# siege -v -r 1 -c 6 https://ec2-13-125-215-175.ap-northeast-2.compute.amazonaws.com/thumb.png

{	"transactions":			           6,
	"availability":			      100.00,
	"elapsed_time":			        0.04,
	"data_transferred":		        0.07,
	"response_time":		        0.02,
	"transaction_rate":		      150.00,
	"throughput":			        1.81,
	"concurrency":			        3.50,
	"successful_transactions":	           6,
	"failed_transactions":		           0,
	"longest_transaction":		        0.04,
	"shortest_transaction":		        0.01
}
ubuntu# siege -v -r 1 -c 6 https://ec2-13-125-215-175.ap-northeast-2.compute.amazonaws.com/thumb.png

{	"transactions":			           2,
	"availability":			       33.33,
	"elapsed_time":			        0.03,
	"data_transferred":		        0.02,
	"response_time":		        0.07,
	"transaction_rate":		       66.67,
	"throughput":			        0.83,
	"concurrency":			        4.67,
	"successful_transactions":	           2,
	"failed_transactions":		           4,
	"longest_transaction":		        0.03,
	"shortest_transaction":		        0.01
}

```

### Basic Auth
 - ./htpassword파일 만들어야 함
 - 라이브러리 설치 필요
    + ubuntu
```
# apt-get install apache2-utils
```
    + centos
```
# yum install httpd-tools
```
 - 파일 생성
```
# htpasswd -c /etc/nginx/.htpasswd user
New password: 
Re-type new password: 
Adding password for user user1
```
 - 파일 확인
```
# cat /etc/nginx/.htpasswd 
user1:$apr1$B9I61G3o$U8USA2sM5YGhPRiNubQlm/
```
 - 환경설정
```
location / {
        auth_basic "Secure Area"; # basic auth 설정
        auth_basic_user_file /etc/nginx/.htpasswd; # 아디/패스워드 있는 곳
        limit_req zone=MYZONE burst=5 nodelay;
        try_files $uri $uri/ =404;
    }

```
 - 시스템 재시작
```
# systemctl reload nginx
```
 - 웹페이지 접속하면 아디/비번 뜸
![image](https://user-images.githubusercontent.com/22423285/198419504-b0482b18-256d-45d2-b253-2034e9337976.png)


### Hardening Nginx
 - remove few setting to focus on
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server {
          listen 80;
          server_name 13.125.215.175;
          return 301 https://$host$request_uri;
  }

  server {

    listen 443 ssl http2;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.html;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    # Disable SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    # Optimize cipher suits
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

    # Enalbe DH Params (Diffie Helmond Key Exchange)
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;

    # Enable HSTS
    add_header Strict-Transport-Security "max-age=31536000" always;

    # SSL session
    ssl_session_cache shared:SSL:40m;
    ssl_session_timeout 4h;
    ssl_session_tickets on;


    location / {
        limit_req zone=MYZONE burst=5 nodelay;
        try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

  }
}
```
 - 취약점 보강위해 업데이트 한다
```
# apt-get update
# apt-get upgrade
```
 - nginx 버전업 할것 있으면 버전업
    + http request 헤더에 보면 nginx 버전이 나온다.
    + 취약점이 있는 버전이면 이걸로 공격한다
```
# nginx -v # 버전확인
```
#### http request header에서 nginx 버전 지우기
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server_tokens off; # 이 부분을 추가해주면 된다

  ...
}
```
 - 시스템 재시작
```
# systemctl reload nginx
```
 - curl로 사라졌는지 확인
```
# curl -Ik https://13.125.215.175
HTTP/2 200 
server: nginx # 여기에 버전이 사라지고 nginx만 나와야 한다
date: Fri, 28 Oct 2022 03:57:44 GMT
content-type: text/html
content-length: 4490
last-modified: Sat, 15 Oct 2022 00:20:13 GMT
etag: "6349fcbd-118a"
strict-transport-security: max-age=31536000
accept-ranges: bytes
```

#### X frame 옵션 적용
 - 악의적인 사이트에서 내 사이트를 iframe으로 감싸서 진짜 사이트인척 할 수 있다.
 - 따라서 브라우저에게 iframe을 사용하지 말라고 헤더에 적어주는 옵션 추가가 필요하다
 - 같은 origin의 사이트만 가능하도록 허용("SAMEORIGIN")
```
user www-data;

pid /var/run/nginx.pid;

worker_processes auto;

events {
  worker_connections 1024;
}

http {

  include mime.types;

  server_tokens off;

  server {
          listen 80;
          server_name 13.125.215.175;
          return 301 https://$host$request_uri;
  }

  server {

    listen 443 ssl http2;
    server_name 13.125.215.175;

    root /sites/demo;

    index index.html;

    add_header X-Frame-Options "SAMEORIGIN"; # 이 부분에서 적용된다
    add_header X-XSS-Protection "1; mode=block"; # cross-site scripting 이 발견되면 로딩 금지하라

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    # Disable SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

  ...
}
```
 - 안쓰는 모듈은 취약점이 있기 때문에 삭제
    + ./configure --help옵션을 보면 --without으로 시작하는 것들이 있다.
    + 이것들은 기본모듈로 깔리는 애들인데 안쓰면 옵션 주면된다
```
# ./configure --help | grep without
```
 - autoindex 모듈 삭제
```
# ./configure --without-http_autoindex_module --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module --with-http_image_filter_module=dynamic --with-http_v2_module
```
 - 빌드 및 설치, 실행
```
# make
# make install
# systemctl start nginx
```
