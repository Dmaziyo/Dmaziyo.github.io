---
title : 'JS和CSS以及DOM加载顺序'
date : 2024-03-13T13:05:17+08:00
draft : false
tags : ["前端基础"]
---

---
####  JS对DOM的影响
**验证方式**:
思路1：在html 标签内容前后写上script，然后打上断点，观察浏览器什么时候出现内容
![](https://raw.githubusercontent.com/Dmaziyo/blog-img/main/js-interceptDOM5.jpg)
结果:当断点出现在中途的时候，DOM的解析也停止了
思路2：script采用外部引入的方式，并且卡死请求时间，观察DOM状态
![](https://raw.githubusercontent.com/Dmaziyo/blog-img/main/js-interceptDOM.jpg)
结果：将script请求卡死后，浏览器只渲染出来了HelloWorld,而the world没有出来

**结论**:浏览器是线性加载文档的，当碰到JS脚本时，就会执行JS的内容，而后续内容得等JS脚本执行完成后再加载。


####  CSS对DOM的影响
**验证方式**:配置服务器提供接口返回css文件，并且在请求过程设置响应时间，在进行响应css文件的时候，同时使用js监听事件'DOMContentLoaded'，观察DOM和css之间的关系[参考链接]( https://juejin.cn/post/6973949865130885157#heading-3)
![](https://raw.githubusercontent.com/Dmaziyo/blog-img/main/css-intercepteDOM.jpg)
**结论**:CSS文件不会阻塞DOM文件解析


####  CSS对JS的影响
**验证方式**:配置服务器提供接口返回css文件，并且在请求过程设置响应时间，在进行响应css文件的时候，设置两组进行对照，一组是在css前面执行js，一组是在css后面执行js。观察是否有影响。
![](https://raw.githubusercontent.com/Dmaziyo/blog-img/main/css-js1.jpg)
结果：当css在script前面时，script被css阻塞了，无法执行
![](https://raw.githubusercontent.com/Dmaziyo/blog-img/main/css-js2.jpg)
结果：当script在css前面时，script正常运行
**结论**:CSS文件会阻塞JS文件加载，而又因为js会阻塞DOM的加载，所以CSS会间接阻塞DOM的加载.

**总结**：可以通过电影来解释，电影在拍一镜到底，但是可以暂停不能重开，css相当于服化道，DOM相当于演员。js相当于编剧。服化道和演员可以同时进行准备。但如果编剧中途修改了剧本，那么电影就得停下来等剧本修改完成后再拍,