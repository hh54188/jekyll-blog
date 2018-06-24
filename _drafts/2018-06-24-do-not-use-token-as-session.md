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

更具体的说，上面用于调用 API 的 token 我们称为细分为 access token。通常 access token 是有有效期限的，如果过期就需要重新获取。那么如何重新获取？现在我们要让时光倒流一会，回顾第一次获取 token 的流程是怎样的:

1. 首先你需要向 Google API 注册你的应用程序，注册完毕之后你会拿到证书信息（credentials）包括
 ID 和 secret。不是所有的程序类型都有 secret, 比如 JavsScript 程序就没有。
2. 接下来就要向 Google 请求 access token。这里我们先忽略一些细节，例如请问参数（当然需要上面申请到的 secret）以及不同类型的程序的请求方式等。重要的是，如果你想访问的是用户资源，这里就会提醒用户进行授权。
3. 如果用户授权完毕。Google 就会返回 access token。又或者是返回授权代码（authorization code），你再通过代码取得 access token
4. token 获取到之后，就能够带上 token 访问 API 了

流程如下图所示：
![google token flow](./images/token-as-session/webflow.png)

注意在第三步通过 code 兑换 access token 的过程中，Google 并不会仅仅返回 access token，还会返回额外的信息，这其中和之后更新相关的就是 refresh token

一旦 access token 过期，你就可以通过 refresh token 再次请求 access token。

以上只是大致的流程，比如更新 access token 当然也可以不需要 refresh token，这要根据你的请求方式和访问的资源类型而定。

这里又会引起另外的两个问题：
1. 如果 refesh token 也过期了怎么办？这就需要用户重新登陆授权了
2. 为什么要区分 refrsh token 和 access token ？如果合并成一个 token 然后调整过期时间稍长，并且每次失效之后用户重新登陆授权就好了？这个问题会和后面的相关概念有关，稍后再回答


