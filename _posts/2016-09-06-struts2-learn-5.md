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

# 2 %、#、$这三个符号的使用

## 2.1 符号 *#* 的使用
`#`是OGNL查找符，当目标对象在根部或顶部，也就是根部的第一层树节点时，可直接使用对象名，并且必须只写对象名，不可使用#以作为区分。  
struts2 OGNL的查找范围为OGNL context和ActionContext，其包含下面是顶节点  

|  名称 |   作用 |    例子 |
| ------------ | ------------ | ------------ |
|parameters   | 包含当前HTTP请求参数的Map    | `#parameters.id[0]`作用相当于`request.getParameter("id")`   |
|  request  |  包含当前HttpServletRequest的属性（attribute)的Map |  `#request.userName`相当于`request.getAttribute("userName")`   |
| session  | 包含当前HttpSession的属性（attribute）的Map   | `#session.userName`相当于`session.getAttribute("userName")`   |
|  application  | 包含当前应用的ServletContext的属性（attribute）的Map  |  `#application.userName`相当于`application.getAttribute("userName")`   |  

**注**：attr 用于按*request* > *session* > *application*顺序访问其属性（attribute），`#attr.userName`相当于按顺序在以上三个范围（scope）内读取userName属性，直到找到为止。  
另外#在集合中可以作为筛选元素的条件，即用于过滤和投影（projecting)集合，如  
{% highlight java %}
books.{?#this.price<100}//其中books是Action类的集合元素
{% endhighlight %}  
还可以，构造Map，如: `#{'foo1':'bar1', 'foo2':'bar2'}`  
`#{'foo1':'bar1', 'foo2':'bar2'}`这种方式常用在给*radio*或*select*、*checkbox*等标签赋值上。如果要在页面中取一个map的值可以这样写：  

{% highlight java %}
<s:property value="#myMap['foo1']"></s:property>
<s:property value="#myMap['foo1']"></s:property>
{% endhighlight %}

## 2.2 符号 *%* 的使用

`%`符号的用途是在标签的属性被理解为字符串类型时，告诉执行环境`%{}`里的是OGNL表达式。  
例如：
{% highlight java %}
<s:set name="myMap" value="#{'key1':'value1','key2':'value2'}"/>
<s:property value="#myMap['key1']"/>
<s:url value="#myMap['key1']">   //输出：#myMap['key1']
<s:url value="%{#myMap['key1']}"//输出：value1
{% endhighlight %}

## 2.3 符号 *$* 的使用
`$ {}`实际上传统EL的写法。


# 3  OGNL 表达式读取值栈中的属性值

##  3.1 获取值栈中 ContextMap 中的数据
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

##  3.2 获取值栈对象栈中的数据
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

##  3.3 访问数组类型的属性
有些属性将返回一个对象数组而不是单个对象, 可以像读取任何其他对象属性那样读取它们。这种数组型属性的各个元素以逗号分隔, 并且不带方括号。可以使用下标访问数组中指定的元素: `colors[0]`。可以通过调用其 length 字段查出给定数组中有多少个元素: `colors.length`。  
示例：
{% highlight html %}
<%
	String[] names = new String[]{"a", "b", "c", "d"};
	request.setAttribute("names", names);
%>
length: <s:property value="#attr.names.length"></s:property>
<br><br>
names1:<s:property value="#attr.names[1]"></s:property>
<br><br>
names2:<s:property value="#attr.names[2]"></s:property>
<br><br>
{% endhighlight%}

##  3.4 访问 Map类型的属性
<p id="lable1"/>
读取一个 Map 类型的属性将以如下所示的格式返回它所有的键值对（[留意差别](#lable2)）：  
`{key-1=value-1, key-2=value-3, ... , key-n=value-n}`  
若希望检索出某个 Map 的值, 需要使用如下格式:`map[key]`   
可以使用 `size` 或 `size()` 得出某个给定的 Map 的键值对的个数.  
可以使用 `isEmpty` 或 `isEmpty()` 检查某给定 Map 是不是空。   
<p id="lable2"/>
可以使用如下语法来创建一个 Map([留意差别](#lable1)):  
`#{ key-1=value-1, key-2=value-3, ... , key-n=value-n }`  
示例：
{% highlight html %}
<%
	Map<String, String> letters = new HashMap<String, String>();
	request.setAttribute("letters", letters);
	letters.put("aa","a");
	letters.put("bb","b");
	letters.put("cc","c");
%>
size: <s:property value="#attr.letters.size"></s:property>
<br><br>
aa:<s:property value="#attr.letters['aa']"></s:property>
<br><br>
bb:<s:property value="#attr.letters['bb']"></s:property>
<br><br>
{% endhighlight%}

##  3.5 访问 List 类型的属性
有些属性将返回的类型是 *java.util.List*, 可以像读取任何其他属性那样读取它们. 这种 List 的各个元素是字符串, 以逗号分隔, 并且带方括号。  
可以使用下标访问 List 中指定的元素: `colors[0]`  
可以通过调用其 size 方法或专用关键字 size 的方法查出给定List 的长度: `colors.size` 或 `colors.size()`  
可以通过使用 `isEmpty()` 方法或专用关键字 `isEmpty` 来得知给定的 List 是不是空.：`colors.isEmpty` 或 `colors.isEmpty()`  
还可以使用 OGNL 表达式来创建 List, 创建一个 List 与声明一个 Java 数组是相同的: `{“Red”, “Black”, “Green”}`  

##  3.6 调用字段和方法
可以利用 OGNL 调用  

- 任何一个 Java 类里的静态字段.  
- 被压入到 ValueStack 栈的对象上的公共字段和方法.  

默认情况下, Struts2 不允许调用任意 Java 类静态方法, 需要重新设置 *struts.ognl.allowStaticMethodAccess* 标记变量的值为 true。([Struts2.3.20不支持OGNL静态方法调用allowStaticMethodAccess](http://blog.csdn.net/shijiebei2009/article/details/42581815 "Struts2.3.20不支持OGNL静态方法调用allowStaticMethodAccess"))  
调用静态字段需要使用如下所示的语法:  
`@fullyQualifiedClassName@fieldName: @java.util.Calendar@DECEMBER`  
`@java.util.Calendar@DECEMBER`  
调用一个实例字段或方法的语法, 其中 object 是 Object Stack 栈里的某个对象的引用:  
`.object.fieldName: [0].datePattern`  
`.object.methodName(argumentList): [0].repeat(3, “Hello”)`  

# 4 使用 EL 访问值栈中对象的属性 
`<s:property value=“fieldName”>` 也可以通过 JSP EL 来达到目的: `${fieldName}`  
**原理**： Struts2 将包装 `HttpServletRequest` 对象后的 `org.apache.struts2.dispatcher.StrutsRequestWrapper` 对象传到页面上, 而这个类重写了 `getAttribute()` 方法。  

# 5 声明式异常处理 
*exception-mapping *元素: 配置当前 action 的声明式异常处理  
*exception-mapping* 元素中有 2 个属性：

- **exception**: 指定需要捕获的的异常类型。异常的全类名  
- **result**: 指定一个响应结果, 该结果将在捕获到指定异常时被执行, 既可以来自当前 action 的声明, 也可以来自 global-results 声明.  

可以通过 `global-exception-mappings` 元素为应用程序提供一个全局性的异常捕获映射. 但在 `global-exception-mappings` 元素下声明的任何 `exception-mapping` 元素只能引用在 `global-results` 元素下声明的某个 result 元素  
声明式异常处理机制由  `ExceptionMappingInterceptor` 拦截器负责处理, 当某个 `exception-mapping` 元素声明的异常被捕获到时, `ExceptionMappingInterceptor` 拦截器就会向 `ValueStack` 中添加两个对象:  

- exception: 表示被捕获异常的 Exception 对象  
- exceptionStack: 包含着被捕获异常的栈  

可以在视图上通过 `<s:property>` 标签显示异常消息  
示例：
{% highlight html %}
<action name="product-save" class="com.hhl.struts2.valueStack.Product" method="save">
	<exception-mapping exception="java.lang.ArithmeticException" result="input"></exception-mapping>
	<result name="input">/input.jsp</result>
	<result>/details.jsp</result>
</action>
{% endhighlight%}