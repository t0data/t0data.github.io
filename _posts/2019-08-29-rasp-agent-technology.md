---
layout: post
title: RASP技术划水
categories: 安全产品
description: RASP技术划水
keywords: RASP,Agent
---


> ```
> 作者:t0data,曾有幸参与过RASP产品的研发，本文尝试从RASP的基本功能入手，结合业界商业RASP的产品特性，讨论在互联网应用技术路线下，不同开发语言（Java/.NET/PHP/Python）或运行环境下的RASP Agent技术实现，粗浅地阐述RASP产品技术原理与功能特点，抛砖引玉，为RASP技术感兴趣的小伙伴们提供切入思路。
> ```

### 一、RASP的由来

​	RASP的全称是Runtime application self-protection,中文的含义是运行时应用自我保护，是Gartner 在2012年首次提出并将其定义：
​	 *<u>Runtime application self-protection (RASP) is a security technology that is built or linked into an application or application runtime environment, and is capable of controlling application execution and detecting and preventing real-time attacks.</u>*

​		按照Gartner的定义，RASP使用运行时检测技术来对所运行软件内部信息进行检测和阻止计算机攻击。该技术不同于基于周边其他的保护技术，比如防火墙它只能通过使用网络信息来检测和阻止攻击，无上下文感知。而RASP技术通过监视其输入并阻止那些可能允许攻击的技术来提高软件的安全性，同时保护运行时环境免受不必要的更改和篡改。

​	受RASP保护的应用程序较少依赖防火墙等外部设备来提供运行时安全保护，当检测到威胁时，RASP可以防止威胁被利用并采取其他操作进行阻止，比如终止用户会话，关闭应用程序，警告安全人员并向用户发送警告等。 RASP旨在缩小应用安全测试和网络边界控制所留下的空白地带，深入应用运行环境内部，实时了解数据和事件流，以监测或阻止开发过程中无法预见的新威胁。

​	在过去的几年里，不少安全厂商开始了RASP的落地技术实践，其中比较知名的有HP Application Defender、WARATEK  ApplicationSecurity for Java、OWASP AppSensor 、Shandowd、 Prevoty Runtime Application Security以及百度Open RASP等。无论是哪个厂商的产品，我们都可以看出来一个显著的特征：**RASP产品形态通常取决于应用程序的运行环境或应用程序服务器层，与之集成运行**。

从RASP的功能来看，能为客户带来高投入产出比的安全场景主要有：
1.无法进行迭代开发的或正在线上运行的老应用程序
2.应用程序中引入了大量的第三方组件库，而无法修改第三方组件源代码
3.修复应用程序需要花费大量的成本
4.防御某些未知类型的攻击

这些适用场景，是当前很多企业面临的困境，但当前业界所成型的产品，在性能上、防护能力上、运行环境覆盖上都存在或多或少的缺陷，这或许也是Gartner 在《2018年应用安全优先级矩阵》中将RASP技术定位为未来2~5年，RASP发展优先级、关注度、采用量为“高”的安全技术的原因。Gartner 期望RASP技术的发展和成熟，解决当前RASP自身的问题后，能极大地改变当前诸多企业应用安全的困境。

### 二、RASP的产品结构

RASP在产品定位上是应用层自适应的安全解决方案，目标是在不改变应用架构或应用代码的前提下，达到快速检测并阻止攻击的目的。一般来说，如果不改变应用架构或应用代码，那么RASP需要做不同的开发语言运行环境的适配，且易于安装和卸载；如果需要达到检测并阻止攻击，则RASP需要根据不同的攻击场景，定制不同的检测和响应模块；同时为了产品的易用性和运营需要，则RASP的数据需要完成可视化，通常与SIEM对接或有自己的管理平台来做数据分析。

我们以HP Application Defender、Prevoty Runtime Application Security、百度Open RASP三者为蓝本，并综合市场上其他新型RASP厂商的产品，RASP的产品形态主要分私有化部署和SaaS服务化接入两种形态。其中，私有化部署形态以老牌厂商为主，提供全套产品安装交付的方式在客户现场实施部署；而SaaS服务化接入形态以创新型安全厂商为主，中心化的云端部署SaaS服务，客户以应用接入的方式完成产品交付。但无论哪种形态的RASP，其产品架构通常都会包含管理端和接入端（或SDK）。

基于RASP产品形态和包含核心功能模块，通用的RASP产品结构图大体如下：

![](/images/posts/img/rasp/01.jpg)

