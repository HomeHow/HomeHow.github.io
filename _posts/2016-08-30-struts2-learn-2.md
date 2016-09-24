---
layout: post
title: struts2学习笔记（二）
subtitle: Action以及result配置
date: 2016-08-30T
author: HomeHow
header-img: img/post-bg-js-module.jpg
catalog: true
tags:
  - struts2学习
---

# 1 Action简述 #

*action*应用程序可以完成的每一个操作. 例如: 显示一个登陆表单; 把产品信息保存起来。  
Action类: 普通的 Java 类,可以有属性和方法, 同时必须遵守下面这些规则:
   
* **属性的名字必须遵守与 JavaBeans 属性名相同的命名规则。**属性的类型可以是任意类型. 从字符串到非字符串(基本数据库类型)之间的数据转换可以自动发生  
* **必须有一个不带参的构造器**  
* **至少有一个供 struts 在执行这个 action 时调用的方法**  
* **同一个 Action 类可以包含多个 action 方法.**  
* **Struts2 会为每一个 HTTP 请求创建一个新的 Action 实例**  

*Struts2*的核心功能是*action*，对于开发人员来说，使用*Struts2*主要就是编写*action*,例如实现``HelloAction``的时候有**三种方法**:    

**1)** 实现`com.opensymphony.xwork2.Action`接口，并实现该接口中的`execute()`方法  

**2)** 在实际开发中，*action*类很少直接实现*Action*接口，通常都是从`com.opensymphony.xwork2.ActionSupport`类继承，例如
{% highlight java%}
public class HelloAction extends ActionSupport {  
      
    private static final long serialVersionUID = 1L;  
  
    public String execute(){  
        return "success";  
    }  
}  
{% endhighlight %}

*ActionSupport*实现了*Action*接口和其他一些可选的接口，提供了输入验证，错误信息存取，以及国际化的支持，选择从*ActionSupport*继承，可以简化*action*的定义。  
**3)** *Struts2*并不是要求所有编写的*action*类都要实现*Action*接口，也可以直接编写一个普通的Java类作为*action*，只要实现一个返回类型为String的无参的*public*方法即可，例如：

{% highlight java%}
public class HelloAction {  
      
    public String execute(){  
        return "success";  
    }  
}  
{% endhighlight %}


# 2 Action的配置 #

开发好*action*之后，好需要对*action*进行配置，以告诉*Struts2*框架，针对某个*URL*的请求应该交由哪个*action*进行处理。  

## 2.1 Action映射 ##
*action*映射是*Struts2*框架中的基本“工作单元”，*action*映射就是将一个请求*URL*(即*action*的名字)映射到一个*action*类，当一个请求匹配某个*action*的名字时，框架就使用这个映射来确定如何处理请求。例如：  
{% highlight xml %}
<action name="product-save" class="com.hhl.struts2.helloworld.Product" method="save">
	<result name="details">/WEB-INF/pages/details.jsp</result>
</action>
{% endhighlight %}
*action*元素的主要属性与子节点如下：  
**name**：对应一个*struts2*的请求的名字（或对一个*servletPath*，但去除‘/’和扩展名），不包含拓展名  
**class**：默认值为：``com.opensymphony.xwork2.ActionSupport``  
 *ActionSupport*是默认的*Action*类，若某个 *action* 节点没有配置 *class*属性, 则 *ActionSupport* 即为
待执行的 *Action* 类。 而 *execute* 方法即为要默认执行的 *action* 方法  
**method**：的默认值为：``execute`` ，可以自定义方法，如``product-save``action中的``save``方法。
在配置action时，我们可以通过action元素的method属性来指定action调用的 方法，所指定的方法，必须遵循与execute方法相同的格式。 在struts2.xml文件中，我们可以为同一个action类配置不同的别名，并使用 method属性。  
**result**：结果,表示*action*方法执行后可能返回的一个结果,所以一个*action*节点可能会有多个*result*子节点，多个*result*子节点使用*name*来区分  

