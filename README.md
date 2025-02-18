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
