# Nginx

담당자: 강성민
진행 상태: 게시 완료
유형: 서버
마감일: 2024년 2월 4일
회차: 2

# Nginx

### Nginx 란,

![Untitled](image/nginx_logo.png)

[Nginx](https://www.nginx.com/)는 웹 서버 소프트웨어로, 가벼움과 높은 성능을 갖는다. 비슷한 소프트웨어로 Apache HTTP Server(이하 httpd)가 있다. httpd가 프로세스, 쓰레드 구조를 갖는 반면, Nginx는 비동기 이벤트 구조를 가진다. 이러한 구조는 서버에 많은 부하가 생길 경우 더 유리하다.

Nginx는 HTTP 프록시와 웹 서버 기능을 갖고 있으며, 아래와 같은 기능을 할 수 있다.

- SSL 지원
- 로드 밸런싱
- URL 다시쓰기
- 10,000개의 동시 접속 처리
- 맞춤 로깅
- …

### Nginx 명령어

실행중인 nginx에 signal 명령어를 사용하여 동작을 제어할 수 있다. -s 옵션이 signal을 의미한다.

예) nginx -s reload

- stop : fast shutdown. 즉시 종료.
- quit : graceful shudwon. 현재 진행 중인 요청이 모두 끝난 후에 종료.
- reload : 설정 파일만 리로딩
    - 새로운 설정을 검증한 후 이상이 없으면 새로운 프로세스를 실행시켜 기존 프로세스와 교체 작업을 한다.
- reopen : 로그 파일 다시 열기

## Nginx 구성

