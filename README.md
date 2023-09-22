# 一种基于iframe的去端口访问方法

## 现有的方案

### 1.基于 DNS 的 HTTPS & SVCB 记录

优点:最好的实用性,易用性,配置完成就可以直接使用

缺点:主流浏览器不支持,目前只有苹果的Safari支持,兼容性暂且蒙古

### 2.基于HTTP Alternative Services的替代服务

项目地址 [lan-expose](https://github.com/shiyunjin/lan-expose)

优点:切实可行的技术

缺点:兼容性其实不咋样,配置较复杂,需要服务器,仅支持HTTPS

### 3.基于虚拟组网

优点:最好的安全性(配置正常的情况下),兼容性MAX

缺点:必须要安装软件加入网络,配置较繁琐,需要服务器

### 4.反向代理(一般是海外CDN)

优点:配置简单,兼容性MAX

缺点:一般来说延迟拉胯,数据要出国再回国,原地TP

### 5.类似Cloudflare Tunnels的内网穿透

优点:兼容性MAX,配置不算简单

缺点:跟CDN差不多

## 一个全新的思路

### 基于iframe的套娃方案

优点:兼容性较强,配置较简单,无需任何新技术,单静态页面可以直接丢Cloudflare pages

缺点:实际上没有真的在443端口提供服务,只是在浏览器表面去掉了端口,网站favicon.ico没法正常加载(应该是可以解决的)

 [示例页面](https://iframe-742.pages.dev/)

待解决的问题:无法方便的获取favicon.ico

### 如何配置?

一个父页面文件,参考我的index.html文件即可,只需要修改目标页面

子页面需要加入一段JS脚本,脚本内容也在index.html文件中,我只做了两个示例(脚本和人有一个能跑就行)

你可以使用Nginx替换来批量给网站添加脚本(Nginx替换需要关闭后端的Gzip压缩功能)

也可以直接在网页中添加脚本,反正能加进去就行

主页面Nginx配置中需要加入两个命令,如果把页面丢到Cloudflare pages,则不需要配置

```
#强制跳转到index.html
location / {
    try_files '' /index.html;
    }


location = /index.html {
    index  index.html;
}
```

<br/>

<br/>

### 配置思路  (一堆批话,直接看index.html文件最好)

众所周知,iframe可以在网页中套娃一个网页

但是套娃的子页面跟父页面几乎没有任何关系

这里以网站A与网站B举例,A/\*是父页面,B/\*是子页面

我们来一步一步的解决这些问题 

首先是URL与标题

通过在子页面中嵌入一个脚本,在达成条件时使用postMessage向父页面发送一次信息

父页面配置好接收消息,收到消息后解析然后修改标题与URL

这样就完成了子页面到父页面的标题与URL同步

但是父页面到子页面的同步还没完成

你直接使用A/111访问就会发现404了,因为A/实际上没有这个目录

所以你需要配置一下Nginx(使用cloudflare pages无需配置)

把全部请求指向index.html

然后在index中处理URL

获取A/111,然后用B替换A,最后把iframe的src属性改为B/111

这样就完成了父页面到子页面的URL同步