**1.RASP SDK/Agent**
我们不妨先这样称呼SDK或Agent，总之，在RASP产品中存在着这样的一个构成模块，至于叫SDK/Agent哪个称呼更合适要取决于产品的最终交付形态。如果RASP产品最终是以SaaS服务形式对外提供安全防护能力，我们称为SDK似乎更恰当；如果RASP产品最终是以1个服务器端+N个终端接入的方式交付客户，则称为Agent似乎更合适。此模块的主要有两大块功能：**一是RASP与运行环境的兼容性适配，二是检测引擎对应用程序的安全防护**。同时，还可能会兼有安装、卸载、Agent自动更新、Agent自身软件保护、日志采集和发送等功能。它负责接受Server端的安全配置参数，监控/检测应用程序恶意行为，并为Server的决策提供流量数据。其产品模块构成大体如下图所示：
![](/images/posts/img/rasp/02.jpg)

**2.RASP Server**
RASP Server类似于通用应用软件的管理平台，即使在没有RASP Server的情况下，大多数RASP Agent的安全防护能力仍然可用，且因RASP Server不是我们本篇文章讨论的重点，故我们只做简单介绍。考虑在不同的产品交付形态下RASP Server的功能存在较大差异，我们在这里就以最大化SaaS服务交付形态的全集功能为主，来讨论RASP Server的产品构成。RASP Server为整个应用提供管理能力，其主要功能包含两大块：**一、对Agent端数据的收集和可视化呈现；二、Agent端的准入控制**。当然，也会有平台运营管理相关的模块，比如Agent健康性监控、配置参数更新下发、系统管理（账号、角色、权限）等。


### 三、RASP的基本原理

从上文中我们也可以明白，RASP Server端的功能与通用的安全管理平台类基本类似，所以，我们在这里讨论的RASP的技术原理主要以Agent为主，通过不同的运行环境或开发语言下Agent技术实现的阐述，来了解RASP的基本原理。根据各个语言的技术特点和市场产品的技术实现，对于RASP Agent的基本原理大体可分为如下两种类型：

#### **类型一：拦截器/过滤器类型**

此种类型的技术实现与拦截器或过滤器相似，故如此称呼。它们的基本原理的对请求或响应进行拦截/过滤处理，来达到应用保护的目的，我们可以称之为应用层WAF的类似产品，此种类型的产品严格意义上来说，不能称之为RASP产品。其技术原理如下：
**1.Java/.NET语言：**Java Web最流行的MVC框架主要为Spring、Struts1.x、Struts2，而Java Web程序的运行都依赖于Servlet容器，故Spring、Struts1.x、Struts2三大框架中，都遵循Servlet规范实现了自己的Filter；同时，利用Java的反射机制，也可以实现拦截器。它们都可以通过流量监控的形式，完成应用层的安全防护（*.NET框架下的IHttpModule与此类似*）。对此类技术细节不了解的，请阅读ESAPI的WAF模块源码。
**2.PHP语言：**与Java Web三大框架所不同，在PHP的应用中，由于运行环境的技术特点提供了两个先天入口：1）auto_prepend_file 在页面顶部加载文件；2）auto_append_file 在页面底部加载文件。这两个参数可以让应用部署时仅仅修改php.ini文件中auto_prepend_file与auto_append_file的值即可达到流量拦截的目的。PHP的这个特性，成为此类技术产品的切入点。
**3.Python语言**：Python在Web开发中常用框架主要有Django、Flask等，其实我们研究Flask就会发现，其框架中提供了@app.before_request、@app.after_request 和 @app.teardown_request这几个方法，在功能上与过滤器类型。当然，你也可以阅读Flask源码了解更多技术细节，从而解决其他开发框架的问题；或者基于Django的settings.py引用自定义中间件，利用中间件做拦截/过滤的工作，完成此类技术实现。

通过对上面几个开发语言技术分析，我们可以看出拦截器/过滤器类型的RASP产品，类似于传统的WAF产品，对所拦截的网络流量进行安全分析，它无法做到对应用程序中某个代码段或某个函数的行为做精准的安全防护从而导致误伤，这是它的缺点。但因为此类技术比较成熟，故市场上仍有不少此类技术的产品。

#### **类型二：运行时HOOK类型**

