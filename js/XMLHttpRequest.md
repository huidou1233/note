XMLHttpRequest 是一个API, 它为客户端提供了在客户端和服务器之间传输数据的功能。它提供了一个通过 URL 来获取数据的简单方式，并且不会使整个页面刷新。这使得网页只更新一部分页面而不会打扰到用户。XMLHttpRequest 在 AJAX中被大量使用。

XMLHttpRequest是一个Javascript对象。通过它，你可以很容易的取回一个URL上的资源数据。而且除了HTTP，它还支持file和ftp协议。

属性：

| 属性                 | 描述                                       |
| ------------------ | ---------------------------------------- |
| onreadystatechange | 一个JavaScript函数对象，当readyState属性改变时会调用它。回调函数会在user interface线程中调用。 |
| readyState         | 请求的5中状态：0:对象已创建，未调用open()1: open()方法成功调用，但send()方法还未调用2: send()方法已经调用，响应头和响应状态已经返回3: 响应体下载中；responseText 中已经获取了部分数据。4: 请求完成,即响应数据接受完成 |
| response           | 响应实体的类型由responseType来指定，可以是ArrayBuffer, Blob, Document,Javascript对象[即JSON]，或者是字符串。如果请求失败或未完成，则该值为null |
| responseText       | 此次请求的响应为文本，或是当请求未成功或还未发送时为 null。只读。      |
| responseType       | 设置该值能够改变响应类型。就是告诉服务器你期望的响应格式。“” (空字符串):      字符串(默认值)，“arraybuffer"： ArrayBuffer“blob”:  Blob,“document”: Document“json”: JavaScript对象，解析自服务器传递回来的JSON 字符串。“text”: 字符串 |
| status             | 该请求的响应状态码(状态码： 200 ,表示一个成功的请求).只读        |
| statusText         | 该请求的响应状态信息，包含一个状态码和原因短语 (如： ”200 OK“)；只读 |
| upload             | 可以在upload上添加一个事件监听来跟踪上传过程                |
| withCredentials    | 表明在进行跨站(cross-site)的访问控制(Access-Control)请求时，是否使用认证信息(例如cookie或授权的header)。 默认为 false。 |

方法：

**abort**() :如果请求已经被发送，则立刻终止请求

**getAllResponseHeaders**(): 返回所有响应头信息(响应头和值)，如果响应头还没接受，则返回null

**getResponseHeader**(String header): 返回HTTP响应头中指定的键名header对应的值，如果响应头还没被接受,或该响应头不存在,则返回null.

**open**(string **method**, string **url**, optional boolean **async**, optional string **user**, optional string **password**): 初始化一个请求。该方法用于JavaScript代码中;如果是本地代码, 使用 [openRequest()](https://developer.mozilla.org/zh-cn/nsIXMLHttpRequest#openRequest())方法代替.

​	method: 请求所使用的HTTP方法；例如“GET”,"POST","PUT","DELETE"等。如果下个参数是非HTTP(S)的URL，则忽略该参数。

​	url： 该请求所要访问的URL

​	async： 一个可选的布尔值参数，默认为true，意味着是否执行异步操作，如果值为false，则send()方法不会返回任何东西，直到接受到了服务器的返回数据。如果值为true，一个对开发者透明的通知会发送到相关的事件监听者。这个值必须是true,如果multipart 属性是true，否则将会出现一个意外。

​	user: 用户名,可选参数,为授权使用;默认参数为空string.

​	password: 密码,可选参数,为授权使用;默认参数为空string.

**send**(): 发送请求. 如果该请求是异步模式(默认),该方法会立刻返回. 相反,如果请求是同步模式,则直到请求的响应完全接受以后,该方法才会返回.

**setRequestHeader**(string header, string value): 给指定的HTTP请求头赋值。在这之前，你必须确认已经调用open()方法打开一个url

​	header:  将要被赋值的请求头名称

​	value:  给指定的请求头赋的值

**overrideMimeType**():重写有服务器返回的MIME type。这个可用于, 例如，强制把一个响应流当作"text/xml"来处理和解析，即使服务器没有指明数据是这个类型。注意，这个方法必须在send()之前被调用。