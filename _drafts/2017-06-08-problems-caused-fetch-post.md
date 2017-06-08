# 由fetch接口post数据而引起的一系列疑问

你有没有突然发现，有一段代码使用了很久，或许是从stackoverflow复制过来，但是却从来没有想过为什么要这么写。

## 使用原生fetch接口post数据

前几天尝试使用原生的`fetch`接口向后台传输数据，当然是使用`post`方法。后端服务是由基于Node.js的Express框架提供。作为DEMO，前端代码非常简单：

```javascript
var request = new Request('/upload', {
    method: 'POST',
    body: {
        'city': 'Beijing'
    }
});

fetch(request);
```