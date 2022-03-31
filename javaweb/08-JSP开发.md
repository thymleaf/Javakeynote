# JSP开发

我们从前面的章节可以看到，**Servlet** 就是一个能处理HTTP请求，发送HTTP响应的小程序，而发送响应无非就是获取`PrintWriter`，然后输出HTML：

```java
PrintWriter pw = resp.getWriter();
pw.write("<h1>登录</h1>");
pw.write("<form action=\"/test/signin\" method=\"post\">");
pw.write("<p>用户名: <input name=\"username\"></p>");
pw.write("<p>密码: <input name=\"password\" type=\"password\"></p>");
pw.write("<p><button type=\"submit\">登录</button> <a href=\"/\">取消</a></p>");
pw.write("</form>");
pw.flush();
```

只不过，用PrintWriter输出HTML **比较痛苦** ，因为不但要正确编写HTML，还需要插入各种变量。那有没有更简单的输出HTML的办法？

> ***JSP***（Java Server Pages）是一种动态网页开发技术。JSP 文件就是在传统的 HTML 文件中插入 Java 代码和 JSP 标签，它的文件必须放到`/src/main/webapp`下，后缀名为`.jsp`
> 
> JSP 依赖于 Servlet，用户访问 JSP 页面时，JSP 代码会被翻译成 Servlet 代码，最终，以字符串的形式向外输出 HTML 代码。所以，JSP 只是在 Servlet 的基础上做了进一步封装。

## 简单的JSP程序

我们来编写一个`hello.jsp`，内容如下：

```jsp
<html>
<head>
    <title>Hello World - JSP</title>
</head>
<body>
    <%-- JSP Comment --%>
    <h1>Hello World!</h1>
    <p>
    <%
         out.println("Your IP address is ");
    %>
    <span style="color:red">
        <%= request.getRemoteAddr() %>
    </span>
    </p>
</body>
</html>
```

整个**JSP**的内容实际上是一个HTML，但是稍有不同：

- 包含在`<%--`和`--%>`之间的是JSP的注释，它们会被完全忽略；
- 包含在`<%`和`%>`之间的是Java代码，可以编写任意Java代码；
- 如果使用`<%= xxx %>`则可以快捷输出一个变量的值。

JSP页面内置了几个变量，这几个变量可以直接使用：

- out：表示HttpServletResponse的PrintWriter；
- session：表示当前HttpSession对象；
- request：表示HttpServletRequest对象。

## Servlet与JSP异同点

相同点：与 Servlet 一样，JSP 也用于生成动态网页。

不同点如下：

| Servlet                          | JSP                        |
|:--------------------------------:|:--------------------------:|
| Servlet 在 Java 内添加 HTML 代码       | JSP 在 HTML 内添加 Java 代码     |
| Servlet 是一个 Java 程序，支持 HTML 标签   | JSP 是一种 HTML 代码，支持 Java 语句 |
| Servlet 一般用于开发程序的业务层（做一些复杂的逻辑处理） | JSP 一般用于开发程序的表示层（显示数据）     |
| Servlet 由 Java 开发人员创建和维护         | JSP 常用于页面设计，由 Web 开发人员使用   |

JSP本质上就是Servlet，因为JSP在执行前首先被编译成一个Servlet。在Tomcat的临时目录下，可以找到一个`hello_jsp.java`的源文件，这个文件就是Tomcat把JSP自动转换成的Servlet源码：

```java
package org.apache.jsp;
import ...

public final class hello_jsp extends org.apache.jasper.runtime.HttpJspBase
    implements org.apache.jasper.runtime.JspSourceDependent,
               org.apache.jasper.runtime.JspSourceImports {

    ...

    public void _jspService(final javax.servlet.http.HttpServletRequest request, final javax.servlet.http.HttpServletResponse response)
        throws java.io.IOException, javax.servlet.ServletException {
        ...
        out.write("<html>\n");
        out.write("<head>\n");
        out.write("    <title>Hello World - JSP</title>\n");
        out.write("</head>\n");
        out.write("<body>\n");
        ...
    }
    ...
}
```