| 파일 또는 폴더 명 | 파일 / 폴더 구분 | 설명 |
| --- | --- | --- |
| nginx.conf | 파일 | nginx의 기본 설정 파일. http 안에 여러 server를 가질 수 있다.<br>conf.d/*.conf, sites-enabled/* 를 http 안에서 읽어들여 각 server 설정을 외부 파일로 분리하여 관리할 수 있다. |
| conf.d | 폴더 | conf 확장자의 파일들만 nginx.conf 설정에 포함된다. 설정에서 뺄 파일은 단순히 확장자를 변경하면 되므로 관리하기 편리하다. 현재 nginx에서 주로 사용하는 방법이다. |
| sites-enabled | 폴더 | sites-available의 폴더에서 사용할 설정들만 심볼릭 링크로 생성하여 사용한다. nginx_ensite, nginx_dissite 에 의해 관리되며, 데비안의 a2ensite, a2disite 명령어와 관련이 있다. httpd 에서 사용하던 방식으로, 현재 nginx에서 굳이 사용할 필요는 없다. |
| sites-available | 폴더 | server 설정 파일들을 보관한다. |

## Nginx 설정

기본적으로 /etc/nginx/nginx.conf 설정 파일을 사용한다.

최상위 지시자는 각기 다른 타입의 트래픽을 담당한다.

- events - 일반적인 연결 처리
- http - HTTP 트래픽
- mail - Mail 트래픽
- stream - TCP, UDP 트래픽

위의 최상위 지시자 하위에 server 지시자를 한개 이상 사용할 수 있다. 각 server마다 요청을 처리하는 방법을 정의한다. 특히 http 지시자의 server는 각각 도메인 또는 IP 주소로 구분되며, 한 개 이상의 location 컨텍스트를 사용하여 URI에 따라 어떻게 처리할지 설정할 수 있다.

### Server

```bash
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	# SSL configuration
	# listen 443 ssl default_server;
	# listen [::]:443 ssl default_server;
	
	root /var/www/html;

	index index.html index.htm index.nginx-debian.html;

	server_name test.com;

	location / {
		try_files $uri $uri/ =404;
	}
}
```

- listen - 해당 server가 처리할 IP 또는 포트를 지정한다. IPv4, 6 모두 사용 가능.
    - default_server - 동일한 IP, port 로 설정한 server의 요청 중 도메인이 아무것도 일치하지 않을 경우 해당 server가 요청을 처리한다.
    - ssl - HTTPS 프로토콜로 요청을 받는다. ssl_certificate, ssl_certificate_key 로 인증서와 비밀키 파일을 지정해야 한다.
- server_name - HTTP 요청의 Host 헤더, 즉 요청한 도메인을 확인한다. ‘*’의 와일드 카드, 정규 표현식을 사용할 수 있다.
    - *.example.com ⇒ www.example.com, test.example.com 은 매칭되나 example.com 은 매칭되지 않음.
    - server_name example.com *.example.com 을 사용하면 모두 매칭 가능
    - ⇒ .example.com 으로 두 개를 합쳐서 표현할 수 있다. (example.com, test.example.com 매칭)
    - 도메인에 매칭되는 server_name을 갖는 server가 많을 경우 아래의 순서로 server와 매칭된다.
        1. 정확히 일치
        2. ‘*’로 시작하며 가장 길게 일치
        3. ‘*’로 끝나며 가장 길게 일치
        4. 정규 표현식에 첫 번째로 일치 (정규표현식을 일반적으로 ‘^’를 맨 처음, ‘$’를 맨 마지막에 사용하여 어디까지가 표현식인지 나타내지만, Nginx에서는 맨 앞에 ‘~^’를 사용해야 한다.)
        - 성능 최적화 - Nginx는 위 4개 중 위에서 3번째 까지 각각의 해시 테이블을 갖고 있다. 요청의 도메인을 각 해시 테이블에서 순서대로 확인하므로 아주 빈번하게 사용되는 도메인은 정확한 도메인을 설정하여 가장 빠르게 찾도록 하는 것이 유리하다.
    - server_name 으로 빈값 “” 을 사용할 수 있다. 이 경우 Host 헤더가 없는 요청을 처리한다.
- root - 해당 시스템의 경로를 HTTP 요청의 IP 또는 도메인과 매핑해준다. 정적인 자원을 매핑할 때 사용한다.
    - server_name : example.com, root : /var/www/html 일 경우
    - http://example.com ⇒ /var/www/html
    - http://example.com/abc.html ⇒ /var/www/html/abc.html
- index - 루트 경로로 들어온 요청을 처리할 기본 페이지를 설정한다. 여러개를 나열할 경우 앞에서 부터 파일을 확인하여 존재하는 파일을 사용한다.
- location - URI 패턴을 지정하여 각 패턴에 따라 요청을 다르게 처리하도록 설정할 수 있다.
    - /, /patten.. - 해당 URI 로 시작하는 요청에 해당한다. ‘/’은 루트 경로로 모든 요청에 해당하는 패턴이다.
        - /pattern ⇒ /pattern/test 요청 처리 가능
    - = /pattern.. - 지정한 URI와 정확히 일치하는 요청을 처리한다.
    - 정규표현식 - ‘~’ 로 시작하여 요청을 처리할 정규표현식을 사용할 수 있다.
    - 만약 한 요청이 여러 location에 모두 해당한다면, 가장 긴 문자열과 일치하는 location으로 처리한다.
        - 예) location : /, /test, /test/abc, 요청 : /test/abc ⇒ 3개의 location 모두 일치하지만 /test/abc 가 가장 긴 문자열이므로 해당 location에서 처리
- rewrite - 요청 URI을 정규표현식을 이용하여 다른 문자열로 치환할 때 사용한다. 다른 문자열로 치환 후 리다이렉트하도록 할 수도 있다.
    - rewrite {regex} {replacement} [flag];
    - flag
        - last - 문자열 치환 후 다시 매칭되는 location를 탐색한다. location 내부에서 잘못 사용할 경우, 치환 후에도 같은 location에 계속 매칭되어 요청이 처리되지 않아 500에러가 발생할 수 있다.(이 땐 last 대신 break를 사용해야 한다.)
        - break - last와 달리 문자열 치환까지만 한다.
        - redirect - 302 코드와 함께 임시 리다이렉트를 한다. ‘http://’, ‘https://’ 로 치환하지 않는 경우 사용한다.
        - permanent - 301 코드와 함께 영구 리다이렉트를 한다.
    - rewrite 대신 아래 옵션을 사용하면 HSTS(HTTP Strict-Transport-Security) 설정을 하여 클라이언트 측에서 내부 리다이렉트가 되도록 설정할 수도 있다.
- proxy_pass - Reverse Proxy를 위해 사용한다. 리다이렉트시키는 rewrite나, 정적인 파일과 매핑하는 root와 달리 지정한 URL로 요청을 매핑시킨다. 특정 URL 또는 upstream을 지정하여 사용한다.
    - URL로 요청을 전달할 때 현재 요청의 Host, IP 등을 그대로 전달하려면 아래 옵션을 함께 사용해야 한다. 사용하지 않을 경우 Nginx는 기본적으로 요청의 Host를 해당 시스템의 IP로 변경한다.
    
    ```bash
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    ```
    

### Reverse Proxy

우리가 일반적으로 말하는 프록시는 Forward Proxy다. 여러 클라이언트의 요청을 프록시 서버가 받아 대신 서버로 전달한다. 프록시 서버가 클라이언트를 대신하여 서버는 클라이언트를 알 수 없게 된다.

반면 리버스 프록시는 반대로 동일한 곳으로 들어온 클라이언트 요청을 각기 다른 서버로 전달한다. 클라이언트는 요청을 처리하는 서버가 정확히 어디인지 알 수 없게 된다. 로드 밸런싱에 사용될 수 있다.

## HTTP Load Balancing

로드 밸런싱은 웹 또는 웹 어플리케이션 서버의 여러 인스턴스를 하나의 그룹으로 묶어  관리하여 자원 사용 최적화, 처리량 증가, 지연시간 감소, 장애 허용 시스템(Fault tolerance, 일부 인스턴스에 문제가 발생하더라도 정상적으로 수행)을 위해 사용한다.

http 지시자 내부에 upstream 지시자를 사용하여 그룹을 지정할 수 있다.

```bash
http {
		# 3개의 서버를 backend라는 이름의 그룹으로 묶음
    upstream backend {
        server backend1.example.com weight=5;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }
}
```

위의 그룹에 요청을 매핑하려면 proxy_pass 를 사용하여 요청을 넘겨주어야 한다.

```bash
server {
    location / {
				# 이 server로 들어온 모든 요청은 backend 그룹으로 보내진다.
        proxy_pass http://backend;
    }
}
```

### 로드 밸런싱 정책 종류

한 그룹에 대해 요청이 들어왔을 때 어떤 server가 요청을 처리할 지 정하는 정책은 여러가지가 있다.

- 라운드 로빈 - 기본적으로 이 정책이 사용됨. server의 가중치를 고려하면서 순서대로 요청을 처리.
- 최소 연결 - least_conn; 각 server의 연결 수를 확인하여 가장 적은 server로 요청을 처리한다. 가중치를 고려한다.
- IP Hash - ip_hash; IPv4의 일부 또는 IPv6 전체 주소를 해시하여 매칭되는 server로 요청을 처리한다. 한 그룹의 일부 server가 사용 불가능한 상태가 되지 않는 한, **같은 주소의 요청은 같은 server로 처리하는 것을 보장**한다(서버 세션 유지 가). 일부 server를 그룹에서 잠시 제거해야할 때 server를 제거하기보단, down 옵션을 사용할 수 있다. IP Hash 정책에서 해당 server로 가는 요청은 다음 server가 처리하게 된다.
- 일반 Hash - hash .. ; 사용자가 지정한 값을 해시하여 매칭되는 server로 요청을 처리한다. IP, 포트 또는 URI 등을 사용할 수 있다.

### 로드 밸런싱 옵션

- weight=? - 한 그룹에서 A server가 가중치 5, B server가 가중치 1이라면, 6번의 요청 중 5번은 A, 1번은 B server가 처리하게 된다.
- backup - backup으로 설정된 server는 같은 그룹의 다른 server가 모두 사용 불가능한 상태가 될 때까지 요청을 받지 않는다.
- down - 해당 server를 영구적으로 사용 불가능한 것으로 설정. 한 그룹 내에서 해당 server로 요청을 처리하지 않는다.
- max_fails=number - 설정한 횟수만큼 server가 요청 처리를 실패하면 해당 server를 실패 상태로 표시한다. 0으로 설정하면 실패를 확인하지 않는다.(기본값 1)
- fail_timeout=time - server가 실패 상태가 되었을 때 요청을 처리하지 않는 시간을 설정한다. 시간이 지나면 다시 요청을 시도한다.(기본값 10초)

### 로드 밸런싱 Health check

로드 밸런싱은 passive 헬스 체크를 지원한다. server가 요청을 처리하지 못하고 실패하면 해당 server를 실패 상태로 표시한다. 이후 일정 시간이 지나면 다시 요청 처리를 시도하여 성공하면 정상 상태로 되돌린다. max_fails, fail_timeout 옵션을 사용하여 server 별로 설정할 수 있으며, max_fails=0; 을 설정하면 헬스 체크를 하지 않는다.

### 로드 밸런싱 예

```bash
upstream backend {
		# 아무런 정책을 지정하지 않았으므로 라운드 로빈
		# least_conn;
		# ip_hash;
		# hash $request_uri consistent;
    server backend1.example.com weight=5;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```

## Nginx 설치

```bash
# 설치 가능한 패키지 리스트 최신화
sudo apt update
# 설치 가능한 nginx 패키지 확인
apt list nginx
# Listing... Done
# nginx/jammy-updates 1.18.0-6ubuntu14.4 amd64

# nginx 패키지 설치
sudo apt install nginx
# 설치 완료 후
# nginx 버전 확인
nginx -v
# nginx version: nginx/1.18.0 (Ubuntu)

# nginx 실행상태 확인 (active 상태)
systemctl status nginx
# nginx.service - A high performance web server and a reverse proxy server
# Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
# Active: active (running) since Mon 2024-01-22 22:59:35 KST; 6s ago
```

- 더 최신 버전이나, 다른 버전의 Nginx를 설치하려면 Nginx 각자 OS에 맞는 설치법 참고
    - [https://nginx.org/en/linux_packages.html](https://nginx.org/en/linux_packages.html)

# 참조

- nginx docs
    - [https://docs.nginx.com/nginx/](https://docs.nginx.com/nginx/)
    - [https://nginx.org/en/docs/?_ga=2.146464564.1495774012.1706801606-2124111931.1705929015](https://nginx.org/en/docs/?_ga=2.146464564.1495774012.1706801606-2124111931.1705929015)
