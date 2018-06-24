# 不要把 JWT 用作 session 管理

通常为了弄清楚一个概念，我们需要掌握十个概念。在判断 JWT 是否适用于 session 管理之前，我们要了解什么是 token，以及 access_token 和 refresh_token 的区别；了解什么是 OAuth，什么是 SSO，authorisation 和 authentication 的不同，SSO 下不同策略 OAuth 和 SAML 的不同，以及 OAuth 与 OpenID 的不同。最后我们引出 JSON WEB TOKEN，聊聊 JWT　在 session 管理方面的优势和劣势，同时尝试解决这些劣势，看看成本和代价有多少

本文关于 OAuth 授权和 API 调用实例都来自 Google API。

## 关于 Token

token 即使是在计算机领域中也有不同的定义，这里我们说的token，是指**访问资源的凭据**。例如当你调用Google API，需要带上有效 token 来表明你请求的合法性。这个 token 是 Google 给你的，这代表 Google 给你的授权，你有权力调用 API，访问 API 背后的资源。就好比你有权力在图书馆借阅图书之前你需要办理会员卡，并且每次去的时候都要带上会员卡。

请求 API 时携带 token 的方式也有很多种，通过 HTTP Header 或者 url 参数 或者 google 提供的类库都可以：

```
// HTTP Header:
GET /drive/v2/files HTTP/1.1
Authorization: Bearer <token>
Host: www.googleapis.com/

// URL query string parameter
GET https://www.googleapis.com/drive/v2/files?token=<token>

// Python:
from googleapiclient.discovery import build
drive = build('drive', 'v2', credentials=credentials)
```

