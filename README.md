# Nginx and Keycloak

http://site1.local (Public)
http://site2.local (protected with Basic Authentication)
http://site3.local (protected Keycloak Authentication)
http://keycloak-server:8080/ (Keycloak admin)

edit /etc/hosts and add content below:
```
# dev
127.0.0.1 site1.local
127.0.0.1 site2.local
127.0.0.1 site3.local
127.0.0.1 keycloak-server
```

For site2 BasicAuth, to add new user or update password:
```
brew install httpd 
htpasswd -c nginx/auth/.htpasswd user1
```

```
docker-compose down --remove-orphans
docker-compose up
```


### Notes to be deleted

generating self-signed certs
```
openssl req -x509 -newkey rsa:4096 -keyout ./certs/server.key -out ./certs/server.crt -days 365 -nodes -subj "/CN=localhost"
```

Configure Keycloak
```
Login to Keycloak (http://localhost:8080/ with admin/admin).
Create a Realm: Name it myrealm.
Create a Client:
Name: myclient
Type: OpenID Connect
Redirect URIs: http://site3.local/*
Create Users:
Add users and set their passwords.
Enable the Authorization Feature for the Client.

Navigate to Realm Settings → Tokens
Set SameSite=None for cookies:
Find Cookie Settings
Set SameSite Policy: None
Enable Secure Cookies
```

```
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" http://site3.local/

curl -X POST "http://keycloak-server:8080/realms/myrealm/protocol/openid-connect/token" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "client_id=myclient" \
     -d "grant_type=password" \
     -d "username=myuser" \
     -d "password=mypassword"
```


## Troubleshooting

```
➜  tmp curl -k --verbose https://site3.local
* Host site3.local:443 was resolved.
* IPv6: (none)
* IPv4: 127.0.0.1
*   Trying 127.0.0.1:443...
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / x25519 / RSASSA-PSS
* ALPN: server accepted http/1.1
* Server certificate:
*  subject: CN=localhost
*  start date: Feb 18 14:29:24 2025 GMT
*  expire date: Feb 18 14:29:24 2026 GMT
*  issuer: CN=localhost
*  SSL certificate verify result: self-signed certificate (18), continuing anyway.
*   Certificate level 0: Public key type RSA (4096/152 Bits/secBits), signed using sha256WithRSAEncryption
* Connected to site3.local (127.0.0.1) port 443
* using HTTP/1.x
> GET / HTTP/1.1
> Host: site3.local
> User-Agent: curl/8.11.1
> Accept: */*
>
* Request completely sent off
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
< HTTP/1.1 302 Moved Temporarily
< Server: nginx/1.27.4
< Date: Tue, 18 Feb 2025 14:58:52 GMT
< Content-Type: text/html
< Content-Length: 145
< Connection: keep-alive
< WWW-Authenticate: Bearer realm="myrealm"
< Location: https://keycloak-server:8443/realms/myrealm/protocol/openid-connect/auth?client_id=myclient&redirect_uri=https://site3.local/&response_type=code
<
<html>
<head><title>302 Found</title></head>
<body>
<center><h1>302 Found</h1></center>
<hr><center>nginx/1.27.4</center>
</body>
</html>
```



client_secret = [keycloak-admin-UI/](https://keycloak-server:8443/admin/master/console/#/myrealm/clients/cc0ddfa5-870e-4561-bba1-b3cc180693d0/credentials)

clients -> credentials -> secret

```
curl --insecure -X POST "https://keycloak-server:8443/realms/myrealm/protocol/openid-connect/token" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "client_id=myclient" \
     -d "client_secret=EKtthXE8N6gpQ0IkmGUarpCDV0TgYMpO" \
     -d "grant_type=password" \
     -d "username=eduardo" \
     -d "password=123456"
```

The /userinfo endpoint requires a valid access token in the Authorization header. Without one, at least you'll verify connectivity.

```
curl --insecure -X GET "https://keycloak-server:8443/realms/myrealm/protocol/openid-connect/userinfo"
```