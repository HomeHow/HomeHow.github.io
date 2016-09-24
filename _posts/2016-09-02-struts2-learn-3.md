---
layout: post
title: struts2学习笔记（三）
subtitle: 访问web资源
date: 2016-09-02T00:00:00.000Z
author: HomeHow
header-img: img/post-bg-js-module.jpg
catalog: true
tags:
  - struts2学习
---

# 访问web资源 #
在 *Action* 中, 可以通过以下方式访问 web 的 `HttpSession`, `HttpServletRequest`, `HttpServletResponse`  等资源。

1. 与 Servlet API 解耦的访问方式。  
- 通过`com.opensymphony.xwork2.ActionContext`
- 通过 Action 实现如下接口
  `org.apache.struts2.interceptor.ApplicationAware;`
  `org.apache.struts2.interceptor.RequestAware;`
  `org.apache.struts2.interceptor.SessionAware;`
2. 与 Servlet API 耦合的访问方式
