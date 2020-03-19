---
layout: post
title: Struts2中webconsole.html漏洞利用完全剖析
categories: 漏洞分析
description: Struts2中webconsole.html漏洞利用完全剖析
keywords: Struts2,webconsole.html,漏洞利用
---

近来，随着Struts2漏洞接二连三地被披露，企业对Struts2的心理安全指数也在降低，很多企业或者安全从业人员，在日常的安全检测或者扫描过程中有点草木皆兵，如果发现了webconsole.html页面，则判断系统存在Struts2漏洞，要求各个厂商做对应的安全整改。那么出现webconsole.html页面到底是不是Struts2的安全漏洞呢？

首先，我们来看看Struts2框架中为什么会存在webconsole.html。 在Struts的官网的Debuggin页面中，有如下一段话：
> The Debugging Interceptor provides three debugging modes to provide insight into the data behind the page. The xml mode formats relevant framework objects as an XML document. The console mode provides a OGNL command line that accepts entry of runtime expressions, and the browser mode **adds an interactive page that display objects from the Value Stack**. 
>

我们注意引文被我加粗标注出来的句子，从这里我们可以看出来，此交互页面（即struts/webconsole.html)是Struts官方为了方便开发人员进行Debug而提供的功能。基于此，我们应该有第一个基本的认识：**这是调试功能，只有在调试模式下才能使用。**

接着，我们看到此文档中还有如下语句的描述： 

![](/images/posts/img/s2/s2_01.png)

从这段话中，我们可以看出，Struts2的Debug功能，只有在开启了此功能，即配置了： <constant name="struts.devMode" value="true" /> 基于此，我们应该有第二个认识：**struts/webconsole.html的调试功能只有在启用了调试参数的情况下才会生效，否则即使看到此页面，也不具有调试的功能。**

那么，是不是说，只要开启了Debug模式,struts/webconsole.html就可以进行交互了呢？答案当然是否定的。当我们访问struts/webconsole.html，使用浏览器，按F12进行查看就会发现，webconsole.html页面中加载了几个js脚本。如下图所示：

![](/images/posts/img/s2/s2_02.png)

从图中我们可以看出，webconsole.html页面与后端交互时，使用了Dojo的js框架来完成请求和应答处理，也就是说，webconsole.html页面可以与后端进行正常交互的前提是，项目中使用了Dojo的lib库。而在Struts2中，有一个jar，专门供此功能使用的。如下图：

![](/images/posts/img/s2/s2_03.png)

基于此，我们应该有更深一层的认识：**只有在开启了Debug模式且ClassPath中使用了struts2-dojo-plugin-*.jar的情况下，webconsole.html页面才有可能存在安全漏洞的风险。**

这时我们该明白，如果应用程序中使用了Struts2，启用了Debug模式，且ClassPath中包含了struts2-dojo-plugin-*.jar，那么，当我们访问http://ip:port/app_name/struts/webconsole.html 即可以看到如图中所示的交互界面。但是，看到了界面，就真的可以交互了么？我们接下来看看一下webconsole.html中提交事件是如何触发的？

![](/images/posts/img/s2/s2_04.png)

如上图中的箭头所示，当我们在页面输入内容释放键盘操作时，调用的javascript函数为keyEvent(event)，我们再来跟踪一下这个函数，发现它存在于webconsole.js中，其关键代码如下图：


![](/images/posts/img/s2/s2_05.png)

当触发此函数时，需要两个参数，一个是event，一个是url。event表示键盘的动作事件，这比较好理解，那么url是什么呢？为什么此处并没有传递调用呢？在图中的53行我们看到源码如下： var the_url = url ? url : window.opener.location.pathname; 这句代码的语义是：如果存在url参数则使用url参数，如果不存在，则使用当前对象的父对象的url（相当于使用referer作为url的值）。很显然，此处url值不存在，只能使用当前对象的父对象的url。因此，要想能交互地使用此页面，必须有一个页面作为父页面，打开webconsole.html才可以。故我们需要有一个前置的页面，然后我们在页面上可以通过浏览器调试功能，添加如下代码： <a href="struts/webconsole.html" target="_blank">Click Me</a> 页面如图所示：

![](/images/posts/img/s2/s2_06.png)

当我们点击【Click Me】的超链接后，弹出的webconsole.html才是可交互的页面。

![](/images/posts/img/s2/s2_07.png)

基于此，我们应该有第四个认识：**webconsole.html是否能交互是需要有指定接受消息的url路径。**那么，是不是所有的url都可以进行交互呢？当然答案也是否定的。我们都知道webconsole.html主要用来进行Debug，其使用OGNL表达式，而OGNL表达式需要以Struts2的Action为入口，也就是说，通常情况下，这个url的路径类似于http://ip:port/app_name/xxx.action,是以action结尾的，且其实现类必须集成于com.opensymphony.xwork2.ActionSupport （后面的原理与S2-032一致），只有基于以上叙述的所有条件都满足的情况下，webconsole.html才是真正可交互的。

说到这里，我想你应该明白什么样的情况下你所看到的webconsole.html才是存在安全风险的。至于你可能出于安全目的，在禁用了devMode之后，仍然不希望其他人员看到webconsole.html页面，则可以直接删除webconsole.html的源文件，它的位置存在于：

![](/images/posts/img/s2/s2_08.png)

我们手工删除struts2-core-*.jar\org\apache\struts2\interceptor\debugging\文件夹下的browser.ftl、console.ftl、webconsole.html、webconsole.js即可。删除完毕后，当我们再次访问，将会出现404页面。



