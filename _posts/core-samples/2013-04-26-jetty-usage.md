---
layout: post
category : jetty
title : jetty 使用
tagline: "Supporting tagline"
tags : [jetty]
---
{% include JB/setup %}


今天测试了一下jetty环境下的hsf webapp的部署，配置方法如下： 

1、在Jetty_HOME/target目录下放置一个taobao-hsf.sar
，为应用服务器提供HSF服务。

2、Jetty_HOME/start.ini的etc/jetty.xml下面放一行
etc/jetty-hsf.xml，
这个jetty-hsf.xml中的内容如下： 

{% highlight html %}

<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure.dtd">

<Configure id="Server" class="org.eclipse.jetty.server.Server">
        <Call name="addBean">
                <Arg>
                        <New class="com.taobao.hsf.thirdcontainer.jetty.HSFContainerDelegator">
                                <Arg>./target</Arg>
                        </New>
                </Arg>
        </Call>
</Configure>
~               
{% endhighlight %}
   

com.taobao.hsf.thirdcontainer.jetty.HSFContainerDelegator
这个类在Jetty_HOME/lib/ext/hsf.thirdcontainer.jetty-1.0.0-SNAPSHOT.jar中 。

com.taobao.hsf.thirdcontainer.jetty.HSFContainerDelegator
这个类会去target目录下找taobao-hsf.sar，启动HSF环境。


3、在jetty_home/lib/ext下面放置一个hsf.thirdcontainer.jetty-1.0.0-SNAPSHOT.jar

4、修改Jetty_home/etc/jetty-deploy.xml文件，把下面这些配置注释掉：
{% highlight html %}
＜！--
<Call name="addAppProvider">
            <Arg>
              <New class="org.eclipse.jetty.deploy.providers.WebAppProvider">
                <Set name="monitoredDir"><Property name="jetty.home" default="." />/webapps</Set>
                <Set name="defaultsDescriptor"><Property name="jetty.home" default="."/>/etc/webdefault.xml</Set>
                <Set name="scanInterval">5</Set>
                <Set name="contextXmlDir"><Property name="jetty.home" default="." />/contexts</Set>
              </New>
            </Arg>
          </Call>
-->
{% endhighlight %}

注释的目的是防止同一个webapp被jetty两次加载,很重要； 
因为如果不去掉的话，jetty会重新加载一次webapp,而这时使用的
classloader是jetty自身的webapp classloader,会在启动jetty时
出现找不到hsf环境中的class的问题。


5、在jetty_home/contexts/下面放置一个文件
taobao-webapp-jetty.xml
内容如下：

{% highlight html %}

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure.dtd">
<Configure id="webContext" class="org.eclipse.jetty.webapp.WebAppContext">
        <Set name="classLoader">
                <New id="hsfWebAppClassLoader" class="com.taobao.hsf.thirdcontainer.jetty.HSFWebAppClassLoader">
                        <Arg ref="webContext" />
                </New>
        </Set>
        <Set name="contextPath">/</Set>
        <Set name="war">./webapps/juitemcenter</Set>
        <Set name="extractWAR">true</Set>
        <Set name="copyWebDir">false</Set>
    <Set name="configurationClasses">
        <Array type="java.lang.String">
              <Item>org.eclipse.jetty.webapp.WebInfConfiguration</Item>
              <Item>org.eclipse.jetty.webapp.WebXmlConfiguration</Item>
              <Item>org.eclipse.jetty.webapp.MetaInfConfiguration</Item>
              <Item>org.eclipse.jetty.webapp.FragmentConfiguration</Item>
              <Item>org.eclipse.jetty.webapp.JettyWebXmlConfiguration</Item>
              <Item>org.eclipse.jetty.webapp.TagLibConfiguration</Item>
              <Item>com.taobao.hsf.thirdcontainer.jetty.HSFWebInfConfiguration</Item>
        </Array>
    </Set>
</Configure>
~                 
{% endhighlight %}      


这段代码的关键是为webapps目录下的web应用创建一个自定义的app classloader,这样能够去提前加载HSF的class. 