Web Server会根据路径查找对应的`.jsp`文件，如果找到了，就自动编译成Servlet再执行。在服务器运行过程中，如果修改了JSP的内容，那么服务器会自动重新编译。

## JSP脚本

在 JSP 中，可以使用 JSP 脚本写入 Java 代码。JSP 脚本可以包含任意数量的 Java 语句，变量、方法和表达式。

JSP 脚本语法如下：

```jsp
<% Java语句 %>
```

等效于:

```jsp
<jsp:scriptlet>
    Java语句
</jsp:scriptlet>
```

**示例**

```jsp
<html>
<body>
    <%
        String name = request.getParameter("uname");
        String url= request.getParameter("url");
        out.print("欢迎" + name + "，我们的网址是：" + url);
    %>
</body>
</html>
```

## JSP声明语句

JSP 声明语句用于声明一个或多个变量、方法，以供后面的 Java 代码使用。您必须先对变量和方法进行声明，才能使用它们。

JSP 声明语法如下：

```jsp
<%! 声明语句 %>
```

等效于:

```jsp
<jsp:declaration>
    声明语句
</jsp:declaration>
```

**示例**

```java
<%! int num = 0; %>
<%! Circle a = new Circle(2.0); %>
<%!
    public void show () {
    }
%>
```

**JSP脚本和声明的区别**

JSP 脚本只能声明变量，不能声明方法。JSP 声明语句可以声明变量和方法。

JSP 脚本会把包含的内容转译插入到 Servlet 的 service() 方法中，也就是 `<% %>` 中定义的变量是局部变量。这也是 JSP 脚本不能声明方法的原因，因为 Java 不允许方法中嵌套方法。

JSP 声明会把包含的内容添加到 Servlet 类中（在任何方法之外），也就是 `<%! %>` 中定义的变量是成员变量，方法是成员方法。

## JSP表达式

JSP 表达式可以把变量或者表达式输出到网页上，不需要 ```out.print() ```就能输出数据。

JSP 表达式语法如下：

```jsp
<%= 表达式 %>
```

等效于:

```jsp
<jsp:expression>
    表达式
</jsp:expression>
```

> 可以将 <%=表达式 %> 理解为 <% out.println(表达式) %> 的简写方式。这里需要注意，JSP 表达式不能以分号结尾。

**示例**

```jsp
<html>
<body>
    当前时间: <%= java.util.Calendar.getInstance().getTime() %> 
</body>
</html>
```

## JSP指令

**JSP 指令**（directive）用来告诉 Web 服务器如何处理 JSP 页面的请求和响应。

指令分为以下 3 种类型:

| 指 令                | 说 明                         |
|:------------------:|:---------------------------:|
| <%@ page ... %>    | 定义与页面相关的属性，例如脚本语言、错误页面和缓冲要求 |
| <%@ include ... %> | 引入其它 JSP 文件                 |
| <%@ taglib ... %>  | 声明并导入标签库                    |

### page指令

page 指令用来定义当前页面的相关属性。page 指令可以在 JSP 页面的任意位置编写，通常放在 JSP 页面的顶部。

page 指令的语法如下：

```jsp
<%@ page attribute = "value" %>
```

page 指令常用属性:

