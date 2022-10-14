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
# ./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/erro.log --http-log-path=/var/log/nginx/access.log --with-pcre -pid-path=/var/run/nginx.pid --with-http_ssl_module
```
 - 빌드하기
```
# make
```
 - 설치하고 기동하기
```
# make install
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
 - vim으로 스크립트를 붙여 넣는다
```
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

 - 먼저 /lib/systemd/system/nginx.service 경로에 파일을 만든다파일
 - 먼저 /lib/systemd/system/nginx.service 경로에 파일을 만든다
 
