# Authenticating Users with OIDC

## 配置Keycloak
在Keycloak上配置一个新的client：
- Client ID: `oidc-playground`
- Access Type: `public`
- Root URL: `http://localhost:8000`
- Valid Redirect URIs: `http://localhost:8000/*`
- Web Origins: `+`


## Run application

因为使用Keycloak 19，所以需要
- 修改`app.js`，修改Keycloak地址为`http://localhost:8080`
- 修改`index.html`，修改`input-issuer`为`http://localhost:8080/realms/myrealm`

```bash
npm install
npm start
```

## Access application

http://localhost:8000/

修改Issuer为`http://localhost:8080/realms/myrealm`

点击"Load OpenID Provider Configuration"


OIDC标准endpoint:
- `<base URL>/.well-known/openid-configuration`
- Example: `http://localhost:8080/realms/myrealm/.well-known/openid-configuration`

重要endpoint：
- `authorization_endpoint`: ⽤于⾝份验证请求的 URL
- `token_endpoint`: ⽤于令牌请求的 URL
- `introspection_endpoint`: ⽤于⾃省请求的 URL
- `userinfo_endpoint`: ⽤于 UserInfo 请求的 URL
- `grant_types_supported`: ⽀持的授权类型列表
- `response_types_supported`：⽀持的响应类型列表

Keycloak supports the `authorization_code` grant type and the `code` and `token` response types.

## 验证用户过程

1. **⽤⼾**点击应⽤程序中的 **登录（Login）** 按钮。
2. **应⽤程序**⽣成**认证请求 (authentication request)**。
3. 认证请求以**302 重定向**的形式发送给⽤⼾，指⽰**⽤⼾代理**重定向到 Keycloak 提供的 **授权端点（authorization endpoint)** 。
4. ⽤⼾代理通过认证请求使⽤应⽤程序指定的查询参数打开授权端点。
5. Keycloak 向⽤⼾显⽰登录⻚⾯。⽤⼾输⼊他们的⽤⼾名和密码并提交表单。
6. Keycloak 验证了⽤⼾的凭据后，它会创建⼀个**授权码（authorization code)**，并将其返回给应⽤程序。
7. 应⽤程序现在可以⽤**授权码**交换 **ID 令牌（ID Token)**以及**刷新令牌(Refresh Token)**。


Test:
1. Click "Generate Authentication Request"
2. Click "Send Authentication Request", redirect to Keycloak login page

Authentication Request:
```bash
http://localhost:8080/realms/myrealm/protocol/openid-connect/auth

client_id=oidc-playground
response_type=code
redirect_uri=http://localhost:8000/
scope=openid
```

Keycloak login page url:
```bash
http://localhost:8080/realms/myrealm/protocol/openid-connect/auth?client_id=oidc-playground&response_type=code&redirect_uri=http://localhost:8000/&scope=openid
```

`scope=openid`表示发起的是认证请求

Login with users created under `myrealm` realm.

Authentication Response (authorization code):
```bash
code=c703afd1-ca49-4d8c-9aa2-c4f7ed27fc1d.34d77f58-b609-4085-98e5-f34202b9c640.9f3bc13e-936d-4eba-b3cf-fa3689534dd2
```

## 换取令牌

Token Request:
```bash
http://localhost:8080/realms/myrealm/protocol/openid-connect/token

grant_type=authorization_code
code=c703afd1-ca49-4d8c-9aa2-c4f7ed27fc1d.34d77f58-b609-4085-98e5-f34202b9c640.9f3bc13e-936d-4eba-b3cf-fa3689534dd2
client_id=oidc-playground
redirect_uri=http://localhost:8000/
```

authorization code一分钟内有效，且只能用一次。

Token Response:
```json
{
  "access_token": "...",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "...",
  "token_type": "Bearer",
  "id_token": "...",
  "not-before-policy": 0,
  "session_state": "34d77f58-b609-4085-98e5-f34202b9c640",
  "scope": "openid profile email"
}
```

同时返回了access_token, refresh_token, id_token。
其中access_token的有效期为5分钟。
Token的类型是Bearer，不记名令牌，谁拿到谁就可以用。
session_state：这是⽤⼾与 Keycloak 的会话 ID。

## 了解ID Token

签名的JWT格式：
```bash
<Header>.<Payload>.<Signature>
```

ID Token - header:
```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "n69HmaitvshGuv8SowVCQPBjFlfjHZId1fXN19vMMBU"
}
```

ID Token - payload:
```json
{
  "exp": 1659423789,
  "iat": 1659423489,
  "auth_time": 1659422748,
  "jti": "3041f0f3-0159-49f0-bbfa-de8313344539",
  "iss": "http://localhost:8080/realms/myrealm",
  "aud": "oidc-playground",
  "sub": "899147e8-b028-4123-8804-974712838738",
  "typ": "ID",
  "azp": "oidc-playground",
  "session_state": "34d77f58-b609-4085-98e5-f34202b9c640",
  "at_hash": "md_v42BlsWt1jmRIaVCDKA",
  "acr": "0",
  "sid": "34d77f58-b609-4085-98e5-f34202b9c640",
  "email_verified": false,
  "name": "John Dae",
  "preferred_username": "john",
  "given_name": "John",
  "family_name": "Dae",
  "picture": "https://avatars.githubusercontent.com/u/110177654?s=200&v=4"
}
```

Signature