| 属 性          | 取 值                                                         | 说 明                                       | 举 例                                                                                 |
| ------------ | ----------------------------------------------------------- | ----------------------------------------- | ----------------------------------------------------------------------------------- |
| buffer       | none、缓冲区大小（默认值为 8kb）                                        | 指定输出流是否有缓冲区                               | <%@ page buffer="16kb" %>                                                           |
| autoFlush    | true（默认值）、false                                             | 指定缓冲区是否自动清除                               | <%@ page autoFlush="true" %>                                                        |
| contentType  | text/html; charset = ISO-8859-1、 text/xml；charset = UTF-8 等 | 指定 MIME 类型和字符编码                           | <%@ page contentType="text/html;charset=UTF-8" %>                                   |
| errorpage    | 页面路径                                                        | 指定当前 JSP 页面发生异常时，需要重定向的错误页面               | <%@ page errorpage="myerrorpage.jsp" %>  注意：myerrorpage.jsp 的 isErrorpage 值必须为 true |
| isErrorpage  | true、false（默认值）                                             | 指定当前页面为错误页面                               | <%@ page isErrorpage="true" %>                                                      |
| extends      | 包名.类名                                                       | 指定当前页面继承的父类，一般很少使用                        | <%@ page extends="mypackage.SampleClass"%>                                          |
| import       | 类名、接口名、包名                                                   | 导入类、接口、包，类似于 Java 的 import 关键字            | <％@ page import = " java.util.Date" ％> <%@ page import="java.io.*, java.lang.*"%>   |
| info         | 页面的描述信息                                                     | 定义 JSP 页面的描述信息，可以使用 getServletInfo() 方法获取 | <%@ page info="这里是编程帮的页面信息"%>                                                       |
| isThreadSafe | true（默认值）、false                                             | 是否允许多线程使用                                 | <%@ page isThreadSafe="false" %>                                                    |
| language     | 脚本语言                                                        | 指定页面中使用的脚本语言                              | <%@ page language= "java" %>                                                        |
| session      | true（默认值）、false                                             | 指定页面是否使用 session                          | <%@ page session="false" %>                                                         |
| isELIgnored  | true（默认值）、false                                             | 指定页面是否忽略 JSP 中的 EL                        | <%@ page isELIgnored="false" %>                                                     |

**示例**

下面通过 **page 指令**的 **import** 属性导入``` java.util.Date``` 类，显示欢迎信息和用户登录的日期时间。```login.jsp``` 代码如下：

```jsp
<%@ page import="java.util.Date" language="java"
    contentType="text/html;charset=utf-8"%>
<html>
<body>
    <br /> 您登录的时间是<%=new Date()%>
</body>
</html>
```

### include指令

**include 指令** 用于在 JSP 页面引入其它内容，可以是 JSP 文件、html 文件和文本文件等，相当于把文件的内容复制到 JSP 页面。引入的文件和 JSP 页面同时编译运行。

使用 include 指令有以下优点：

- 增加代码的可重用性
- 使 JSP 页面的代码结构清晰易懂
- 维护简单

include 的语法如下：

```jsp
<%@ include file="URL" %>  
```

**示例**

在 ```index.jsp``` 页面使用 include 指令引入 ```head.jsp```。```head.jsp``` 代码如下：

```jsp
<p>header内容</p>
```

```index.jsp``` 代码如下：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<body>
    <%@ include file="head.jsp" %>
</body>
</html>
```

### taglib指令

在 JSP 中，我们可以使用 **taglib 指令**声明并引入标签库。

> JSP Tag需要正确引入taglib的jar包，并且还需要正确声明，使用起来非常复杂，对于页面开发来说，**不推荐**使用 JSP Tag，因为我们后续会介绍更简单的模板引擎，这里我们不再介绍如何使用taglib。

## JSP动作

JSP 动作使用 XML 语法格式的标签来控制服务器的行为。利用 JSP 动作可以动态地插入文件、重用 JavaBean 组件、把用户重定向到另一个页面、为 Java 插件生成 HTML 代码等。

JSP 动作与 JSP 指令的不同之处如下：

- JSP 指令在翻译阶段执行，从而设置整个 JSP 页面的属性。JSP 页面被执行时首先进入翻译阶段，程序会先查找页面中的 JSP 指令，并将它们转换成 Servlet。所以，JSP 指令是在页面转换时期被编译执行的，且编译一次。
- JSP 动作在请求处理阶段执行，它们只有执行时才实现自己的功能。通常用户每请求一次，动作标识就会执行一次。

### include动作

```<jsp:include>``` 动作用来在页面中引入文件，文件可以是 HTML、JSP 页面和文本文件等。通过 include 动作，我们可以多次使用同一个页面，增加了代码可重用性。例如，在页面中使用 include 动作引入头部和底部页面。

```<jsp:include>``` 的语法如下：

```jsp
<jsp:include page="relativeURL | <%=expression%>" flush="true" />  
```

page 指定引入页面的路径，flush 表示在引入文件前是否刷新缓冲区，默认为 false。

**示例**

```head.jsp``` 代码如下：

```jsp
<p>header内容</p>
```

```index.jsp``` 代码如下：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<body>
    <jsp:include page="head.jsp"></jsp:include>
</body>
</html>
```

