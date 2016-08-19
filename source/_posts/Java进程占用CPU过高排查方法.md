---
title: Java进程占用CPU过高排查方法
date: 2016-07-27 10:06:05
categories:
- Development
tags: 
- JAVA
- CPU过高
---
## 1.利用TOP命令查看，发现Java进程持续占高CPU

 	PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                  
 	4233 root      20   0 6667m 2.3g 8312 S 264.1 30.4  79979:06 java
 	
## 2.利用TOP -H找到对应进程下面线程的耗CPU情况
	# top -H -p 4233
	PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                  
 	7041 root      20   0 6667m 2.3g 8312 S 20.1 30.4   5524:52 java                                     
 	7048 root      20   0 6667m 2.3g 8312 S 18.8 30.4   5098:27 java                                     
 	9625 root      20   0 6667m 2.3g 8312 S 18.5 30.4   7205:30 java                                     
	10111 root      20   0 6667m 2.3g 8312 S 17.8 30.4   6008:08 java                                     
	10254 root      20   0 6667m 2.3g 8312 S 17.8 30.4   6001:12 java                                     
	10923 root      20   0 6667m 2.3g 8312 S 17.5 30.4   5999:17 java                                     
	10033 root      20   0 6667m 2.3g 8312 S 17.1 30.4   6004:10 java                                     
 	4353 root      20   0 6667m 2.3g 8312 S 16.8 30.4   7209:04 java                                     
 	9685 root      20   0 6667m 2.3g 8312 S 16.8 30.4   6002:45 java                                     
 	7042 root      20   0 6667m 2.3g 8312 S 16.5 30.4   7193:56 java                                     
 	9910 root      20   0 6667m 2.3g 8312 S 16.5 30.4   6010:04 java                                     
	10893 root      20   0 6667m 2.3g 8312 S 16.5 30.4   7195:46 java                                     
 	4235 root      20   0 6667m 2.3g 8312 S 15.5 30.4 815:26.39 java                                      	4238 root      20   0 6667m 2.3g 8312 R 12.9 30.4 815:54.11 java                                     
 	4236 root      20   0 6667m 2.3g 8312 S 12.5 30.4 811:10.16 java                                     
 	4237 root      20   0 6667m 2.3g 8312 S  9.6 30.4 814:50.57 java                                     
 	4239 root      20   0 6667m 2.3g 8312 S  2.0 30.4 752:01.99 java                                      	4294 root      20   0 6667m 2.3g 8312 S  0.7 30.4  56:16.83 java                                     
 	4280 root      20   0 6667m 2.3g 8312 S  0.3 30.4  68:25.58 java                                     
 	4233 root      20   0 6667m 2.3g 8312 S  0.0 30.4   0:00.00 java                                     
 	4234 root      20   0 6667m 2.3g 8312 S  0.0 30.4   0:07.16 java                                     
 	4240 root      20   0 6667m 2.3g 8312 S  0.0 30.4   0:10.14 java                                     
 	4241 root      20   0 6667m 2.3g 8312 S  0.0 30.4   0:08.93 java                                     
 	4242 root      20   0 6667m 2.3g 8312 S  0.0 30.4   0:00.00 java                                     
 	4243 root      20   0 6667m 2.3g 8312 S  0.0 30.4   0:52.14 java                                     
 	4244 root      20   0 6667m 2.3g 8312 S  0.0 30.4   0:47.82 java                                      	4245 root      20   0 6667m 2.3g 8312 S  0.0 30.4   0:00.00 java                                     
 	4246 root      20   0 6667m 2.3g 8312 S  0.0 30.4  19:30.64 java                                     
 	4249 root      20   0 6667m 2.3g 8312 S  0.0 30.4   0:03.20 java 

<!-- more --> 	
## 3.将需要查看的线程ID转换为16进制格式
	# printf "%x\n" 7041
	1b81
	
## 4.打印线程的堆栈信息
	# jstack 4233|grep 1b81 -A 30 
	"http-80-61" daemon prio=10 tid=0x00007f75e8067000 nid=0x1b81 runnable [0x00007f75733f1000]
   		java.lang.Thread.State: RUNNABLE
        	at com.saas.biz.commen.FilterChar.ignoreCaseIndexOf(FilterChar.java:94)
        	at com.saas.biz.commen.FilterChar.repex(FilterChar.java:41)
        	at com.saas.sys.dbm.Dbexecute.setQueryParam(Dbexecute.java:627)
        	at com.saas.sys.dbm.Dbexecute.selBizQuery(Dbexecute.java:426)
        	at com.saas.biz.dao.areaDAO.AreaExt.selByList(AreaExt.java:25)
        	at com.saas.biz.AreaInfoMgr.AreaInfo.getAllClassListForAdd(AreaInfo.java:1493)
        	at org.apache.jsp.car.supplier_jsp._jspService(supplier_jsp.java:116)
        	at org.apache.jasper.runtime.HttpJspBase.service(HttpJspBase.java:70)
        	at javax.servlet.http.HttpServlet.service(HttpServlet.java:723)
        	at org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:388)
        	at org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:313)
        	at org.apache.jasper.servlet.JspServlet.service(JspServlet.java:260)
        	at javax.servlet.http.HttpServlet.service(HttpServlet.java:723)
        	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:290)
        	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:206)
        	at org.apache.catalina.core.ApplicationDispatcher.invoke(ApplicationDispatcher.java:646)
        	at org.apache.catalina.core.ApplicationDispatcher.processRequest(ApplicationDispatcher.java:436)
        	at org.apache.catalina.core.ApplicationDispatcher.doForward(ApplicationDispatcher.java:374)
        	at org.apache.catalina.core.ApplicationDispatcher.forward(ApplicationDispatcher.java:302)
        	at org.tuckey.web.filters.urlrewrite.NormalRewrittenUrl.doRewrite(NormalRewrittenUrl.java:195)
        	at org.tuckey.web.filters.urlrewrite.RuleChain.handleRewrite(RuleChain.java:159)
        	at org.tuckey.web.filters.urlrewrite.RuleChain.doRules(RuleChain.java:141)
        	at org.tuckey.web.filters.urlrewrite.UrlRewriter.processRequest(UrlRewriter.java:90)
        	at org.tuckey.web.filters.urlrewrite.UrlRewriteFilter.doFilter(UrlRewriteFilter.java:417)
        	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:235)
        	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:206)
        	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:233)
        	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:191)
        	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:470)
        	
## 5.在堆栈信息里面找到你熟悉的字眼，去排查相关问题。