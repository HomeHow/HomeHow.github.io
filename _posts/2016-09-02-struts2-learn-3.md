---
author: HomeHow
catalog: true
date: 2016-09-02T00:00:00.000Z
header-img: img/post-bg-js-module.jpg
layout: post
subtitle: 访问web资源
tags:
  - struts2学习
title: struts2学习笔记（三）
---

# 访问web资源 #
在 *Action* 中, 可以通过以下方式访问 web 的 `HttpSession`, `HttpServletRequest`, `HttpServletResponse`  等资源。

1. 与 Servlet API 解耦的访问方式。  
- 通过`com.opensymphony.xwork2.ActionContext`
- 通过 Action 实现如下接口  
  `org.apache.struts2.interceptor.ApplicationAware`
  `org.apache.struts2.interceptor.RequestAware`
  `org.apache.struts2.interceptor.SessionAware`
  `org.apache.struts2.interceptor.ParameterAware`
2. 与 Servlet API 耦合的访问方式  
- 通过`org.apache.struts2.ServletActionContext`  
- 通过实现对应的`xxxAware`接口  

## 1 与 Servlet API 解耦的访问方式
为了避免与 *Servlet API* 耦合在一起, 方便 *Action* 做单元测试, *Struts2* 对 `HttpServletRequest`, `HttpSession` 和 `ServletContext` 进行了封装, 构造了 3 个 *Map* 对象来替代这 3 个对象, 在 *Action* 中可以直接使用 `HttpServletRequest`,` HttpServletSession`, `ServletContext` 对应的 *Map* 对象来保存和读取数据。  

### 1.1 使用 ActionContext
`ActionContext` 是 Action 执行的上下文对象, 在 ActionContext 中保存了 Action 执行所需要的所有对象, 包括 `parameters`, `request`, `session`, `application` 等。  
获取 `HttpSession` 对应的 Map 对象:  
{% highlight java %}
public Map getSession()
{% endhighlight %}
获取 `ServletContext` 对应的 Map 对象:  
{% highlight java %}
public Map getApplication()
{% endhighlight %}
获取`parameters`对应的 Map 对象:  
{% highlight java %}
public Map getParameters()
{% endhighlight %}
获取 `HttpServletRequest` 对应的 Map 对象:  
**注意：**`public Object get(Object key)`: `ActionContext` 类中没有提供类似 `getRequest()` 这样的方法来获取 `HttpServletRequest` 对应的 Map 对象. 要得到 `HttpServletRequest` 对应的 Map 对象, 可以通过为 `get()` 方法传递 `request` 参数实现  
例如：
{% highlight java %}
public class TestActionContextAction {
    public String execute(){
        ActionContext actionContext = ActionContext.getContext();

        Map<String,Object> applicationMap = actionContext.getApplication();
        applicationMap.put("applicationKey","applicationValue");

        Object date = applicationMap.get("date");
        System.out.println("date:" + date);

        Map<String,Object> sessionMap = actionContext.getSession();
        sessionMap.put("sessionKey","sessionValue");
        if (sessionMap instanceof SessionMap){
            SessionMap sm = (SessionMap) sessionMap;
            sm.invalidate();
            System.out.println("Session have invalidated!");
        }

        Map<String,Object> requestMap = (Map<String,Object>) actionContext.get("request");
        requestMap.put("requestKey","requestValue");

        Map<String,Object> parameter = actionContext.getParameters();
        System.out.println(((String[]) parameter.get("name"))[0]);

        return "success";
    }
}
{% endhighlight %}
配置struts2.xml文件
{% highlight xml %}
<action name="TestActionContext" class="com.hhl.struts2.action.TestActionContextAction">
	<result name="success">/test-actionContext.jsp</result>
</action>
{% endhighlight %}
test-actionContext.jsp文件  
{% highlight html %}
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page language="java" contentType="text/html;charset=UTF-8" pageEncoding="UTF-8"%>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
</head>
<body>
    <h4>Test ActionContext Page</h4>

    application : ${applicationScope.applicationKey }
    <br><br>

    session : ${sessionScope.sessionKey }
    <br><br>

    request : ${requestScope.requestKey }
    <br><br>
</body>
</html>
{% endhighlight %}  

### 1.2 实现XxxAware接口
Action 类通过可以实现`RequestAware`,`SessionAware`,`ApplicationAware`等接口, 让 Struts2 框架在运行时向 Action 实例注入 `parameters`, `request`, `session` 和 `application` 对应的 Map 对象。  
以application为例：
{% highlight java %}
public class TestAwareAction implements ApplicationAware{
    public String execute(){

        application.put("applicationKey2","applicationValue2");

        System.out.println("date:" + application.get("date"));

        return "success";
    }

    private Map<String, Object> application;

    @Override
    public void setApplication(Map<String, Object> application) {
        this.application = application;
    }
}
{% endhighlight %}
配置struts2.xml文件
{% highlight xml %}
<action name="TestAware"  class="com.hhl.struts2.action.TestAwareAction">
	<result name="success">/test-aware.jsp</result>
</action>
{% endhighlight %}
test-aware.jsp文件  
{% highlight html %}
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page language="java" contentType="text/html;charset=UTF-8" pageEncoding="UTF-8"%>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
</head>
<body>
    <h4>Test ActionContext Page</h4>

    application : ${applicationScope.applicationKey2 }
    <br><br>
</body>
</html>
{% endhighlight %}  

## 2 与 Servlet 耦合的访问方式
直接访问 Servlet API 将使 Action 与 Servlet 环境耦合在一起,  测试时需要有 Servlet 容器, 不便于对 Action 的单元测试。  

### 2.1 通过ServletActionContext
**用ServletActionContext得到request再得到sesion和application**
直接获取 HttpServletRequest 对象：  
`ServletActionContext.getRequest()`
直接获取 HttpSession 对象：  
`ServletActionContext.getRequest().getSession()`
直接获取 ServletContext 对象：  
`ServletActionContext.getServletContext()`  
以application为例：
{% highlight java %}
public class TestServletActionContextAction {
    public String execute(){
        HttpServletRequest request = ServletActionContext.getRequest();
        HttpSession session = ServletActionContext.getRequest().getSession();
        ServletContext servletContext = ServletActionContext.getServletContext();

        System.out.println("execute...");

        return "success";
    }
}
{% endhighlight %}
配置struts2.xml文件
{% highlight xml %}
<action name="TestservletActionContextAction" class="com.hhl.struts2.action.TestServletActionContextAction">
    <result>/success.jsp</result>
</action>
{% endhighlight %}
success.jsp文件  
{% highlight html %}
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page language="java" contentType="text/html;charset=UTF-8" pageEncoding="UTF-8"%>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
</head>
<body>
    <h4>Success Page</h4>
</body>
</html>
{% endhighlight %}  