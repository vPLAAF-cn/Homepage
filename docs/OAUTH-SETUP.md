# vPLAAF 使用 APOC OAuth 登录配置说明

本站已接入 **pilot.apocfly.com（APOC 飞行员中心）** 的 OAuth2 登录，与论坛登录方式一致。后端 OAuth API 为 **https://api.apocfly.com/**（文档同 [api.half-nothing.cn](https://api.half-nothing.cn/)）。

论坛登录示例（注意 **id 不能为空**）：  
`https://pilot.apocfly.com/oauth2?app_name=APOC+%E8%AE%BA%E5%9D%9B&id=34&scopes=profile`

## 一、前端已实现内容

- **登录页**：`/login.html` — 点击「使用 APOC 账号登录」会跳转到 **pilot.apocfly.com/oauth2**。若未填写 `clientId`，会提示且**不会跳转**（避免出现 `id=` 为空）。
- **回调页**：`/login-callback.html` — 接收授权码、校验 state，并通过 **tokenProxyUrl** 代理换取 token。
- **配置脚本**：`js/oauth-config.js` — 授权页参数：`app_name`、`id`、`scopes`，以及 `redirect_uri`、`response_type`、`state`、PKCE。

## 二、你需要在 APOC 侧完成

1. **申请 OAuth 客户端（client_id 必填，否则授权页 id 为空）**  
   - 在 APOC 后台或通过 [创建客户端](https://api.half-nothing.cn/408688404e0) 接口创建 OAuth 应用，获取 **client_id**（论坛示例在授权 URL 中为 `id=34`，可能是数字或字符串）。  
   - 同时获取 **client_secret**（换 token 时必须，且不能放在前端，故必须使用 `api/token` 代理）。

2. **在客户端配置中设置 redirect_uri**  
   必须与本站实际回调地址**完全一致**，例如：
   - 本地：`http://127.0.0.1:5500/login-callback.html`
   - 线上：`https://你的域名/login-callback.html`

3. **接口地址**  
   - **授权页**：`https://pilot.apocfly.com/oauth2`（与论坛一致，参数含 `app_name`、`id`、`scopes`）。  
   - **换 token**：`https://api.apocfly.com/api/oauth/token`，要求 **POST application/json**，且必填 **client_id、client_secret**（见 [获取访问 token](https://api.half-nothing.cn/408639179e0)），因此浏览器必须通过 **tokenProxyUrl** 代理换 token。

## 三、前端配置（必做）

在 **`js/oauth-config.js`** 中填写：

```javascript
config.clientId = '你在 APOC 申请到的 client_id';  // 不能为空，否则授权页 id= 会为空
```

可选：修改展示名称（授权页显示的应用名）：

```javascript
config.appName = 'vPLAAF 官网';
```

确认 `config.redirectUri` 与在 APOC 后台配置的 redirect_uri 一致（脚本会按当前站点自动生成）。

## 四、换 token 必须使用代理（client_secret 不能放前端）

根据 [获取访问 token](https://api.half-nothing.cn/408639179e0)，接口要求 **client_id、client_secret** 且请求体为 **application/json**，因此必须通过服务端代理换 token，不能在前端直连。

### 使用本站提供的 serverless 代理

1. 将 **`api/token.js`** 部署到支持 Serverless 的平台（如 Vercel）。  
2. 环境变量：`APOC_OAUTH_CLIENT_ID`、`APOC_OAUTH_CLIENT_SECRET`；可选 `APOC_OAUTH_TOKEN_URL`（默认 `https://api.apocfly.com/api/oauth/token`）。  
3. 在 `js/oauth-config.js` 中设置：  
   `config.tokenProxyUrl = 'https://你的部署地址/api/token';`

## 五、相关链接

- [pilot.apocfly.com 论坛登录示例](https://pilot.apocfly.com/oauth2?app_name=APOC+%E8%AE%BA%E5%9D%9B&id=34&scopes=profile)
- **API**：[https://api.apocfly.com/](https://api.apocfly.com/)
- [认证请求](https://api.half-nothing.cn/408641146e0)（GET /oauth/authorize，参数 client_id、redirect_uri、response_type、scope、state、code_challenge、code_challenge_method）
- [获取访问 token](https://api.half-nothing.cn/408639179e0)（POST /oauth/token，application/json，必填 client_id、client_secret）
- [创建客户端](https://api.half-nothing.cn/408688404e0)、[撤销令牌](https://api.half-nothing.cn/408694877e0)、[修改授权状态](https://api.half-nothing.cn/408912607e0) 等
- 本站登录页：`/login.html`

完成上述配置后，用户即可在 vPLAAF 站点通过「使用 APOC 账号登录」完成与论坛一致的 pilot.apocfly.com OAuth 登录。
