# Nginx and Keycloak

http://site1.local (Public)
http://site2.local (protected with Basic Authentication)
http://site3.local (protected Keycloak Authentication)


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
docker-compose down
docker-compose up
```


### Notes to be deleted

generating self-signed certs
```
openssl req -x509 -newkey rsa:4096 -keyout ./nginx/certs/nginx.key -out ./nginx/certs/nginx.crt -days 365 -nodes -subj "/CN=localhost"
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