## 2.2 result配置 ##
结果，表示*action*方法执行后可能返回的一个结果，所以一个*action*节点可能会有多个*result*子节点，多个*result*子节点使用*name*来区分，例如：
{% highlight xml %}
<action name="testResult" class="com.hhl.struts2.action.TestResultAction">
<result name="success" type="dispatcher">/success.jsp</result>
<result name="login" type="redirect">/login.jsp</result>
<result name="index" type="redirectAction">
    <param name="actionName">testAction</param>
    <param name="namespace">/atHHL</param>
</result>
<result name="test" type="chain">
    <param name="actionName">testAction</param>
    <param name="namespace">/atHHL</param>
</result>
</action>
{% endhighlight %}  
*result*有两个属性，*name*与*type*：  
**1) name:**标识一个*result*，和*action*配置的*method*方法的可能的一个返回值相对应，默认值为*success*  
**2) type:**默认为**dispatcher**  
在``struts2.xxxx-core.jar/struts-default.xml``中定义的类型:  

| Type 类型值        | 作用说明           |
| ------------- |:-------------:|
| dispatcher（默认值）      | 用来转向页面，通常处理 JSP |
| redirect      | 重定向到一个URL      |
| redirectAction | 重定向到一个 Action      |
| chain| 用来处理Action 链|
| plainText | 显示源文件内容，如文件源码 |
| freemarker | 处理 FreeMarker 模板 |
| httpheader | 控制特殊 http 行为的结果类型 |
| stream | 向浏览器发送 InputSream 对象，通常用来处理文件下载，还可用于返回 AJAX 数据。|
| velocity | 处理 Velocity 模板 |
| xslt | 处理 XML/XLST 模板 |  

**1、dispatcher结果类型**  
   Struts2在后台使用Servlet API 的``RequestDispatcher``来转发请求，因此在用户的整个请求/响应过程中，目标Servlet/JSP接收到的``request``/``response``对象，与最初的Servlet/JSP相同。  
   Dispatcher结果类型的实现是``org.apache.struts2.dispatcher.ServletDispatcherResult``,该类的二个属性(property):``location``和``parse``，这两个属性可以通过struts.xml配置文件中的result元素的param子元素来设置。param元素的name属性指定结果类型实现类的属性名,param元素的内容是属性的值。例如:  
{% highlight xml %}
  <result name=“success”  type=“dispatcher”>
    <param name=“location” >/success.jsp</param>
     <param name=“parse” >true</param>
  </result>
{% endhighlight xml %}
  **说明：**  
  **A、**``location``参数用于指定action执行完毕后要转向的目标资源，``parse``属性决定了location是否可以通过使用OGNL来引用参数，默认为true。  
location参数是默认的参数，在所有的Result实现类中，都定义了一个字符串类型的``DEFAULT_PARAM``静态常量，专门用于指定默认的参数名。 ``DEFAULT_PARAM``常量的定义：``public static final String DEFAULT_PARAM=“location”``;  
  **B、**在设置location参数时，可以在参数值中使用OGNL表达式。  
{% highlight xml %}
<action name=“viewNews” class=“com.ibm.ViewNewsAction”
<result name=“success”  type=“dispatcher”>
    <param name=“location” >/viewNews.jsp?id=${id}</param>
     <param name=“parse” >true</param>
</result>
</action>   
{% endhighlight %}
考虑到默认值的使用（``dispatcher``和``location``都是默认值），上述可以简化为：
{% highlight xml %}
<action name=“viewNews” class=“com.ibm.ViewNewsAction”>
<result name=“success” >viewNews.jsp?id=${id}</result>
</action>
{% endhighlight %}

**2、redirect结果类型（重定向到一个Url，也可以是Action或一个页面）**  
   ``redirect``结果类型在后台使用``HttpServletResponse``的``sendRedirect``方法将请求重定向到指定的URL，它的实现类是``org.apache.struts2.dispatcher.ServletRedirectResult``.该类同样有二个**属性**(property):``location``和``parse``。  
在使用redirect结果类型的场景中，用户要完成一次与服务器之间的交互，浏览器需要完成两次请求，**因此第一次请求中的数据在第二次请求中是不可用的**，这意味在目标资源中是不能访问action实例、action错误以及错误等。如果有某些数据需要在目标资源中访问，  
  i、一种方式是将数据保存到Session中，  
  ii、另一种方式是通过请求参数来传递数据。  
