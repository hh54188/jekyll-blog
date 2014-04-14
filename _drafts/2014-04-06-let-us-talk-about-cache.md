## Local Storage

早在2011年Google和Bing就是最早将localstorage使用于手机端的网站之一

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


### 6. App Cache没法配合响应式，只能把全分辨率下的图片下载下来

### 7. 我们没法知道缓存页面是如何失败的

当页面跨域时，或者40x,50x时，或者是网络请求失败时，进入fallback页面

- 好处是如果用户在线但是站点垮了，浏览器会立即显示缓存数据，很好的用户体验
- 坏处是，我们没法获得确切垮掉的原因

### 8. 禁止跨域

### 9. 可以通过XHR去请求缓存资源，但是通常会失败，

因为Webkit浏览器返回的`statuscode`为0，但是不一定其他浏览器也有这个bug

```
$.ajax( url ).always( function(response) {
 // Exit if this request was deliberately aborted
 if (response.statusText === 'abort') { return; } // Does this smell like an error?
 if (response.responseText !== undefined) {
  if (response.responseText && response.status < 400) {
   // Not a real error, recover the content    resp
  }
  else {
   // This is a proper error, deal with it
   return;
  }
 } // do something with 'response'
});
```

### 从离线的Wikipedia聊起

http://diveintohtml5.info/offline.html#fallback

每一个页面都指向一个空的mainfest文件，当用户浏览这些页面时，这些页面便成了缓存的一部分

但这么做是灾难性的！

用户不知道哪些页面被缓存了，开发者也不知道，没有对应的API。即使能获得manifest文件，但它也无法告诉我们哪些页面被缓存了

为了能够让用户看到最新的页面，根据上面的规则2，我们应该修改manifest文件，但多频繁更新一次？只要有一个wikipedia页面更新了，我们的manifest文件就更新？这样可能导致更新失败

假设我们已经访问了上千个页面，那么一旦manifest更新之后，上千个页面也要更新？appcache并没有一个移出的机制


### 我们的离线需求是什么

- 在线的时候能展现实时的数据，离线的时候也可以

- 能够让开发者控制什么被缓存，如何被缓存

- 能够让用户来控制缓存，比如“离线收藏”按钮

- 每一个被访问过的页面都能够离线访问

### App Cache最好可以用于静态资源文件

### 使用Localstorage来弥补不足

比如我们想保存`articles/1.html`时，我们可以这么干：

```
// Get the page content
var pageHtml = document.body[removed];
var urlPath = location.pathName;

// Save it in local storage
localStorage.setItem( urlPath, pageHtml );

// index用于保存缓存信息，让我们能知道究竟什么页面被缓存了

// Get our index
var index = JSON.parse( localStorage.getItem( 'index' ) );

// Set the title and save it
index[ urlPath ] = document.title;
localStorage.setItem( 'index', JSON.stringify( index ) );
```

当我们想离线访问`1.html`时：

```
var pageHtml = localStorage.getItem( urlPath );
if ( !pageHtml ) {
    document.body[removed] = `Page not available';
} else {
    document.body[removed] = localStorage.getItem( urlPath );
    document.title = localStorage.getItem( 'index' )[ urlPath ];
}

```