### forward动作

```<jsp:forward>``` 动作用来将请求转发到另一个页面中，请求的参数数据会被一起转发到目标页面。

```<jsp:forward>``` 的语法如下：

```jsp
<jsp:forward page="url"/>
```

其中，page 指定需要转发文件的相对路径，且指定的文件只能是该 Web 应用中的文件。

**示例**

在``` login.jsp ```页面中使用``` <jsp:forward>``` 跳转到 ```index.jsp``` 页面，```login.jsp``` 页面代码如下：

```jsp
<%@ page language="java" contentType="text/html;charset=utf-8"%>
<html>
<head>
<title>jsp:forward</title>
</head>
<body>
    <jsp:forward page="index.jsp" />
</body>
</html>
```

```index.jsp``` 页面代码如下：

```jsp
<%@ page import="java.util.Date" language="java"
    contentType="text/html;charset=utf-8"%>
<html>
<head>
<title>编程帮-jsp:forward</title>
</head>
<body>
    <br /> 您登录的时间是<%=new Date()%>
</body>
</html>
```

### param动作

```<jsp:param>``` 动作用来传递参数信息，经常和其它动作一起使用，例如``` <jsp:include>``` 和 ```<jsp:forward>```。

```<jsp:param>``` 的语法如下：

```jsp
<jsp: param name="param_name" value="param_value" />
```

name 指定参数名称，value 指定参数值。

**示例**

在 ```login.jsp``` 中传递参数，并转发到 ```index.jsp``` 页面。```login.jsp``` 代码如下：

```jsp
<%@ page language="java" contentType="text/html;charset=utf-8"%>
<html>
<head>
<title>jsp:param</title>
</head>
<body>
    <jsp:forward page="index.jsp">
        <jsp:param name="username" value="hello" />
    </jsp:forward>
</body>
</html>
```

```index.jsp``` 代码如下：

```jsp
<%@ page language="java" contentType="text/html;charset=utf-8"%>
<html>
<head>
<title>jsp:param</title>
</head>
<body>
    您好，欢迎登录<%=request.getParameter("username")%>！
</body>
</html>
```

## JSP内置对象

为了简化页面的开发过程，JSP 提供了一些内置对象。

JSP 内置对象又称为隐式对象，它们由容器实现和管理。在 JSP 页面中，这些内置对象不需要预先声明，也不需要进行实例化，我们可以直接在脚本和表达式中使用。

### 九大内置对象

JSP 中定义了 **9 个内置对象**，它们分别是：```request```、```response```、```session```、```application```、```out```、```pagecontext```、```config```、```page``` 和 ```exception```，这些对象在客户端和服务器端交互的过程中分别完成不同的功能。

| 对 象         | 类型                                     | 说 明                                                                                             |
|:-----------:| -------------------------------------- | ----------------------------------------------------------------------------------------------- |
| request     | javax.servlet.http.HttpServletRequest  | 获取用户请求信息                                                                                        |
| response    | javax.servlet.http.HttpServletResponse | 响应客户端请求，并将处理信息返回到客户端                                                                            |
| out         | javax.servlet.jsp.JspWriter            | 输出内容到 HTML 中                                                                                    |
| session     | javax.servlet.http.HttpSession         | 用来保存用户信息                                                                                        |
| application | javax.servlet.ServletContext           | 所有用户共享信息                                                                                        |
| config      | javax.servlet.ServletConfig            | 这是一个 Servlet 配置对象，用于 Servlet 和页面的初始化参数                                                          |
| pageContext | javax.servlet.jsp.PageContext          | JSP 的页面容器，用于访问 page、request、application 和 session 的属性                                           |
| page        | javax.servlet.jsp.HttpJspPage          | 类似于 Java 类的 this 关键字，表示当前 JSP 页面                                                                |
| exception   | java.lang.Throwable                    | 该对象用于处理 JSP 文件执行时发生的错误和异常；只有在 JSP 页面的 page 指令中指定 isErrorPage 的取值 true 时，才可以在本页面使用 exception 对象。 |

