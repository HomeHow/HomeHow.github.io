---
author: HomeHow
catalog: true
date: 2016-09-06T00:00:00.000Z
header-img: img/post-bg-js-module.jpg
layout: post
subtitle: OGNL与声明式异常处理
tags:
  - struts2学习
title: struts2学习笔记（五）
---
# 1 OGNL #
OGNL，全称为*Object-Graph Navigation Language*，它是一个功能强大的表达式语言，用来获取和设置Java对象的属性，它旨在提供一个更高的更抽象的层次来对Java对象图进行导航。OGNL表达式的基本单位是"导航链"，一般导航链由如下几个部分组成：  

- 属性名称（property）  
- 方法调用（method invoke）  
- 数组元素  

所有的OGNL表达式都基于当前对象的上下文来完成求值运算，链的前面部分的结果将作为后面求值的上下文。例如：`names[0].length()`。OGNL是通常要结合Struts 2的标志一起使用。主要是`#`、`%`和`$`这三个符号的使用  
在 JSP 页面上可以可以利用 OGNL访问到值栈(ValueStack) 里的对象属性.

------------


# 2  OGNL 表达式读取值栈中的属性值

##  2.1 获取值栈中 ContextMap 中的数据
若希望访问值栈中 ContextMap 中的数据（例如request, session, application等）, 需要给 OGNL 表达式加上一个前缀字符 #. 如果没有前缀字符 #, 搜索将在 ObjectStack 里进行。可以使用如下形式：  

- `#object.propertyName`  
- `#object['propertyName']`  
- `#object["propertyName"]`  

示例，在jsp文件中，利用s:property 标签和 OGNL 读取：  
session 中的 code 属性:  
{% highlight html %}
<s:property value="#session.code"></s:property>
{% endhighlight%}
request 中的 customer 属性的 name 属性值:  
{% highlight html %}
<s:property value="#request.customer.name"></s:property>
{% endhighlight%}
attribute中(按 request, session, application 的顺序)的 lastAccessDate 属性:  
{% highlight html %}
<s:property value="#attr.lastAccessDate"></s:property>
{% endhighlight%}

##  2.2 获取值栈对象栈中的数据
若希望访问值栈对象栈中的数据，可以使用如下形式：  

- `object.propertyName`  
- `object['propertyName']`  
- `object["propertyName"]`  

ObjectStack 里的对象可以通过一个从零开始的下标来引用。ObjectStack 里的栈顶对象可以用 [0] 来引用, 它下面的那个对象可以用 [1] 引用。  
示例，在jsp文件中，利用s:property 标签和 OGNL 读取：  
栈顶对象的 message 属性值:  
{% highlight html %}
<s:property value="[0].message"></s:property>
<br><br>
<s:property value="[0]['message']"></s:property>
<br><br>
{% endhighlight%}
**注：**若在指定的对象里没有找到指定的属性, 则到指定对象的下一个对象里继续搜索. 即 [n] 的含义是从第 n 个开始搜索, 而不是只搜索第 n 个对象。若从栈顶对象开始搜索, 则可以省略下标部分。也就是说如下上述代码也可以写成：  
{% highlight html %}
<s:property value="message"></s:property>
<br><br>
{% endhighlight%}

request 中的 customer 属性的 name 属性值:  
{% highlight html %}
<s:property value="#request.customer.name"></s:property>
{% endhighlight%}
attribute中(按 request, session, application 的顺序)的 lastAccessDate 属性:  
{% highlight html %}
<s:property value="#attr.lastAccessDate"></s:property>
{% endhighlight%}


{% highlight html %}
{% endhighlight%}

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


------------


# 3 ActionContext
`ActionContext`是Action运行的上下文，每个ActionContext是一个基本的容器，包含着Action运行需要的数据，比如请求参数、会话等。ActionContext是线程安全的，每个线程有一个独立的ActionContext。  
`com.opensymphony.xwork2.ognl.OgnlValueStack`部分源码中的如下：
{% highlight java %}
static ThreadLocal<ActionContext> actionContext = new ThreadLocal();
    public static final String ACTION_NAME = "com.opensymphony.xwork2.ActionContext.name";
    public static final String VALUE_STACK = "com.opensymphony.xwork2.util.ValueStack.ValueStack";
    public static final String SESSION = "com.opensymphony.xwork2.ActionContext.session";
    public static final String APPLICATION = "com.opensymphony.xwork2.ActionContext.application";
    public static final String PARAMETERS = "com.opensymphony.xwork2.ActionContext.parameters";
    public static final String LOCALE = "com.opensymphony.xwork2.ActionContext.locale";
    public static final String TYPE_CONVERTER = "com.opensymphony.xwork2.ActionContext.typeConverter";
    public static final String ACTION_INVOCATION = "com.opensymphony.xwork2.ActionContext.actionInvocation";
    public static final String CONVERSION_ERRORS = "com.opensymphony.xwork2.ActionContext.conversionErrors";
    public static final String CONTAINER = "com.opensymphony.xwork2.ActionContext.container";
    private Map<String, Object> context;

    public ActionContext(Map<String, Object> context) {
        this.context = context;
    }
	
	.......
    
    public void setValueStack(ValueStack stack) {
        this.put("com.opensymphony.xwork2.util.ValueStack.ValueStack", stack);
    }

    public ValueStack getValueStack() {
        return (ValueStack)this.get("com.opensymphony.xwork2.util.ValueStack.ValueStack");
    }
	
	.......
	
}
{% endhighlight %}

ActionContext里面存放有很多的值，典型如：  

