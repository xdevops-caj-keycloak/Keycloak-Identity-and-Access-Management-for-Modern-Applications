# Secure your first application

## Keycloak配置

### 基本配置
- Realm: `myrealm`
- Group: `mygroup`
- Role: `myrole`
- User: `john` (any users you like)

### Keyclok Client配置

- Client: `myclient`
- Client Type: `openid-connect`
- Root URL: `http://localhost:8000`
- Valid Redirect URIs: 
    - `http://localhost:8000/*` (登录成功后跳转的地址)
- Web Origins: `+` (That adds all the Valid Redirect URIs that you defined above to the Web Origins headers.)

### keycloak.json

在Client详情页，点击Actions，选择Download adapter config，查看keycloak.json


Ref:
- https://stackoverflow.com/questions/59018604/keycloak-no-access-control-allow-origin-header-is-present-on-the-requested-r


## User profile配置
Attributes:
- `picture`: `https://avatars.githubusercontent.com/u/110177654?s=200&v=4`

## 程序修改

### 前端程序修改
因为使用Keycloak 19，默认URL后面没有`auth`，所以要将`frontend/app.js`中的Keycloak地址改为`http://localhost:8080`.

### 后端程序修改
因为使用Keycloak 19，默认URL后面没有`auth`，所以要将`backend/keycloak.json`中的Keycloak地址改为`http://localhost:8080`.

## 访问前端

运行程序：
```bash
npm install
npm start
```

前端：
http://localhost:8000/

1. **Login** with previous created `john` user.

2. Show ID Token
Let's take a look at the ID token that Keycloak issued. 
Click on the **Show ID Token** button. 
The ID token is used by the application to establish the identity of the authenticated user.

3. Show Access Token
Next, let's take a look at the access token. Click on the **Show Access Token** button
Currently, the information within the tokens are the default fields available in Keycloak.
If you want to add additional information, Keycloak is very flexible in allowing you to
customize the contents within the tokens.

4. Show picture in user profile
参见上面的“User profile配置”

5. **Refresh** Access Token

## 访问后端

运行程序：
```bash
npm install
npm start
```

后端：
http://localhost:3000/

1. Click on the **Public endpoint link**. You will see a message saying Public message!. 

2. click on the **Secured endpoint link**. Now you will see a message saying Access denied.

When you click **Invoke Service**, the frontend sends an AJAX request to the backend service, including the access token in the request, which allows the backend to verify that the invocation is done on behalf of a user who has the required role to access the endpoint.

## Logout

在`myclient`的client settings中
- `Valid post logout redirect URIs`: `http://localhost:8000/*`

Ref.
- https://stackoverflow.com/questions/71984843/keycloak-using-react-user-can-login-but-when-i-try-logout-i-get-a-message-inva