安全类的很多产品都是基于hook的思路，不同是产品之间比拼的往往是谁比谁hook得更底层，从这层意思上说，拦截器也算是hook的一种形态，只是我们在这里讨论的是如何基于运行环境做更底层的RASP hook。这类技术的RASP产品，是目前市场上的主流RASP技术实现手段，下面就从Java、PHP、Python三个主流编程语言来讨论此类技术细节。

##### 1.Java RASP的核心实现

在理解RASP技术实现之前，我们先来了解一下Java程序的基本原理。

###### **基本原理**

![](/images/posts/img/rasp/03.jpg)

如上图所示，Java程序从编码到运行主要分编译、字节码加载前、字节码加载后三个阶段，其中后两个阶段都在运行期，围绕这三个阶段来实现Java程序的拦截或hook技术主要分静态和动态两类，每一类又可分为不同的技术实现，现将各个技术的优缺点分析如下：
![](/images/posts/img/rasp/04.jpg)

从上表的分析我们可以看出，无论是基于JDK动态代理机制，还是基于Cglib+ASM实现的动态字节码生成机制，抑或者基于Javassist实现的自定义类加载器修改运行期字节码，都没有**字节码转换技术**的效果好。那么，字节码转换技术到底是什么呢？

字节码转换技术是Java 5 提供的新特性，开发者使用 Instrumentation，可以构建一个字节码转换器，在字节码加载前进行干预，以通过注入拦截点逻辑的方式起到hook的目的。其原理如下图所示：
![](/images/posts/img/rasp/05.jpg)

拦截器被注入到我们想要跟踪的方法中，并调用before())和after())方法来处理hook相关的业务逻辑， 通过hook点检测，Agent只能从必须hook的方法进行业务处理，这使得执行过程变得紧凑，性能影响更小。

###### **技术实现**

讲完了Java Hook的基本原理，下面我们就来看看Agent hook的基本技术实现。
使用 Instrumentation实现字节码转换，也就是在运行main方法之前，实现内定的方法名premain，关于此技术，网上很多例子，不明白的小伙伴可以阅读这篇文章（https://blog.csdn.net/xxzblog/article/details/79822681）。
我们在这里也使用此方法，而对于字节码的处理这里使用Bytebuddy，其核心代码如下：

```java
 public static void premain(String agentArgs, Instrumentation instrumentation) throws Exception {
        final PluginFinder pluginFinder;
        try {
        	//初始化默认配置参数
            DefaultConfig.initialize(agentArgs);
            //加载所有插件
            pluginFinder = new PluginFinder(new PluginBootstrap().loadPlugins());
        }catch (Exception e) {
            //agent初始化失败
            return;
        }

        final ByteBuddy byteBuddy = new ByteBuddy();
        //直接忽略白名单java类库，比如Bytebuddy、slf4j、log4j、javassist、asm、java反射类等
        AgentBuilder agentBuilder = new AgentBuilder.Default(byteBuddy)
            .ignore(DefaultConfig.Whitelist());

        try {
        	//注入instrumentation、Transformer、Listener
            agentBuilder
            	.type(pluginFinder.buildMatch())
            	.transform(new Transformer(pluginFinder))
            	.with(AgentBuilder.RedefinitionStrategy.RETRANSFORMATION)
            	.with(new Listener())
            	.installOn(instrumentation);
            //注册插件
            ServiceManager.boot();
        } catch (Exception e) {
        	return;
        }

        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
            @Override public void run() {
            	//注销插件
                ServiceManager.shutdown();
            }
        }, "service shutdown thread"));
    }
```

对于Transformer的实现里，需要通过插件查找器pluginFinder查询所有插件对应的Intercept，而每一个插件的Intercept实现里有Hook点的定义，通常划分为Constructors方法hook点，InstanceMethods方法hook点、StaticMethods方法hook点。比如说如果是Method类的hook，则需要实现before()方法和after()方法。而在before()方法或after()方法实现，统一定义在Check里，根据安全攻防的场景会做不同的具体Check实现类。当Check类检测后，发现存在异常行为，则调用不同Action类进行响应和处置。而对于Check实现类的具体实现也是RASP技术中的难点部分，如何精准地识别恶意输入并作出处置行为，是需要对攻防场景的深入理解的。其调用流程图如下：
![](/images/posts/img/rasp/06.png)