- *Request的parameters*：请求中的参数，要注意这里的数据是从请求对象里面拷贝出来的，因此这里数据的变化是不会影响到请求对象里面的参数的值的  
- *Request的Attribute*：请求中的属性，这里其实就是个Map，存放着请求对象的属性数据，这些数据和请求对象的Attribute是连动的  
- *Session的Attribute*：会话中的属性，这里其实就是个Map，存放着会话对象的属性数据，这些数据和会话对象的Attribute是连动的  
- *Application的Attribute*：应用中的属性，这里其实就是个Map，存放着应用对象的属性数据，这些数据和应用对象的Attribute是连动的  
- *Value stack*：也就是狭义值栈，ActionContext以value stack作为被OGNL访问的根，简单点说，OGNL在没有特别指明的情况下，访问的就是value stack里面的数据  
- *attr*：在所有的属性范围中获取值，依次搜索page、request、session和application。  

可以看到，在`ActionContext`里面其实是包含着`ValueStack`对象，正是因为这个原因，再加上ActionContext还包含其他的数据，因此把ActionContext称为广义值栈。  


------------

# 4 值栈的使用

## 4.1 ActionContext的使用
要获取*ActionContext*有两个基本的方法，如果在不能获取到*ActionInvocation*的地方，可以直接使用*ActionContext*一个静态的`getContext`方法，就可以访问到当前的*ActionContext*了，示例如下：  
{% highlight java %}
ActionContext ctx = ActionContext.getContext();  
{% endhighlight %}
如果在能获取到*ActionInvocation*的地方，比如在拦截器里面、自定义的*Result*里面等，可以通过*ActionInvocation*来获取到*ActionContext*，示例如下：  
{% highlight java %}
ActionContext ctx = actionInvocation.getInvocationContext();  
{% endhighlight %}
ActionContext主要的功能是用来存放数据的，典型的方法如下：  
- `get(String key)`：根据key从ActionContext当前的存储空间里面获取相应的值  
- `put(String key, Object value)`：把值存储在ActionContext的存储空间里面  
- `Map<String,Object>` getApplication()：返回ServletContext中存储的值  
- `Map<String,Object> getSession()`：返回HttpSession中存储的值  
- `Map<String,Object> getContextMap()`：返回当前context存储的值  
- `Map<String,Object> getParameters()`：返回HttpServletRequest对象里面存储的，客户端提交的参数  
- `ValueStack getValueStack()`：获取OGNL的值栈  

## 4.2 ValueStack的使用
ValueStack主要是通过OGNL表达式来访问，也就是说，在Struts2里面主要是通过标签来访问的。ValueStack有一个特点，如果访问的值栈里有多个对象，*且相同的属性在多个对象中同时出现，则值栈会按照从栈顶到栈底的顺序，寻找第一个匹配的对象*。  
如果要显示的获取，由于ActionContext中存在着ValueStack，因而可以直接由ActionContext对象的`getValueStack()`方法即可获取。  
ValueStack主要的功能也是用来存放数据的，典型的方法如下：  
- `Object findValue(String expr)`：根据表达式在value stack中，按照缺省的访问顺序去获取表达式对应的值
- `void setValue(String expr, Object value)`：根据表达式，按照缺省的访问顺序，向value stack中设置值
- `Object peek()`：获取value stack中的顶层对象，不修改value stack对象
- `Object pop()`：获取value stack中的顶层对象，并把这个对象从value stack中移走
- `void push(Object o)`：把对象加入到value stack对象中，并设置成为顶层对象

------------

# 5 示例
本例展示了利用OGNL表达式来访问值栈的一个例子：  
{% highlight java %}
public class Product {
    private Integer productId;
    private String productName;
    private String productDesc;

    private double productPrice;

    public Integer getProductId() {
        return productId;
    }

    public void setProductId(Integer productId) {
        this.productId = productId;
    }

    public String getProductName() {
        return productName;
    }

    public void setProductName(String productName) {
        this.productName = productName;
    }

    public String getProductDesc() {
        return productDesc;
    }

    public void setProductDesc(String productDesc) {
        this.productDesc = productDesc;
    }

    public double getProductPrice() {
        return productPrice;
    }

    public void setProductPrice(double productPrice) {
        this.productPrice = productPrice;
    }

    @Override
    public String toString() {
        return "Product [productId=" + productId + ", productName="
                + productName + ", productDesc=" + productDesc
                + ", productPrice=" + productPrice + "]";
    }

    public String save(){
        System.out.println("save: " + this);
        return "details";
    }

    public String test(){
        System.out.println("test");
        return "success";
    }

    public Product() {
        System.out.println("Product's constructor...");
    }
}
{% endhighlight %}
配置struts2.xml文件
{% highlight xml %}
<action name="product-save" class="com.hhl.struts2.helloworld.Product" method="save">
      <result name="details">/WEB-INF/pages/details.jsp</result>
</action>
{% endhighlight %}
input.jsp文件  
{% highlight html %}
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Title</title>
</head>
<body>
<form action="product-save.action" method="post">
    ProductName:<input type="text" name="productName"/>
    <br><br>

    ProductDesc:<input type="text" name="productDesc"/>
    <br><br>

    ProductPrice:<input type="text" name="productPrice"/>
    <br><br>

    <input type="submit" value="Submit"/>
    <br><br>
</form>
</body>
</html>
{% endhighlight %}  
details.jsp文件，利用OGNL表达式来访问值栈  
{% highlight html %}
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="s" uri="/struts-tags" %> 
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <title>Title</title>
    </head>
    <body>

        ProductName: <s:property value="productName"></s:property>
        <br><br>

        ProductDesc: <s:property value="productDesc"></s:property>
        <br><br>

        ProductPice: <s:property value="productPrice"></s:property>
        <br><br>
    </body>
</html>
{% endhighlight %}  
