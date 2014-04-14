## App Cache:

### How to use:

we create a **manifest file** listing all the assets the site needs:

```
CACHE MANIFEST
assets/6/script/mainmin.js
assets/6/style/mainmin.css
assets/6/style/fonts/pro.ttf
assets/6/style/imgs/sprites1.png
```

then link that manifest to the html page via an attribute:

```
<html manifest="offline.appcache">
```
The HTML page itself isn’t listed in the manifest. **Pages that associate with a manifest become part of it.**

### 1. 总是首先从缓存中载入，再更新，即使你在线上：FILES ALWAYS COME FROM THE APPLICATIONCACHE, EVEN IF YOU’RE ONLINE

当你访问一个离线应用时，你会立即得到缓存中的版本。一旦页面完成了渲染，浏览器再去检查mainfest和缓存文件的更新

如果有更新的话会fire一个`updateready`事件，但考虑到这个时候用户已经开始操作了，并不能刷新页面

但我们可以给出一个`有更新可用，请刷新页面`的提示，就像Google Reader和Google Gmail一样

### 2. 只有在manifest文件更新了才会更新内容：THE APPLICATIONCACHE ONLY UPDATES IF THE CONTENT OF THE MANIFEST ITSELF HAS CHANGED

如果你的mainfest文件中打算缓存50个页面，那么每次刷新页面总不能都去check一遍这50个页面有没有更新，所以比较好的做法是，只checkmanifest文件有没有更新

但html页面的地址不可能经常变动，于是可以采用更改注释的方法，比如

```
CACHE MANIFEST
# v1 whatever.html
```

### 3. 从mainfest中请求的文件，仍然会遵循http cache请求的规则：HE APPLICATIONCACHE IS AN ADDITIONAL CACHE, NOT AT ALTERNATIVE ONE

如果发出的请求的header中，max-age或者cache-control告诉浏览器没有过期，那么仍然不会向服务器发出请求

### 4. NEVER EVER EVER FAR-FUTURE CACHE THE MANIFEST

### 5. 没有被缓存的资源在缓存页面中是不会被加载的：ON-CACHED RESOURCES WILL NOT LOAD ON A CACHED PAGE

比如这个页面：http://appcache-demo.s3-website-us-east-1.amazonaws.com/without-network/

当然可以修改配置来禁止这样的行为

```
CACHE MANIFEST
# v1index.htmlNETWORK:
*
```
`*`表示浏览器允许请求没有缓存的文件