比如百度RASP的插件实现中，对SQL注入的识别算法是一种高效的创新，其原理是对比分析在有参数输入和无参数输入两种情况下，所执行的SQL语句的语义有无发生改变，尤其是参数的个数，当解析语义后的参数个数与无参数输入情况下的参数个数不一致时，则认为存在SQL注入。这比传统的SQL注入检测（通过sql保留字、特殊字符）来检测效果要好得多，误报率也会降低。当我们在实现某个具体的Check类时，尽量取众家之长为我所用，结合RASP的精准识别，来提高RASP的防护效果。

###### 小结

Java RASP的技术实现已比较成熟，在这一小节里我们通过Java Agent机制+ByteBuddy组件，详细的讲解了hook技术的细节，并结合RASP Agent的技术实现，讨论了关键领域模型中的安全插件、拦截器、检查器、动作等相互之间的关系，为想深入了解RASP技术的小伙伴提供基本信息引导。

##### 2.dotNET运行环境下RASP的核心实现

.NET环境下的RASP在市场上出现的比较早，但开源类的RASP产品至今未出现。和Java版本的RASP一样，我们仍然从技术的视角去分析Agent的技术实现。与其他开发语言相比，.NET因微软的封闭体系缘故所以我们在考虑实现时其运行环境要简单得多，准确地说我们更多的是考虑Asp.Net 的程序运行，帮助我们很好地避开了类似Java语言在Tomcat、Resin、WebLogic等不同中间件容器中的处理困境，下面我们来分析分析其Hook的基本原理。

###### 基本原理

首先我们来看看ASP.NET Framework应用程序运行原理。如下图：
![](/images/posts/img/rasp/07.png)

如图中所描述，IIS将客户端请求的aspx类型的页面分配给aspnet_isapi.dll，接着aspnet_isapi.dll通过Http PipeLine管道将请求发给ASP.NET Framework调用IIS worker进程（IIS6.0叫做 w3wq.exe，IIS5.0叫做 aspnet_wp.exe)，由HttpRuntime来处理这个Http请求，HttpRuntime根据请求调用相关接口，处理HttpRequest请求并作出HttpResponse响应。

而在ASP.NET Core中，其运行原理如下图所示：
![](/images/posts/img/rasp/08.jpg)

ASP.NET Core 应用程序是在.NET Core 控制台程序下调用特定的库，所有的ASP.NET托管库都是从`Program`开始执行，而不是由IIS托管。`Main`方法负责初始化Web主机，调用Startup和执行应用程序。主机将调用`Startup`类下面的`Configure`和`ConfigureServices`方法。一个ASP.NET Core 程序中，`Startup` 类是必须的。ASP.NET Core在程序启动时会从`Program`类中开始执行，然后再找到配置的`Startup`的类，如果不指定`Startup`类会导致启动失败。在`Startup`中必须定义`Configure`方法，而`ConfigureServices`方法则是可选的，方法会在程序第一次启动时被调用，类似传统的ASP.NET MVC的路由和应用程序状态均可在`Startup`中配置，也可以在此初始化所需中间件。

从ASP.NET Core 和ASP.NET Framework的运行原理我们初步可以判定，如果能在ASP.NET Core 程序的Startup阶段注入RASP Agent的入口，既解决了切入点问题，又回避了ASP.NET Framework依赖的IIS版本问题。那么ASP.NET Core 提供这样的入口么？

答案是有的，在ASP.NET Core的API中有IHostingStartup接口，开发人员可以编写这个接口的实现类，并利用ASPNETCORE_HOSTINGSTARTUPASSEMBLIES和DOTNET_ADDITIONAL_DEPS等参数设置，采用在启动时从外部程序集向应用中添加增强组件的方式，实现Automatic Agent的功能。而对于Hook点的拦截，ASP.NET Core也有类似AOP的机制，解决了入口问题，其他的也就好解决了。

###### 技术实现

下面我们就来看看核心编码实现。
采用IHostingStartup方式，加载RASP入口类。
![](/images/posts/img/rasp/09.jpg)
其中RaspAgentCore包含基本的环境变量设置、RASP配置以及RASP服务提供者，再通过RASP服务提供者加载各类场景的Interceptor实例。而对于Hook的拦截，采用Attribute方式进行处理。
自定义HookPoint：
![](/images/posts/img/rasp/10.jpg)
比如说，当hook点为HttpRequest时，则在对应的Interceptor里设置并处理：
![](/images/posts/img/rasp/11.jpg)



###### 小结

.NET环境下的RASP产品在市场上出现比较早，可惜一直没有使用过此类产品，本节仅仅从技术实现的角度，分析基本的原理并讨论Agent的切入点。

##### 3.Python运行环境下RASP的核心实现