例如：
{% highlight xml %}
<result name="success" type="redirect">  
  <param name="location">foo.jsp</param>
  <param name="parse">false</param><!--不解析OGNL-->
</result>
{% endhighlight %}
再比如：
{% highlight xml %}
<package name="passingRequestParameters"extends="struts-default"
      namespace="/passingRequestParameters">  

   <!--  
     The redirect-action url generated will be : 
/genReport/generateReport.jsp?reportType=pie&width=100&height=100  
   -->  

  <action name="gatherReportInfo" class="...">  
    <result name="showReportResult" type="redirect">  
      <param name="location">generateReport.jsp</param>  
      <param name="namespace">/genReport</param>  
      <param name="reportType">pie</param>  
      <param name="width">100</param> 
      <param name="height">100</param> 
     </result>  
  </action>  
</package>
{% endhighlight %}  

**3、redirectAction结果类型（重定向到一个Action）**  
经常用于防止表单重复提交，比方说在增加完用户之后要显示列表 
 redirectAction结果类型的实现类是``org.apache.struts2.dispatcher.ServletActionRedirectResult``,该类是``ServletRedirectResult``的子类，因此我们也就可以判断出redirectAction结果类型和redirect结果类型的后台工作原理是一样的，即都是利用``HttpServletResponse``的``sendRedirect``方法将请求重定向到指定的URL。例如： 
 
*1）重定向到不同命名空间下*：
{% highlight xml linenos%}
<package name="public"extends="struts-default">  
  <action name="login"class="...">  
      <result type="redirectAction">  
        <param name="actionName">dashboard</param> 
        <param name="namespace">/secure</param> 
      </result> 
   </action> 
</package> 
{% endhighlight %} 
*说明：*这里第4行与下文**2）**中xml代码中第3行对应  

*2）重定向到同一命名空间下的action*：
{% highlight xml linenos%}
<package name="secure"extends="struts-default" namespace="/secure">
    <action name="dashboard" class="...">  
      <result>dashboard.jsp</result> 
      <result name="error"type="redirectAction">error</result> 
    </action> 
    <action name="error" class="...">  
      <result>error.jsp</result>
    </action> 
</package> 
{% endhighlight %}
*说明：*这里第4行与第6行对应  

*3）重定向到Action并且传参*  
{% highlight xml %}
<package name="passingRequestParameters"extends="struts-default"namespace="/passingRequestParameters">  
  <action name="gatherReportInfo" class="...">  
    <result name="showReportResult" type="redirectAction">  
      <param name="actionName">generateReport</param> 
      <param name="namespace">/genReport</param> 
      <param name="reportType">pie</param> 
      <param name="width">100</param>
      <param name="height">100</param>
      <param name="empty"></param>
      <param name="supressEmptyParameters">true</param> 
    </result> 
  </action> 
</package>
{% endhighlight %}  
*说明：*相当于发送了URL请求：  
``/genReport/generateReport.action?reportType=pie&width=100&height=100``  

**4、链接类型chain**  
chain是一种特殊的视图结果，用来将Action执行完之后链接到另一个Action中继续执行，新的Action使用上一个Action的上下文（ActionContext），数据也会被传递，chain结果类型有4个属性，分别是：  

- ``actionName``:链接到的另一个action的名字  
- ``namespace`` :用于明确被链接到的action的命名空间，缺省为当前明明空间  
- ``method``:缺省为``execute()方法``  
- ``skipActions``:指定一个使用 , 连接的Action的name组成的集合，一般不建议使用这种类型的结果  

例如：  

*链接到同一命名空间下的Action*
{% highlight xml %}
<package name="public"extends="struts-default">  
    <action name="createAccount" class="...">  
        <result type="chain">login</result>
   </action> 
    <action name="login" class="...">  
        <!--Chain to another namespace --> 
        <result type="chain">  
           <param name="actionName">dashboard</param> 
           <param name="namespace">/secure</param> 
       </result> 
   </action> 
</package>  
{% endhighlight xml %} 
{% highlight xml %}
<package name="secure" extends="struts-default"namespace="/secure">  
    <actionname="dashboard" class="...">  
       <result>dashboard.jsp</result> 
   </action> 
</package>
{% endhighlight xml %} 


