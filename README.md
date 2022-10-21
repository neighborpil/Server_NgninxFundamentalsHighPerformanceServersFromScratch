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
