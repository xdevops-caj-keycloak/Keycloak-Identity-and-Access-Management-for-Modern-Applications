# Integrate Spring Boot application with Keycloak

## 应用程序介绍

前端应用frontend:
- spring-boot-starter-oauth2-client
- spring-boot-starter-security

后端应用backend:
- spring-boot-starter-oauth2-resource-server
- spring-boot-starter-security
- `class SecurityConfig extends WebSecurityConfigurerAdapter`

## 修改应用程序

### 修改frontend

修改`frontend/src/main/resources/application.yaml`:
1. 设置端口为`8081`.

```yaml
server:
  port: 8081
```

2. 修改Keycloak的地址为`http://localhost:8080`

### 修改backend

修改`frontend/src/main/resources/application.yaml`:
1. 设置端口为`8081`.

```yaml
server:
  port: 8081
```

2. 修改Keycloak的地址为`http://localhost:8080`


## 配置Keycloak 

### Keycloak Clients

在`myrealm` realm下创建clients：

Server-side web application:
- Client ID: `mywebapp`
- Root URL: `http://localhost:8081`
- Valid redirect URIs: `http://localhost:8081/*`
- Web origins: `+`
- Access Type: `Confidential`
- Authentication flow: `Standard flow`

The client using to protect the backend application:
- Client ID: `mybackend`
- Root URL: `http://localhost:8081`
- Valid redirect URIs: `http://localhost:8081/*`
- Web origins: `+`
- Access Type: `Confidential`
- Authentication flow: `Standard flow`, `Direct Access Grants Enabled`

> Tips: 在生产上不建议开启`Direct Access Grants Enabled`，开启后可以通过用户名和密码直接和Keycloak交换token，而无需通过Keycloak登录页面，也无需先换code，再拿code换token。
> This enables support for Direct Access Grants, which means that client has access to username/password of user and exchange it directly with Keycloak server for access token. In terms of OAuth2 specification, this enables support of 'Resource Owner Password Credentials Grant' for this client.

### Keycloak Users

- Username : `john`


## 访问应用程序

### 访问frontend

运行程序：
```bash
mvn spring-boot:run
```

访问`http://localhost:8081/`，跳转到Keycloak登录页面，输入用户名密码登录后，跳转到首页。

### 访问backend

运行程序：
```bash
mvn spring-boot:run
```

以Direct Asscess方式直接通过username和password获取Token（而无需先换code，再换token：
```bash
# env variables
CLIENT_SECRET=<client_secret>
USER=<username>
USER_PASSWORD=<password>

# curl
export access_token=$( \
 curl -X POST http://localhost:8080/realms/myrealm/protocol/openid-connect/token \
   --user mybackend:$CLIENT_SECRET \
   -H 'content-type: application/x-www-form-urlencoded' \
   -d "username=${USER}&password=${USER_PASSWORD}&grant_type=password" | jq -r '.access_token' \
)

# httpie
export access_token=$( \
  http --form POST http://localhost:8080/realms/myrealm/protocol/openid-connect/token \
    username=${USER} password=${USER_PASSWORD} grant_type=password \
    --auth mybackend:${CLIENT_SECRET} | jq -r '.access_token' \
)
   
```

Invoke the backend service with retrived access token:
```bash
# curl
curl -X GET \
 http://localhost:8081 \
 -H "Authorization: Bearer "$access_token

# httpie
http http://localhost:8081 Authorization:"Bearer $access_token"
```

References:
- https://www.ctl.io/developers/blog/post/curl-vs-httpie-http-apis

## References

- https://docs.spring.io/spring-security/reference/index.html
- https://spring.io/guides/tutorials/spring-boot-oauth2/
- https://github.com/spring-projects/spring-security-samples
- [Spring Security OAuth2 Login](https://docs.spring.io/spring-security/reference/servlet/oauth2/login/index.html#oauth2login)
- https://docs.spring.io/spring-security-oauth2-boot/docs/current/reference/html5/

到目前来看Spring有以下方式和Keycloak集成包括
1. Spring提供的方式
- spring-boot-starter-oauth2-client
- spring-boot-starter-oauth2-resource-server
- spring-security-oauth2-boot

2. Keycloak提供的adatper方式
- keycloak-spring-boot-starter
- keycloak-spring-security-adapter