关于python应用的RASP产品，目前市场比较少，更多的是在尝试推广阶段。

###### 基本原理

鉴于Python2和Python3的差异、不同的frameworks（比如[Django](https://www.djangoproject.com/)、[Flask](http://flask.pocoo.org/)、[Pyramid](https://trypyramid.com/)、[aiohttp](https://aiohttp.readthedocs.io/en/stable/)）以及不同的应用Servers（Django runserver、gunicorn、chaussette）等现状，RASP Agent需要解决这些兼容性问题才能提高RASP产品的易用性，才跟其他语言的RASP Agent产品一样，通过简单的安装或配置就达到为python应用提供安全防护的能力，这才是理想的。

在python的web开发中，有一个很重要的概念叫WSGI，全称是Web Server Gateway Interface，主要用来定义web服务器（nginx、apache、iis等）与 web应用（Flask、Django等）之间的接口规范。通过规范约束，遵循WSGI协议的 web服务器和 web应用可以完成server与application解耦，即可以有多个实现WSGI server的服务器，也可以有多个实现WSGI application的frameworks，那么就可以选择任意的server和application组合实现自己的web应用。例如uWSGI和Gunicorn都是实现了WSGI server协议的服务器，Django，Flask是实现了WSGI application协议的web框架。在实际项目中，可以根据项目的具体情况搭配使用，达到 web服务器和 web应用之间随意的组合目的。

[uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/)是一个web服务器，实现了WSGI协议，uwsgi协议，http协议等，旨在为部署分布式集群的python网络应用开发一套完整的解决方案。uWSGI服务器自己实现了基于uwsgi协议的server部分，使用者只需要在uwsgi的配置文件中指定application的地址，uWSGI就能直接和应用框架中的WSGI application通信。uwsgi的启动可以把参数加载命令行中，也可以使用不同类型的配置文件，诸如 .ini, .xml, .yaml 配置文件的形式加载参数，这个特性为Python的RASP Agent提供了较好的入口。除此之外，Python自身的import hook机制也为我们实现RASP Agent提供了方便的入口，利于开发者去拓展RASP细节性的功能。

###### 技术实现

通过上文的分析，我们大概了解了为了解决RASP Agent与Python application的兼容性，使用与uWSGI服务器的方式是当前情况下一种较好的解决方案。下面我们就来看看技术实现的细节。

我们先来看看入口函数，其基本实现是启用后台保护线程，运行RASP Agent：
![](/images/posts/img/rasp/12.jpg)

在HOOK导入函数，通过import hook机制注入各个HOOK点。
![](/images/posts/img/rasp/13.jpg)

在检测引擎的里，通过类初始化和回调机制，连接hook点和响应策略

![](/images/posts/img/rasp/14.jpg)



callback作为检测规则的全集，包含不用攻击场景的判定，并根据判断结果，传递给下一个环节，做出事件响应，最终呈现给前端用户。

###### 小结

通过修改WSGI文件的方式引入RASP，能解决不同python web框架下的兼容性问题，但仍存在非web server场景下的应用无法覆盖，比如使用manage.py shell启动的Django应用、Celery Workers应用等，此实现方式还有很多需要改进的地方。



### 总结

本文从RASP产品的基本功能开始，围绕产品定位和当前市场上RASP产品的形态，以Agent的技术实现为主，概要性的讲述了RASP产品在不同的运行环境（Java、.NET、Python）下的实现原理，并通过部分源代码的逻辑展示，粗浅了讨论了RASP的技术细节。当然，当前环境下，RASP产品还是属于新生事物阶段，产品的兼容性、性能都需要经历实践应用的验证来增强使用者的信心。随着RASP技术的发展和成熟，笔者相信未来会出现更多优秀的RASP类型产品，能做到用户所期望的那样，通过简单的部署或安装即可提供系统的安全防护能力，弥补当前市场安全产品在应用层安全防护能力的不足。

*注：由于本文涉及多个编程语言技术，水平有限，难免出现错误，欢迎交流、指正，邮件请发送t0data@hotmail.com！*

### 参考资料

1.https://uwsgi-docs.readthedocs.io/en/latest/Configuration.html

2.https://www.iteye.com/topic/1116696

3.https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/platform-specific-configuration?view=aspnetcore-2.2

4.https://www.cnblogs.com/luyuwei/p/3673789.html

5.https://www.oldboyedu.com/zuixin_wenzhang/index/id/873.html
