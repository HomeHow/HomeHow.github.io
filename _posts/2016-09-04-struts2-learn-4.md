---
author: HomeHow
catalog: true
date: 2016-09-04T00:00:00.000Z
header-img: img/post-bg-js-module.jpg
layout: post
subtitle: 值栈
tags:
  - struts2学习
title: struts2学习笔记（四）
---

# 1 值栈 #
值栈是对应每一个请求对象的轻量级的内存数据中心，在这里统一管理着数据，供`Action`、`Result`、`Interceptor`等Struts2的其他部分使用，这样一来，数据被集中管理起来而不会凌乱，大大方便了程序编写。  
关于值栈的另外一个特性就是：大多数情况下，你根本无需关心值栈，你不用管它在哪里，不用管它里面有什么，你只需要去获取自己需要的数据就可以了。也就是说，你可以隐式的使用值栈。当然，如果编写自定义的Result或拦截器等较复杂功能的时候，还是需要显示访问值栈的。  
值栈能够线程安全的为每个请求提供公共的数据存取服务。当有请求到达的时候，Struts2会为每个请求创建一个新的值栈，也就是说，值栈和请求是一一对应的，不同的请求，值栈也不一样，而值栈封装了一次请求所有需要操作的相关的数据。正是因为值栈和请求的对应关系，因而值栈能保证线程安全的为每个请求提供公共的数据存取服务。  

## 2 ValueStack
狭义上，值栈通常指的是实现`com.opensymphony.xwork2.util.ValueStack`接口的对象，目前就是`com.opensymphony.xwork2.ognl.OgnlValueStack`类型。  
`OgnlValueStack`对象主要是用来支持OGNL（对象图导航语言）运算的。  
`com.opensymphony.xwork2.ognl.OgnlValueStack`部分源码中的如下：
{% highlight java %}
public class OgnlValueStack implements Serializable, ValueStack, ClearableValueStack, MemberAccessValueStack {
    public static final String THROW_EXCEPTION_ON_FAILURE = OgnlValueStack.class.getName() + ".throwExceptionOnFailure";
    private static final long serialVersionUID = 370737852934925530L;
    private static final String MAP_IDENTIFIER_KEY = "com.opensymphony.xwork2.util.OgnlValueStack.MAP_IDENTIFIER_KEY";
    private static final Logger LOG = LoggerFactory.getLogger(OgnlValueStack.class);
    CompoundRoot root;
    transient Map<String, Object> context;
    Class defaultType;
    Map<Object, Object> overrides;
    transient OgnlUtil ognlUtil;
    transient SecurityMemberAccess securityMemberAccess;
    private boolean devMode;
    private boolean logMissingProperties;
	
	......
	
}
{% endhighlight %}
在上述源码中，在 ValueStack 的内部有两个逻辑部分`CompoundRoot root`与`transient Map<String, Object> context`分别对应了对象栈与*context*。  
- `CompoundRoot root`：`CompoundRoot`类型继承自`ArrayList`类型，实际上是一个栈，Struts  把 Action 和相关对象压入 ObjectStack 中  
- `transient Map<String, Object> context`：实际上是 `OgnlContext` 类型, 是个 Map, 也是对 ActionContext 的一个引用. 里边保存着各种 Map: `requestMap`, `sessionMap`, `applicationMap`, `parametersMap`, `attr`  

## 2 ActionContext


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
例如：
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

### 2.2 ServletXxxtAware
通过实现 ServletRequestAware, ServletContextAware 等接口的方式，例如可以先**得到request接口后，再根据 request的方法去得到session和application**  
例如：
{% highlight java %}
public class TestServletAwareAction implements ServletRequestAware,ServletResponseAware,ServletContextAware{

    @Override
    public void setServletRequest(HttpServletRequest httpServletRequest) {
        System.out.println(httpServletRequest);
    }

    @Override
    public void setServletResponse(HttpServletResponse httpServletResponse) {
        System.out.println(httpServletResponse);
    }

    @Override
    public void setServletContext(ServletContext servletContext) {
        System.out.println(servletContext);
    }

    public String execute(){
        System.out.println("execute...");
        return "success";
    }
}
{% endhighlight %}
配置struts2.xml文件
{% highlight xml %}
<action name="TestServletAware" class="com.hhl.struts2.action.TestServletAwareAction">
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