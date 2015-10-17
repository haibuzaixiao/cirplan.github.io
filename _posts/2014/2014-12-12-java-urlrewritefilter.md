---
layout: post
category : JAVA
title: java的urlrewritefilter简单使用
tags : [java, urlrewrite]
---
{% include JB/setup %}

由于项目中只用了`tomcat`，没有用`Apache`，然后想做`www`重定向。就如让所有访问`baidu.com`都301永久重定向到`www.baidu.com`。然后时间紧急，对`Apache`不是很熟悉（懒的去配置），就直接用了`java`的一个包：
`urlrewritefilter`。

官网是：[http://tuckey.org/urlrewrite/][1]。使用方法也很简单，这里简单说下。

### 1.下载 ###
先去下载`urlrewritefilter-x.x.x.jar `包。下载好放到`WEB-INF/lib`目录下。

### 2.配置web.xml ###
修改web.xml，把下面代码放到所有servlet mappings的前面。

	<filter>
	    <filter-name>UrlRewriteFilter</filter-name>
	    <filter-class>org.tuckey.web.filters.urlrewrite.UrlRewriteFilter</filter-class>
	</filter>
	<filter-mapping>
	    <filter-name>UrlRewriteFilter</filter-name>
	    <url-pattern>/*</url-pattern>
	    <dispatcher>REQUEST</dispatcher>
	    <dispatcher>FORWARD</dispatcher>
	</filter-mapping>

这里是对应的参数文档 [filterparams][2]。这就配置完了（这里有个很坑爹的问题，待会说）。

### 3.增加 `urlrewrite.xml` ###
然后在 `WEB-INF` 文件夹里增加多 `urlrewrite.xml` 文件。

下面是官方标准urlrewrite.xml文档：

	<?xml version="1.0" encoding="utf-8"?>
	<!DOCTYPE urlrewrite PUBLIC "-//tuckey.org//DTD UrlRewrite 4.0//EN"
	        "http://www.tuckey.org/res/dtds/urlrewrite4.0.dtd">

	<!--

	    Configuration file for UrlRewriteFilter
	    http://www.tuckey.org/urlrewrite/

	-->
	<urlrewrite>

	    <rule>
	        <note>
	            The rule means that requests to /test/status/ will be redirected to /rewrite-status
	            the url will be rewritten.
	        </note>
	        <from>/test/status/</from>
	        <to type="redirect">%{context-path}/rewrite-status</to>
	    </rule>


	    <outbound-rule>
	        <note>
	            The outbound-rule specifies that when response.encodeURL is called (if you are using JSTL c:url)
	            the url /rewrite-status will be rewritten to /test/status/.

	            The above rule and this outbound-rule means that end users should never see the
	            url /rewrite-status only /test/status/ both in thier location bar and in hyperlinks
	            in your pages.
	        </note>
	        <from>/rewrite-status</from>
	        <to>/test/status/</to>
	    </outbound-rule>


	    <!--

	    INSTALLATION

	        in your web.xml add...

	        <filter>
	            <filter-name>UrlRewriteFilter</filter-name>
	            <filter-class>org.tuckey.web.filters.urlrewrite.UrlRewriteFilter</filter-class>
	            <init-param>
	                <param-name>logLevel</param-name>
	                <param-value>WARN</param-value>
	            </init-param>
	        </filter>
	        <filter-mapping>
	            <filter-name>UrlRewriteFilter</filter-name>
	            <url-pattern>/*</url-pattern>
	        </filter-mapping>

	     EXAMPLES

	     Redirect one url
	        <rule>
	            <from>/some/old/page.html</from>
	            <to type="redirect">/very/new/page.html</to>
	        </rule>

	    Redirect a directory
	        <rule>
	            <from>/some/olddir/(.*)</from>
	            <to type="redirect">/very/newdir/$1</to>
	        </rule>

	    Clean a url
	        <rule>
	            <from>/products/([0-9]+)</from>
	            <to>/products/index.jsp?product_id=$1</to>
	        </rule>
	    eg, /products/1234 will be passed on to /products/index.jsp?product_id=1234 without the user noticing.

	    Browser detection
	        <rule>
	            <condition name="user-agent">Mozilla/[1-4]</condition>
	            <from>/some/page.html</from>
	            <to>/some/page-for-old-browsers.html</to>
	        </rule>
	    eg, will pass the request for /some/page.html on to /some/page-for-old-browsers.html only for older
	    browsers whose user agent srtings match Mozilla/1, Mozilla/2, Mozilla/3 or Mozilla/4.

	    Centralised browser detection
	        <rule>
	            <condition name="user-agent">Mozilla/[1-4]</condition>
	            <set type="request" name="browser">moz</set>
	        </rule>
	    eg, all requests will be checked against the condition and if matched
	    request.setAttribute("browser", "moz") will be called.

	    -->

	</urlrewrite>

我们经常用到的就两个：

	 //重定向 地址栏显示的是实际地址
	    <rule>
	        <from>/some/olddir/(.*)</from>
	        <to type="redirect">/very/newdir/$1</to>
	    </rule>

	//转发 地址了显示的是虚拟地址
	    <rule>
	        <from>/products/([0-9]+)</from>
	        <to>/products/index.jsp?product_id=$1</to>
	    </rule>
	eg, /products/1234 will be passed on to /products/index.jsp?product_id=1234 without the user noticing.

匹配的是java的正则表达式。

### 4.www301重定向 ###
下面就是我要用到的：
	
	//如果地址的域名不是 www.baidu.com 就会跳转到www.baidu.com
	//为了考虑开发情况，把 localhost 也加到里面
	<rule>
	    <name>seo redirect</name>
	    <condition name="host" operator="notequal">^www.baidu.com</condition>
	    <condition name="host" operator="notequal">^localhost</condition>
	    <from>^/(.*)</from>
	    <to type="permanent-redirect" last="true">http://www.baidu.com/$1</to>
	</rule>  

到这里为止，一切都正常。但是我用转发却不行。

	<rule>
	    <from>/products/([0-9]+)</from>
	    <to>/products/index.jsp?product_id=$1</to>
	</rule>

访问`localhost:8080/products/123`一直报`404`，而且还是乱码。

这个问题纠结了好久。`api`里也没看到哪里出问题了。最后，终于发现，是转发后被`struts`拦截了。所以，要修改`web.xml`。在`struts2`的`<filter-mapping>`里加上：

	<dispatcher>REQUEST</dispatcher>  
	<dispatcher>FORWARD</dispatcher> 

就可以了。浪费好多时间去弄这个问题，囧~

参考：

*  [http://xlaohe1.iteye.com/blog/1130854][3]
*  [http://fanzhongyun.iteye.com/blog/1221170][4]
*  [http://blog.csdn.net/ygf215/article/details/4766285][5]
*  [http://nematodes.org/martin/2010/02/04/301-permanent-redirect-with-tomcat-howto/][6]

[1]: http://tuckey.org/urlrewrite/
[2]: http://urlrewritefilter.googlecode.com/svn/trunk/src/doc/manual/4.0/index.html#filterparams
[3]: http://xlaohe1.iteye.com/blog/1130854
[4]: http://fanzhongyun.iteye.com/blog/1221170
[5]: http://blog.csdn.net/ygf215/article/details/4766285
[6]: http://nematodes.org/martin/2010/02/04/301-permanent-redirect-with-tomcat-howto/