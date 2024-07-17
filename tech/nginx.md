# Nginx (개요와 설정파일, 그리고 로드밸런싱)

### Contents
#### 1. 개요
#### 2. vs Apache?
#### 3. 설정 파일 (nginx.conf)
#### 4. Nginx를 활용한 로드 밸런싱 예시 (Reverse Proxy for Load Balancing)

## 개요
- Nginx는 Apache와 같은 웹서버이며, 클라이언트와 통신하여 **정적 파일**을 제공하는 소프트웨어이다. (동적 파일은 WAS)
- 정적 파일을 제공하는 경량화된 웹서버로서도 기능을 하지만, 요청을 받아 뒷단의 WAS(들)로 전달하는 **리버스 프록시 서버 (Reverse Proxy Server)**, 혹은 **로드 벨런서 (Load Balancer)** 로도 동작을 한다.
    - 프록시 서버라는 점에서 **캐싱** 및 **암호화 (e.g. certbot을 활용한 https)** 도 가능하다!

## vs Apache?
- 위에서 이야기를 했듯이, Nginx는 '경량화'된 웹서버이며, 요청을 처리하는 방법에서 차이점이 존재한다.
    - **Apache**
        - Apache는 요청마다 새로운 스레드를 만들어 요청을 처리한다. \
        - 각 요청에 대한 병렬 처리가 가능하고 빠른 처리가 가능하지만, 요청이 많아질수록 많은 자원을 소모한다는 단점이 있다.
    - **Nginx**
        - Nginx는 **Event-Driven** 방식으로 요청들을 처리한다. (NodeJS가 요청을 처리하는 방법과 비슷하다.)
        - 요청의 수와 관계없이 하나 혹은 고정된 개수의 프로세스가 형성이 되고, 각 프로세스는 비동기적 (asynchronous) 논블로킹 (Non-blocking)방식으로 여러개의 요청을 처리한다. 
            - 다음과 같은 프로세스가 형성된다.
                - **Master Process**: 설정 파일을 읽고 (nginx.conf) 유효성을 검사
                - **Worker Process**: 요청을 처리 (개수는 설정파일에서 정의가 됨.)
        - 이와 같은 이유로 적은 자원으로 요청들을 효율적으로 처리할 수 있지만, CPU-Intensive하거나 복잡한 로직 처리가 필요한 요청들이 많이 온다면 응답에 대한 latency가 길어진다는 단점이 존재한다. (비동기적 논블로킹 방식에서 존재하는 길어지는 이벤트 큐 문제.) 

---

## 설정 파일 (nginx.conf)
- 기본 위치: `/etc/nginx/nginx.conf`

### 1. Core
```nginx
user root 
# nginx process가 어느 사용자(그룹)로서 동작을 할 것인지. (user root === nginx는 모든 권한을 갖게 된다.)

worker_processes auto
# Worker Process의 수. (worker_processes auto === 자동으로 CPU 코어 수로 설정)

error_log /var/log/nginx/error.log
# nginx 에러로그 저장하는 파일명

pid /var/run/nginx.pid
# nginx process id 저장하는 경로
```

### 2. Event Block
```nginx
events {
    worker_connections 1024;
    # 하나의 worker process가 동시 처리 가능한 connection 수. (Client Connection + Proxy Target Connection)
}
```

### 3. HTTP Block
```nginx
http {
    include /etc/nginx/mime.types;
    # nginx의 설정 include

    default_type application/octet-stream;
    # nginx의 기본 content type


    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    # 작성되는 log format (https://nginx.org/en/docs/varindex.html 공식 문서 참고)
    
    access_log /var/log/nginx/access.log main;
    # log에 접근하기 위한 경로

    sendfile on;
    # sendfile syscall 사용 여부 (sendFile 시스템 콜을 사용 시, 파일 전송의 속도가 빨라지고 메모리 사용이 감소함)

    keepalive_timeout 65;
    # Client 접속 유지 시간

    # ... Server Block

    include /etc/nginx/conf.d/*.conf;
    # nginx의 모든 설정 include 

    tcp_nopush off
    # TCP 연결이 되어있는 클라이언트에게 데이터를 즉시 보낼 지 (on - 즉각적 전송, 보안 이슈보다 성능적 이점이 중요할 시), 버퍼를 두고 보낼지 (off) 설정

}
```

### 4. Server Block
- 전달 받은 요청에 대해서 어떤 서버로 전달할지 정의한다.

#### Server Block (Common)
```nginx
server { 
        listen 8080; 
        # 8080포트로 오는 요청을 주시하여 처리 (Proxy)
        
        server_name test.io www.test.io
        # 호스트 이름, 해당 호스트 이름으로 요청이 들어오는 것을 proxy.

        access_log /var/log/nginx/access.log;
        # HTTP로 접속 기록 로깅 위치

        rewrite ^/number/(\w+) /count/$1; 
        # url 변경. 정규식 사용 가능. $1은 variable.

        location {...}
        # 경로별 어떤 resource를 전달할 것인지에 대해서 설정
}
```

