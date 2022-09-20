# SSL 인증서와 적용

## SSL  (Secure Sockets Layer) 과 TLS (Transport Layer Security )

- 암호화 기반 인터넷 보안 프로토콜
- SSL을 사용중인 웹사이트는 HTTPS 를 사용
- SSL과 TLS는 현재는 같은 뜻으로 의미하며 TLS1.0은 SSL3.0을 계승한다

## HTTP 와 HTTPS

<aside>
💡 HTTP 와 HTTPS
- HTTP (Hypertext Transfer Protocol)  : HyperText인 html을 전송하기 위한 통신규약을 의미한다.
- HTTPS : **SSL 프로토콜 사용하여 HTTP 요청을 암호화 하여 통신할수 있게함**

</aside>

## SSL  인증서 ?

- SSL 인증서란 클라이언트와 서버간의 통신을 **제 3자가 보증**을 해주는 **문서**이다.
- 통신 내용이 공격자에게 노출 되는 것 방지
- 클라이언트가 접속하는 서버가 신뢰 할 수 있는 서버인지 판단
- 통신 내용의 무결성 보장

<aside>
💡 SSL 인증서의 내용
- 서비스의 정보 (인증서를 발급한 CA, 서비스 도메인 등)
- 서버 측 공개키 (공개키 내용, 공개키 암호화 방법 등)

</aside>

<aside>
💡 CA(Certificate authority) 
- SSL 보장을 위해 엄격하게 공인된 기업을 의미
- CA 리스트에 포함되어 있어야 공인된 CA 가 될 수 있음

</aside>

## SSL 인증 주고받는 단계

![스크린샷 2022-04-21 오후 9.06.03.png](SSL%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%E1%84%89%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%8C%E1%85%A5%E1%86%A8%E1%84%8B%E1%85%AD%E1%86%BC%20e2f21a75d1094098abe571a42fcc7a54/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-04-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.06.03.png)

<aside>
💡 Handshake 단계를 통해 클라이언트와 서버간에  SSL인증서를 주고받게됨

</aside>

1. 클라이언트 서버 접속
- 클라이언트 측에서 생성된 랜덤 데이터
- 클라이언트 측에서 지원하는 암호화 방식들 : 클라이언트와 서버간 암호화 방식이 다를 수 있기때문에 사전에 협상을 거침
- 세션 아이디 : 이미 SSL 핸드쉐이킹을 했다면 기존의 세션을 활용하는데 이때 사용할 연결에 대한 식별자를 서버 측으로 전송

2. 서버 측에서의 응답
- 서버 측에서 생성한 랜덤 데이터 
- 서버가 선택한 클라이언트의 암호화 방식 : 클라이언트가 전달한 암호화 방식 중 서버 측에서 사용한 암호화 방식을 선택해 정보를 교환
- 인증서 

3. 클라이언트가 서버의 인증서가 CA에 의해 발급 된 것인지 확인
- 클라이언트가 내장된 CA 리스트와 서버가 보낸 인증서를 비교 (인증서가 없다면 경고 메시지 출력)
- 2단계를 통해 전달받은 서버측 랜덤 데이터와 클라이언트 랜덤 데이터를 조합해 pre master secret 키를 생성함
- **서버가 제공한 공개키**로 이 pre master secret 키를 암호화하여 서버로 전송

4. 서버는 클라이언트가 전송한 pre master secret 값을 자신의 비공개키로 복호화
- 서버와 클라이언트 모두 pre master secret 값을 공유 하게 됨.
- pre master secret 은 session key 값을 생성 하는데 이 값을 이용하여 서버와 클라이언트간에 데이터를 대칭키 방식으로 암호화 한 후 주고받음 

5. 클라이언트와 서버의 handshake 종료

## Nginx 에 SSL 인증서 적용하기

- nginx 설정 파일

```bash
	server {
        listen       80;
        listen       [::]:80;
        server_name  [도메인 명];
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        return 301 https://$host$request_uri;
   }

   server {
        listen 443 ssl;
        server_name [도메인 명];

        ssl on;
        ssl_certificate [ssl 인증서 키 경로 key.crt 파일];
        ssl_certificate_key [ssl 인증서 키 경로 개인키 rsa 파일];
        ssl_prefer_server_ciphers on;
        location / {
                proxy_pass http://localhost:8080;
        }
        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
     }
```

- 인터넷 브라우저에 [도메인url] 로 접속시 서버 내부에선 ssl 인증서를 통해 HTTPS 통신하고, 웹서버를 통해 8080포트에 연결된 WAS와 데이터를 주고받게됨