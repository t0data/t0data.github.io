---
layout: post
title: Burp Suite 实战指南
categories: 安全图书
description: BurpSuite实战
keywords: BurpSuite、web渗透、安全工具
---

## BurpSuite实战


<h2><a>引子</a></h2>
<p>刚接触web安全的时候，非常想找到一款集成型的<a href="http://www.secpulse.com/archives/category/tools" target="_blank">渗透测试工具</a>，找来找去，最终选择了<a href="http://www.secpulse.com/archives/44268.html" target="_blank">Burp Suite</a>，除了它功能强大之外，还有就是好用，易于上手。于是就从网上下载了一个破解版的来用，记得那时候好像是1.2版本，功能也没有现在这么强大。在使用的过程中，慢慢发现，网上系统全量的介绍<a href="http://www.secpulse.com/archives/44268.html" target="_blank">BurpSuite</a>的书籍太少了，大多是零星、片段的讲解，不成体系。后来慢慢地出现了不少介绍BurpSuite的视频，现状也变得越来越好。但每每遇到不知道的问题时，还是不得不搜寻BurpSuite的官方文档和英文网页来解决问题，也正是这些问题，慢慢让我觉得有必要整理一套全面的BurpSuite中文教程，算是为web安全界做尽自己的一份微薄之力，也才有了你们现在看到的这一系列文章。</p>
<p>我给这些文章取了IT行业图书比较通用的名称: 《BurpSuite实战指南》，您可以称我为中文编写者，文章中的内容主要源于BurpSuite官方文档和多位国外安全大牛的经验总结，我只是在他们的基础上，结合我的经验、理解和实践，编写成现在的中文教程。本书我也没有出版成纸质图书的计划，本着IT人互联分享的精神，放在github，做免费的电子书。于业界，算一份小小的贡献；于自己，算一次总结和锻炼。</p>
<p>以上，是为小记。</p>
<p>感谢您阅读此书，阅读过程中，如果发现错误的地方，欢迎发送邮件到 <a href="mailto:t0data@hotmail.com" target="_blank">t0data@hotmail.com</a>,感谢您的批评指正。</p>
<p>本书包含以下章节内容：</p>
<h4>第一部分 Burp Suite 基础</h4>
<ol>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#1F" target="_blank">Burp Suite 安装和环境配置</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#2F" target="_blank">Burp Suite代理和浏览器设置</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#3F" target="_blank">如何使用Burp Suite 代理</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#4F" target="_blank">SSL和Proxy高级选项</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#5F" target="_blank">如何使用Burp Target</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#6F" target="_blank">如何使用Burp Spider</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#7F" target="_blank">如何使用Burp Scanner</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#8F" target="_blank">如何使用Burp Intruder</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#9F" target="_blank">如何使用Burp Repeater</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#10F" target="_blank">如何使用Burp Sequencer</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#11F" target="_blank">如何使用Burp Decoder</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#12F" target="_blank">如何使用Burp Comparer</a></li>
</ol>
<h4>第二部分 Burp Suite 高级</h4>
<ol>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#13F" target="_blank">数据查找和拓展功能的使用</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#14F" target="_blank">BurpSuite全局参数设置和使用</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#15F" target="_blank">Burp Suite应用商店插件的使用</a></li>
<li><a href="https://www.gitbook.com/content/book/t0data/burpsuite/#16F" target="_blank">如何编写自己的Burp Suite插件</a></li>
</ol>
<h4>第三部分 Burp Suite 综合使用</h4>
<ol>
<li>使用Burp Suite测试Web Services服务</li>
<li>使用Burp, <a href="http://www.secpulse.com/archives/4213.html" target="_blank">Sqlmap</a>进行自动化SQL注入渗透测试</li>
<li>使用Burp、PhantomJS进行XSS检测</li>
<li>使用Burp 、Radamsa进行浏览器fuzzing</li>
<li>使用Burp 、Android Killer进行安卓app渗透测试</li>
</ol>