JSP 的内置对象主要有以下特点：

- 由 JSP 规范提供，不用编写者实例化；
- 通过 Web 容器实现和管理；
- 所有 JSP 页面均可使用；
- 只有在脚本元素的表达式或代码段中才能使用。

### 四大域对象

在 JSP 九大内置对象中，包含四个域对象，它们分别是：***pageContext***（page 域对象）、***request***（request 域对象）、***session***（session 域对象）、以及 ***application***（application 域对象）。

JSP 中的 4 个域对象都能通过以下 3 个方法，对属性进行保存、获取和移除操作。

| 方法                                  | 作用         |
|:-----------------------------------:|:----------:|
| setAttribute(String name, Object o) | 将属性保存到域对象中 |
| getAttribute(String name)           | 获取域对象中的属性值 |
| removeAttribute(String name)        | 将属性从域对象中移除 |

JSP 中的 4 个域对象的作用域各不相同，如下表：

| 作用域         | 描述                                          | 作用范围                                                                   |
| ----------- | ------------------------------------------- | ---------------------------------------------------------------------- |
| page        | 如果把属性保存到 pageContext 中，则它的作用域是 page。        | 该作用域中的属性只在当前 JSP 页面有效，跳转页面后失效。                                         |
| request     | 如果把属性保存到 request 中，则它的作用域是 request。         | 该作用域中的属性只在当前请求范围内有效。服务器跳转页面后有效，例如```<jsp:forward>```；客户端跳转页面后无效，例如超链接。 |
| session     | 如果把属性保存到 session 中，则它的作用域是 session。         | 该作用域中的属性只在当前会话范围内有效，网页关闭后失效。                                           |
| application | 如果把属性保存到 application 中，则它的作用域是 application。 | 该作用域中的属性在整个应用范围内有效，服务器重启后失效。                                           |

### request对象

**JSP request** 是 ```javax.servlet.http.HttpServletRequest``` 的实例对象，主要用来获取客户端提交的数据。

**示例**

在 ```index.jsp``` 页面使用 ```getHeaderNames() ```方法获取 HTTP 头信息，并遍历输出参数名称和对应值。

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ page import="java.util.*"%>
<!DOCTYPE html>
<html>
<head>
<title>request示例</title>
</head>
<body>
    <h2>获取HTTP请求头信息</h2>
    <table width="100%" border="1" align="center">
        <tr bgcolor="#949494">
            <th>参数名称</th>
            <th>参数值</th>
        </tr>
        <%
            Enumeration headerNames = request.getHeaderNames();
            while (headerNames.hasMoreElements()) {
                String paramName = (String) headerNames.nextElement();
                out.print("<tr><td>" + paramName + "</td>\n");
                String paramValue = request.getHeader(paramName);
                out.println("<td> " + paramValue + "</td></tr>\n");
            }
        %>
    </table>
</body>
</html>
```

### response对象

**JSP response** 是 ```javax.servlet.http.HttpServletResponse``` 的实例对象。```response``` 对象和 ```request ```对象相对应，主要用于响应客户端请求，将处理信息返回到客户端。

**示例**

下面在 ```login.jsp``` 新建表单，在 ```checkdetails.jsp``` 接收 ```login.jsp``` 提交的用户名和密码，与指定的用户名和密码相比，相同则登录成功，重定向到 ```success.jsp```；反之登录失败，重定向到 ```failed.jsp```。

```login.jsp``` 代码如下：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<title>response示例</title>
</head>
<body>
    <h2>用户登录</h2>
    <form action="checkdetails.jsp">
        用户名: <input type="text" name="username" /> <br> <br> 
        密码: <input type="text" name="pass" /> <br> <br> 
        <input type="submit" value="登录" />
    </form>
</body>
</html>
```

```checkdetails.jsp``` 代码如下：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<title>response示例</title>
</head>
<body>
    <%
        String username = request.getParameter("username");
        String password = request.getParameter("pass");
        if (username.equals("biancheng") && password.equals("bianchengbang")) {
            response.sendRedirect("success.jsp");
        } else {
            response.sendRedirect("failed.jsp");
        }
    %>
