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

------------


# 2 ValueStack
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