通常，ID 令牌的持续时间很短，以降低令牌泄露的⻛险。这并不意味着应⽤程序必须重新验证⽤⼾⾝份，⽽是
有⼀个单独的刷新令牌可⽤于获取更新的 ID 令牌。刷新令牌的有效期更⻓，只能直接与 Keycloak ⼀起使⽤，这
意味着 Keycloak 可以验证令牌是否仍然有效。

## 刷新令牌

Refresh request:
```bash
http://localhost:8080/realms/myrealm/protocol/openid-connect/token

grant_type=refresh_token
refresh_token=...
client_id=oidc-playground
scope=openid
```

Refresh response:
```json
{
  "access_token": "...",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "...",
  "token_type": "Bearer",
  "id_token": "...",
  "not-before-policy": 0,
  "session_state": "34d77f58-b609-4085-98e5-f34202b9c640",
  "scope": "openid profile email"
}
```

ID Token:
```json
{
  "exp": 1659424309,
  "iat": 1659424009,
  "auth_time": 1659422748,
  "jti": "daccb6eb-624d-4724-9c86-c7231dac4da1",
  "iss": "http://localhost:8080/realms/myrealm",
  "aud": "oidc-playground",
  "sub": "899147e8-b028-4123-8804-974712838738",
  "typ": "ID",
  "azp": "oidc-playground",
  "session_state": "34d77f58-b609-4085-98e5-f34202b9c640",
  "at_hash": "UUsZZXoCUrgxNFjrWWunfQ",
  "acr": "0",
  "sid": "34d77f58-b609-4085-98e5-f34202b9c640",
  "email_verified": false,
  "name": "John Dae",
  "preferred_username": "john",
  "given_name": "John",
  "family_name": "Dae",
  "picture": "https://avatars.githubusercontent.com/u/110177654?s=200&v=4"
}
```



## 用户信息

常用的用户信息属性：
- "name": "John Dae" // 用户全名，显示用
- "preferred_username": "john" // 用户账号，登录用，不能修改
- "given_name": "John" // 名
- "family_name": "Dae" // 姓
- "email": "john@example.com" // 邮箱地址

修改用户信息
- `given_name`
- `family_name`
- `email`

刷新token，在ID Token中可以看到新的用户信息。

### 在ID Token中添加自定义属性

User attributes:
- `myattribute`: `myvalue`

Client scopes:
- `myclaim`

Mappers in `myclaim` client scope:
Configure a new mapper
Choose "User Attribute"

- Name: `myattribute`
- User Attribute: `myattribute`
- Token Claim Name: `myattribute`
- Claim JSON Type: `String`
- Ensured checked "Add to ID token"

Add created client scope `myclaim` into `oidc-playgournd` client as optional client scope.

We're doing this as we want to show how a client can request different information
from Keycloak using the `scope` parameter. This allows the client to **only request the information it needs** at any given time, which is especially useful when the user
is required to consent to access from the application.

Re-authentication in "OpenID Connect Playground" application.

Chang scope as `openid myclaim`.

Run authenticaiton, and then token.

Check if ID Token has `"myattribute": "myvalue"`

小结：
1. attributes可以设置到user级别，都会返回这些attributes
2. 也可以设置到client级别，在认证请求时提供指定的scope才会返回对应的attributes


### 在ID Token中添加角色

By default, roles are not added to the ID token. You can change this behavior by going to `Client Scopes`, then selecting the `roles` client scope. Click on `Mappers`, then select `realm roles`. Turn on "Add to ID Token", and click Save.

Keycloak 19已经默认开启"Add to ID Token"

在ID Token中没有`realm_access`，但是Access Token中有.

修改`client.js`和`index.html`在Token页显示Access Token。

## 调用Userinfo端点

Userinfo request:
```bash
http://localhost:8080/realms/myrealm/protocol/openid-connect/userinfo

Authorization: Bearer <access_token>
```

Userinfo response:
```json
{
  "sub": "899147e8-b028-4123-8804-974712838738",
  "email_verified": false,
  "name": "Johnny Bush",
  "preferred_username": "john",
  "given_name": "Johnny",
  "locale": "en",
  "family_name": "Bush",
  "picture": "https://avatars.githubusercontent.com/u/110177654?s=200&v=4",
  "email": "john@example.com"
}
```

### Add custom information

协议映射器?


## 用户注销

当 Keycloak 收到注销请求时，它会通知同⼀会话中的其他客⼾端注销。然后它将使会话⽆
效，这实际上使所有令牌⽆效。

### 使用ID和Access token过期

使用ID和Access token过期机制，在注销后，无法refresh token，等到ID和access token过期后，就不能再访问了。

有几分钟延迟，不适合长久有效的token的情况。

### 使用OIDC会话管理



### Leveraging OIDC Back-Channel Logout

For **server-side applications**, using the back-channel logout is fairly effective. It does,
however, become a bit complex to deal with for clustered applications with **session stickiness**. 


### A note on OIDC Front-Channel Logout


### 小结

1. 使用短的session和token有效期
2. 需要必须立即注销的情况，使用OIDC Back-Channel Logout

Front channel logout:
- When true, logout requires a browser redirect to client. 
- When false, server performs a background invocation for logout.


在浏览器中查看缓存信息，F12打开developer tool / Application
- Local Storage
  - accessToken
  - idToken
  - refreshToken
- Cookies
  - connect.sid







