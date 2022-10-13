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

#### 만약 인터넷에서 nginx source를 받아서 압축 풀어서 빌드해서     사용하려고 할때에 C 컴파일러가 없다면 인스톨 필요
 - ubuntu
```
# apt-get install build-essential
```
 - centos
```
# yum groupinstall "Development Tools"
```
 - 컴파일방법
```
# ./configure
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
 - nginx 설정하기
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
