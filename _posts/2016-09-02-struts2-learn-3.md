---
author: HomeHow
catalog: true
date: {  }
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
**注意：**public Object get(Object key): ActionContext 类中没有提供类似 getRequest() 这样的方法来获取 HttpServletRequest 对应的 Map 对象. 要得到 HttpServletRequest 对应的 Map 对象, 可以通过为 get() 方法传递 “request” 参数实现  
例如：
{% highlight java linenos %}
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
