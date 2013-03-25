---
layout: post
title: "CASTGC Cookie and HttpOnly Flag"
date: 2013-03-25 10:56
comments: true
categories: [java, cas]
---

Let's assume you employ [CAS](http://www.jasig.org/cas) to be your single sign-on solution. Upon receiving correct credentials from the user CAS generates CASTGC cookie,
   which is marked with [Secured](http://en.wikipedia.org/wiki/HTTP_cookie#Secure_and_HttpOnly) flag. And assume you want to add [HttpOnly](http://en.wikipedia.org/wiki/HTTP_cookie#Secure_and_HttpOnly)
   flag to the cookie, to prevent accessing it `via non-HTTP methods`.
   CAS of version 3.5.0 depends on [servlet-api 2.5](http://download.oracle.com/otndocs/jcp/servlet-2.5-mrel-eval-oth-JSpec/) and fancy [setHttpOnly](http://docs.oracle.com/javaee/6/api/javax/servlet/http/Cookie.html#setHttpOnly\(boolean\))
   method has been added in version 3.0 of the servlet-api. So how do we deal with it?

Assuming you run your CAS in a servlet container based on [servlet 3.0 specification](http://download.oracle.com/otndocs/jcp/servlet-3.0-fr-oth-JSpec/)
 (e.g. [Jetty 8](http://eclipse.org/jetty/) or [Tomcat 7](http://tomcat.apache.org/download-70.cgi)) and use [maven](http://maven.apache.org/) as your build tool you can do the following to make it happen:

<ol>
<li> exclude old (2.5 and lower) servlet-api jars from dependencies. For example

```xml
    <dependency>
        <groupId>org.jasig.cas</groupId>
        <artifactId>cas-server-integration-restlet</artifactId>
        <version>${casVersion}</version>
        <exclusions>
            <exclusion>
                <groupId>javax.servlet</groupId>
                <artifactId>servlet-api</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
```
</li>
<li> include new servlet-api jar (notice that artifact id has been changed)

``` xml
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.0.1</version>
        <scope>provided</scope>
    </dependency>
```
</li>
<li> create your own version of <a href="https://github.com/Jasig/cas/blob/master/cas-server-webapp/src/main/java/org/jasig/cas/web/support/CookieRetrievingCookieGenerator.java">CookieRetrievingCookieGenerator</a>
which supports HttpOnly flag. You want to extend this class because CAS relies on it internally.

``` java
    package com.daodecode.cas.web.support;

    import javax.servlet.http.Cookie;

    public class HttpOnlySupportingCookieGenerator
            extends org.jasig.cas.web.support.CookieRetrievingCookieGenerator {
        private boolean cookieHttpOnly = false;

        public void setCookieHttpOnly(boolean cookieHttpOnly) {
            this.cookieHttpOnly = cookieHttpOnly;
        }

        @Override
        protected Cookie createCookie(String cookieValue) {
            Cookie cookie = super.createCookie(cookieValue);
            cookie.setHttpOnly(cookieHttpOnly);
            return cookie;
        }
    }
```
</li>
<li> override TGT Cookie Generator creation providing your version of <code>ticketGrantingTicketCookieGenerator.xml</code>

``` xml
    <bean id="ticketGrantingTicketCookieGenerator"
          class="com.daodecode.cas.web.support.HttpOnlySupportingCookieGenerator"
	    cookieSecure="${cas.cookie.secure}"
        cookieHttpOnly="${cas.cookie.httponly}"
        cookieMaxAge="-1"
		cookieName="CASTGC"
		cookiePath="${cas.cookie.path}" />
```
</li>
</ol>
done!