#### Server Block (Static Page)
```nginx
server { 
        listen 8080; 

        root /Users/.../html; 
        # 루트 페이지 파일경로 (index.html을 제외한 절대경로)

        rewrite ^/number/(\w+) /count/$1; 
        # url 변경. 정규식 사용 가능. $1은 variable.

        error_page 404 /404.html
        location = /404.html {
	        root /Users/.../html;
        }
        # 에러에 대한 지정된 정적 page. 

        location /user { # location에는 root를 제외한 다른 경로들에 대한 url을 정의하는 부분
            
            root /Users/../html/users (index.html 제외)
            try_files /$uri/index.html =404; # 파일이 없는 경우, 해당 경로에 있는 파일을 랜더링. (이 경우는, 절대경로로 입력했을 경우에 fallback설정.) 그래도 실패할 시 404.
        }


        location ~* /user/[0-9] { # dynamic route
            root /Users/../html/user
            try_files /index.html =404;
        }


        location /carbs {
            alias /Users/.../html/users; 
            # alias, 해당 api로 /users가 전달됨
        }

        location /data {
            return 307 /users # /data로 오는 요청에 대해서 /users로 redirect
        }
    }
```


#### Server Block (Reverse Proxy)
```nginx
server { 
        listen 80;
        # 해당 port로 오는 접근을 주시하여 server block에서 처리.

        location / { # 해당 경로에 오는 요청을
            proxy_pass http://127.0.0.1:3000/; # 이곳으로 전달.
        }
    }
```

#### Server Block (Upstream - Load Balancing)
```nginx
    upstream server {
        # upstream: 요청을 어떤 서버로 흘려보낼지 정의
            # 서버가 여러개 정의가 되어있을 경우 규칙에 따라 한 서버로 흘려보냄.
            # 로드 벨런서 역할을 이 곳에서 정의 

        # 로드 벨런서의 규칙 정의 (지시어)
            # 1. "아무것도 적지 않기 - 지시어 없음": 라운드 로빈 방식 (Round Robid)
            # 2. hash: 해시 방식 (Hash)
                # -  각 요청에 대해 사용자가 지정한 택스트와 nginx 변수 조합으로 해시를 계산하여 연결.
                # -  해당 해시가 포함된 요청 == 해당 서버가 성립, 세션 지속성 유지
                # -  e.g. hash $scheme$request_uri => H( (http/https) || 전체 uri )
            # 3. ip_hash: IP 해시 방식 (IP Hash)
                # - hash의 변형: 클라이언트의 IP 주소 기반 해시
            # 4. least_conn: Least Connections
                # - 연결 수 비교 후, 가장 연결이 적은 서버로 요청을 보냄
            # 5. least_time: Least Time (Ngnix Plus)
                # - 연결 수 + 과거 요청에 대한 평균 응답 시간 (가중치)

        server localhost:3000 2; # Weight 2
        server localhost:3001;
        server localhost:3002;

        # server host:port weight=n max_fails=n fail_timeout=n down backup
            # - weight: 서버의 비중. 2로 설정할 시, 일반 서버보다 2배 더 자주 선택.
            # - max_fails: n만큼의 실패가 일어날 시, server failure로 간주.
            # - fail_timeout: max_fails가 지정된 상태에서 해당 시간만큼 서버가 응답하지 않으면 server faiure로 간주
            # - down: 서버 사용 X (ip_hash 지시어 설정 시에만 가능)
            # - backup: 서버 사용 X, 모든 서버가 작동을 하지 않을 때 해당 서버가 사용.

    }
     
```

## 4. Nginx를 활용한 로드 밸런싱 예시 (Reverse Proxy for Load Balancing)
```nginx
user nginx;
worker_processes auto;
pid /var/run/nginx.pid;

error_log /var/log/nginx/error.log warn;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    keepalive_timeout 65;

    upstream server {
        # LB: Round Robin with 3 instances
        server localhost:3000;
        server localhost:3001;
        server localhost:3002;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://server/; # Upstream Name
        }
    }

    include /etc/nginx/conf.d/*.conf;
}
```

###### References
- [NGINX Load Balancing 사용 사례](https://nginxstore.com/blog/nginx/nginx-load-balancing-%EC%82%AC%EC%9A%A9-%EC%82%AC%EB%A1%80/)
- [[🚗 자동화 개발 회고] EC2, PM2, Nginx 를 이용한 Blue/Green 무중단 배포 구축기](https://velog.io/@chltjdrhd777/%EC%9E%90%EB%8F%99%ED%99%94-%EA%B0%9C%EB%B0%9C-%ED%9A%8C%EA%B3%A0-EC2-PM2-Nginx-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%AC%B4%EC%A4%91%EB%8B%A8-%EB%B0%B0%ED%8F%AC-%EA%B5%AC%EC%B6%95%EA%B8%B0)
- [Nginx Docs - HTTP Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)