</body>
</html>
```

### out对象

JSP out 是 javax.servlet.jsp.JspWriter 的实例对象。out 对象包含了很多 IO 流中的方法和特性，最常用的就是输出内容到 HTML 中。

**示例**

out 对象的方法相对比较简单，一般情况下很少使用。下面我们使用 out 对象的 ```print```、```println``` 和 ```newLine``` 方法将内容输出到 HTML 中。```index.jsp``` 代码如下：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ page import="java.util.*"%>
<!DOCTYPE html>
<html>
<head>
<title>out示例</title>
</head>
<body>
    <%
        out.print("欢迎登录");
        out.newLine();
    %>
</body>
</html>
```

### session对象

**JSP session** 是``` javax.servlet.http.HttpSession``` 的实例对象，主要用来访问用户数据，记录客户的连接信息。

**示例**

在 login.jsp 页面登录，并跳转到 index.jsp。login.jsp 代码如下：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<title>session示例</title>
</head>
<body>
    <h2>用户登录</h2>
    <form action="index.jsp">
        用户名: <input type="text" name="username" /> <br> <br>
        密码: <input type="text" name="pass" /> <br> <br>
        <input type="submit" value="登录" />
    </form>
</body>
</html>
```

在 index.jsp 中，使用 session.setAttribute() 方法将用户名存储到 session 对象中，代码如下：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<title>session示例</title>
</head>
<body>
    <%
        String username = request.getParameter("username");
        out.print("欢迎" + username + "登录");
        session.setAttribute("sessname", username);
    %>
</body>
</html>
```

### application对象

JSP application 是 javax.servlet.ServletContext 的实例对象。在服务器部署应用和项目时，Web 容器仅创建一次 ServletContext 实例，也就是说 application 设置的任何属性和值可以用于整个应用（所有 JSP 页面）。可以将 application 对象看作 Web 应用的全局变量。一般用于保存应用程序的公用数据。

> application 对象在 Web 应用运行时一直存在于服务器中，非常占用资源，因此在实际开发中不推荐使用，否则容易造成内存不足等情况。

**示例**

可以使用 application 对象来保存 JSP 页面的访问人数，也就是我们常说的网站计数器，下面通过一个例子来演示。index.jsp 代码如下：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ page import="java.util.*"%>
<!DOCTYPE html>
<html>
<head>
<title>application示例</title>
</head>
<body>
    <%
        Integer count = (Integer) application.getAttribute("count");
        if (count == null) {
            count = 1;
        } else {
            count++;
        }
        application.setAttribute("count", count);
    %>
    您是第<%=count%>位访问客户！
</body>
</html>
```

### config对象

JSP config 是 javax.servlet.ServletConfig 的实例对象，一般用于获取页面和 Servlet 的初始化参数。

**示例**

在 web.xml 文件中定义 Servlet 名称和映射，然后使用 config 对象获取信息。web.xml 代码如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    version="2.5">
    <servlet>
        <servlet-name>BianChengBangServlet</servlet-name>
        <jsp-file>/index.jsp</jsp-file>
        <init-param>
            <param-name>url</param-name>
            <param-value>http://www.biancheng.net</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>BianChengBangServlet</servlet-name>
        <url-pattern>/index</url-pattern>
    </servlet-mapping>
</web-app>
```

在 index.jsp 页面获取 Servlet 名称以及初始化参数，代码如下：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<title>config示例</title>
</head>
<body>
    <%
        String sname = config.getServletName();
        String url = config.getInitParameter("url");
        out.print("Servlet名称为：" + sname + "<br>");
        out.print("初始化参数为：" + url + "<br>");
    %>
</body>
</html>
```

### pageContext对象

pageContext 是 javax.servlet.jsp.PageContext 的实例对象。pageContext 对象表示整个 JSP 页面。

### page对象

JSP page 的实质是 java.lang.Object 对象，相当于 Java 中的 this 关键字。page 对象是指当前的 JSP 页面本身，在实际开发中并不常用。

## EL表达式
