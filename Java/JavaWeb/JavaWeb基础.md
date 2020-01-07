# Java Web基础

# JavaWeb

[TOC]

## Tomcat

**部署**：

1. 将项目复制到webapps文件夹下，或直接复制或打包成war包复制

2. 修改`conf/server.xml`  

   ```xml
   <Host>
   ...
     <Context docBase="项目路径" path="访问路径"/>
   </Host>
   ```

3. `conf/Catalina/localhost/`下创建任意XXX.xml文件

   并在其中编写项目路径，删除path属性，此时，该项目的虚拟访问路径则为新建xml文件的名称

   ```xml
    <Context docBase="项目路径"/>
   ```

   推荐：动态部署，不需要重启服务器

**Tomcat部署war，可将项目整体或者将根目录的所有文件压缩为war文件，放在Tomcat/webapps/下，即可加载项目**

Tomcat/work 文件夹，会存放运行时生成的文件

## Servlet

### 概述

web.xml需要配置Servlet 别名、包路径、访问路径。

```xml
 <servlet>
        <servlet-name>DemoServlet</servlet-name>
        <servlet-class>com.jone.demo.DemoServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>DemoServlet</servlet-name>
        <url-pattern>/servlet</url-pattern>
    </servlet-mapping>
```

```java
import javax.servlet.*;
import java.io.IOException;

public class DemoServlet implements Servlet {

        
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {

    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {

    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {

    }
}
```

### 生命周期

1. 初始化：`init()` ：默认第一次访问时创建，执行可配置创建时机：

   web.xml配置 创建时机

   ```xml
   <servlet>
           <servlet-name>DemoServlet</servlet-name>
           <servlet-class>com.jone.demo.DemoServlet</servlet-class>
           <!--负数 -> 第一次访问时创建-->
           <!-- 0 或正整数 -> 服务器启动时创建-->
           <!--默认为-1-->
           <load-on-startup>-1</load-on-startup>
       </servlet>
       <servlet-mapping>
           <servlet-name>DemoServlet</servlet-name>
           <url-pattern>/servlet</url-pattern>
       </servlet-mapping>
   ```

   Servlet 是单例的，多个用户访问时，可能会有线程问题，尽量不要再servlet中定义成员变量，即使定义了成员变量，也不要对其进行修改操作。

2. 提供服务：`service()`：每次访问Servlet时就会被调用

3. 销毁：`destroy()`：只有正常关闭时才会执行

### 注解配置

Servlet 3.0支持注解配置，可以不需要web.xml配置

```java
//省略key，默认值为设置资源路径 
///@WebServlet({"/url","/url/url1})
@WebServlet("/url") 
public class ServletDemo2 implements Servlet {
  ...
}
/**
WebServlet注解中的属性：
name 名称
urlPatterns 资源路径(数组)
loadOnStartup 创建时机
...
*/
```

路径规则：

 1. /xxx

 2. /xxx/xxx

 3. *.xxx  自定义后缀

    > /* 通配符，优先级最低，如果没有对应匹配才会匹配通配符

### IDEA 与Tomcat的配置

1. Tomcat会为每一个项目单独生成一份配置文件

   启动服务器时，控制台会输出配置路径 ：`CATALINA_BASE`、`CATALINA_HOME`

2. 真正访问的是Tomcat 将项目部署的一个目录

   WEB-INF 目录下的资源不能被浏览器直接访问

### Servlet继承结构

   `Servlet(接口) <— GenericServlet(abstract) <— HttpServlet(abstract)`  

### Request

#### 继承结构：

   `ServletRequest(接口) <— HttpServletRequest(接口) <— RequestFacade(Tomcat包)`

#### 常见的方法

**关于请求头的一些常用方法**：

例：GET /demo/demo1?name=zhangsan http1.1

| 方法名                                            | 描述               | 结合案例的结果                              |
| ------------------------------------------------- | ------------------ | ------------------------------------------- |
| String getMethod()                                | 获取请求方式       | GET                                         |
| String getContextPath()                           | 获取虚拟目录       | /demo                                       |
| String getServletPath()                           | 获取Servlet路径    | /demo1                                      |
| String getQueryString()                           | 获取get请求参数    | name=zhangsan                               |
| String getRequestURI() <br>String getRequestURL() | 获取请求uri        | /demo/demo1<br/>http://localhost/demo/demo1 |
| String getProtocol()                              | 获取协议及版本     | http/1.1                                    |
| String getRemoteAddr()                            | 获取客户机IP       |                                             |
| String getHeader()                                | 获取指定请求头     |                                             |
| Enumeration<String> getHeaderNames()              | 获取所有请求头名称 |                                             |

```java
//设置编码，避免中文乱码
request.setCharacterEncoding("utf-8");
//获取请求体数据
BufferedReader br = request.getReader();
String str = null;
while((str.br.readLine())!=null){
  System.out.println(str)
}
```

#### 获取参数值的一些常用方法

| 方法名                                  | 描述                               | 说明 |
| --------------------------------------- | ---------------------------------- | ---- |
| BufferedReader getReader()              | 获取字符输入流，只能操作字符数据   |      |
| ServletInputStream getInputStream()     | 获取字节输入流，可操作所有类型数据 |      |
| String getParameter(String name)        | 根据参数名获取参数值               |      |
| String[] getParamterValues(String name) | 根据参数名称获取参数值的数组       |      |
| Enumeration<String> getParameterNames() | 获取所有请求参数名称               |      |
| Map<String,String[]> getParameterMap()  | 获取所有参数的map集合              |      |

#### 请求转发

```java
RequestDispatcher dispatcher = request.getRequestDispatcher("path")
dispatcher.forward(request,response)
```

特点：

1. 浏览器地址栏路径不发生变化
2. 只能转发到当前服务器内部资源
3. 转发是一次请求

#### 共享数据：

> 域对象：一个有作用范围的对象，可以在范围内共享数据
>
> request域：一次请求的范围，一般用于请求转发的多个资源中共享数据

| 方法                                      | 描述     |
| ----------------------------------------- | -------- |
| void setAttribute(String name,Object obj) | 存储数据 |
| Object getAttribute(String name)          | 获取数据 |
| void removeAttribute(String name)         | 移除数据 |

### Response

#### 常见返回状态码

| 状态码 | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| 1xx    | 服务器接收到消息，但没有接收完成                             |
| 2xx    | 成功：200                                                    |
| 3xx    | 重定向：302，访问缓存：304                                   |
| 4xx    | 客户端错误，如 参数错误400，没有权限：401 ，页面不存在：404，请求方式错误：405 |
| 5xx    | 服务器错误                                                   |

返回响应头的一些说明：

- `Context-Type` 本次响应体的数据格式和编码格式

- `Content-Disposition`什么格式打开

  in-line ：默认值，在当前界面打开

  attachment;fileName=xxx：以附件形式打开，文件下载

#### 常用方法

| 方法                                  | 描述       |
| ------------------------------------- | ---------- |
| setStatus(int sc)                     | 设置状态码 |
| setHeader(String name,String value)   | 设置响应头 |
| PrintWriter getWriter()               | 字符输出流 |
| ServletOutputStream getOutputStream() | 字节输出流 |

#### 重定向请求

```java
//方法1
//设置重定向状态码
response.setStatus(302);
//设置响应头location
response.setHeader("location","资源uri");

//方法2
response.sendRedirect("资源uri");

```

| 重定向(Redirect)特点                       | 转发(Forward)的特点                     |
| ------------------------------------------ | --------------------------------------- |
| 地址栏发生变化                             | 地址栏不变                              |
| 可以访问其他站点资源                       | 只能访问当前服务器下资源                |
| 重定向是两次请求，不能使用request 共享数据 | 转发是一次请求，可以使用request共享数据 |

 **路径写法**

1. 相对路径

   `./`：当前目录下

   `../`：当前父类目录下

2. 绝对路径

   可以使用`request.getContextPath()`获取虚拟路径

   即：`request.getContextPath()+"/uri"`

#### 字符输出流

```java
//避免中文乱码
response.setCharacterEncoding("utf-8");
//设置头编码，避免中文乱码
response.setHeader("content-type","text/html;charset=utf-8");
//或者
response.setContentType("text/html;charset=utf-8");

response.getWriter().write("你好.....");
```

#### 字节输出流

```java
//避免中文乱码
response.setCharacterEncoding("utf-8");
//设置头编码，避免中文乱码
response.setContentType("text/html;charset=utf-8");

response.getOutputStream().write("你好...".getBytes("utf-8"));
```

<span id="vertifyCode"></span>

#### 生成验证码

```java
int width = 100 ;
int height = 50 ; 
//内存中生成一张图片
BufferedImage image = new BufferedImage(width,height,BufferedImage.TYPE_INT_RGB) ;
// 获取画笔
Graphics g = image.getGraphics() ;
g.setColor(Color.RED) ;
g.fillRect(0,0,width,height) ;
...
g.drawString("",x,y);
// 输出图片
ImageIo.write(image,"jpg",response.getOutputStream) ;
```

```html
<html>
  <head>
    <script>
      window.onload = function(){
        var img = document.getElementById("img");
        img.click = function(){
          //添加时间戳，添加不重复参数，避免读取缓存
          var date = new Date().getTime();
          img.src = "servlet 的uri ?"+date;
        }
      }
        </script>
    </head>
  <body>
    <img id = "img" src = "servlet 的uri"/>
  </body>
</html>

```

### BeanUtil

简化获取参数转换为JavaBean的操作

```java
//首先导入commons-beanutils jar包
Map<String,String[]> map = request.getParamterMap();
User user = new user();
BeanUtils.populate(user,map);  
```

| 方法                             | 描述                        |
| -------------------------------- | --------------------------- |
| setProperty(bean,property,value) | 设置Bean中属性的Value       |
| getProperty(bean,property)       | 获取bean中属性值            |
| populate(Object obj,Map map)     | 将map中的键值对解析到bean中 |

### ServletContext

1. 概念：代表整个web应用，可以和程序的容器(服务器)通信

2. 获取：`request.getServletContext()` 、`HttpServlet.getServletContext()`

3. 功能

   1. 获取MIME Type

      MIME：互联网通信中定义的一种文件数据类型

      格式：大类型/小类型    例如：text/html、image/jpeg

      获取：`String getMimeType(String file)`

   2. 域对象 :共享数据

      | 方法                                      | 描述     |
      | ----------------------------------------- | -------- |
      | void setAttribute(String name,Object obj) | 存储数据 |
      | Object getAttribute(String name)          | 获取数据 |
      | void removeAttribute(String name)         | 移除数据 |

      **范围**：所有用户的所有请求的数据

   3. 获取文件真实路径

      方法：`String getRealPath()`  返回为文件对应目录，可以理解为云服务器Tomcat中文件的资源真实路径

### 获取项目中文件的真实路径

```java
/*
a.txt 存放在src路径下
b.txt 存放在项目根下
c.txt 存放在WEB-INF下

关于classLoader，只能获取src下的文件
*/
ServletContext context = request.getServletContext();
context.getRealPath("/WEB-INF/classes/a.txt");
context.getRealPath("/b.txt");
context.getRealPath("/WEB-INF/c.txt");
```



### 下载文件案例

需求：

1. 界面有一个超链接
2. 点击超链接显示下载提示框
3. 下载文件

> 超链接中的href：
>
> 1. 如果是图片的地址，那么将会展示图片，而不是下载，因为浏览器可以解析
>
> 2. 如果是视频，那么将会下载

步骤：

1. 定义界面，超链接的href指向servlet并传递资源名称(fileName)
2. 创建servlet
   1. 根据fileName使用字节流将文件加载到内存中
   2. 设置response 的响应头`content-disposition:attachment;filename=xxx`
   3. 将数据写出

```xml
<body>
<a href="servlet uri?filename=xxx"></a>
</body>
```



```java
//servlet 中逻辑处理：
String filename = request.getParamter("filename");
ServletContext context = request.getServletContext();
String realPath = context.getRealPath("/img/"+filename);
FileInputStream fis = new FileInputStream(realPath);
String mimeType = context.getMimeType(filename);
response.setHeader("content-type",mimeType);
//解决中文文件名问题
String agent = request.getHeader("user-agent");
//根据浏览器版本，对文件名称进行更改，编码
//设置头
response.setHeader("conotent-disposition","attachment;filename="+filename);
ServletOutputStream sos = response.getOutputStream();
byte[] buff = new byte[1024*8];
int len = 0;
while((len = fis.read(buff))!=-1){
  sos.write(buff,0,len);
}
fis.close()
```

问题：

   文件名称包含中文，下载框不能被正常显示：

解决：

   根据浏览器版本不同，使用编码对文件名进行更改。以便于显示正确的中文字符。



## Cookie&Session

会话：一次会话包含多次请求和响应

功能：在一次会话范围内的多次请求间共享数据

方式：

   客户端：Cookie

   服务器：Session

### Cookie

在一次会话中，将数据保存到**客户端**，多次请求间的共享数据

快速入门：

```java
//1. 创建Cookie对象
new Cookie(String name,String value);
//2. 发送Cookie
response.addCookie(Cookie cookie);
//3. 获取Cookie
Cookie[] request.getCookies();
```

#### 细节问题

1. 一次是否可以发送多个Cookie？

   可以

2. cookie 保存时长

   - 默认情况下，关闭浏览器，cookie失效

   - 设置cookie，持久化存储：`setMaxAge(int seconds);` 

       取值：1. 正数 —>写到硬盘，持久化存储。`seconds`秒后被删除

               2. 负数 —>默认值
    
               3. 零 —>删除cookie

3. cookie 是否能保存中文

   tomcat 8 之前，不能直接存储中文。需要将中文转码--一般采用url编码(%E3….)

   tomcat 8 之后，可以存储中文

4. cookie 数据共享问题

   1. 在一个tomcat服务器中，部署多个web项目，cookie默认不能共享

   `setPath(String path);`：设置cookie的获取路径，默认情况下设置当前的虚拟路径

   ```java
   Cookie cookie = new Cookie();
   //只有当前项目可获取
   cookie.setPath("/cookie");
   //tomcat所有服务器项目都能共享获取cookie
   cookie.setPath("/");
   ```

   2. 不同tomcat服务器中的cookie共享

      `setDomain(String path);`

      如果设置一级域名相同，多个服务器之间cookie可共享

#### 特点和作用

1. cookie存储数据在客户端
2. 浏览器对单个cookie大小有限制(4K左右)，以及对同一个域名下总cookie数量有限制(20个)，不同浏览器有不同的限制。

作用：

1. 一半用于存储一些不太敏感的数据
2. 不登录的情况下，服务器对客户端的身份识别



### Session

`HttpSession`:在一次会话中，将数据保存到**服务器端**，多次请求间的共享数据，

获取：`request.getSession();`

| 方法                                      | 描述     |
| ----------------------------------------- | -------- |
| Object getAttribute(String name)          | 获取参数 |
| void setAttribute(String name,Object obj) | 设置参数 |
| void removeAttribute(String name)         | 移除参数 |

#### 原理

   服务器怎么确定在一次会话的多次请求中获取的session是同一个的？

   session 是依赖于cookie的。

   第一次获取session，没有cookie，会在内存中创建一个cookie对象。在响应时，响应头中会包含一个`set-cookie:JSESSIONID=xxx`，下一次请求时，request会携带`JSESSIONID`的请求头，服务器会根据`JSESSIONID`寻找对应的session

```mermaid
sequenceDiagram
Title:原理
客户端->>服务器: 第一次请求(没有cookie)
Note over 服务器: 创建session、cookie
Note over 服务器: 携带JSESSIONID
服务器-->>客户端: 响应
客户端->>服务器: 再次请求(cookie、JSESSIONID)
Note over 服务器:查找Session
服务器-->>客户端: 响应
```

#### 细节问题

1. 客户端关闭，服务器不关闭，session是否相同

   默认不是。

   如果期望指定时间内，不管客户端是否关闭都能获取到相同的session：

   ```java
   HttpSession session = request.getSession();
   Cookie cookie = new Cookie("JSESSIONID",session.getId());
   cookie.setMaxAge(60*60);//一小时有效期
   response.addCookie(cookie);
   ```

   > 个人理解：第一次访问服务器时，将session对应的sessionId存放进cookie 返回客户端，客户端保存了cookie，就算关闭了客户端，再次请求服务器时，会携带之前存放的cookie，服务器会根据cookie中的sessionId再去查找之前的session，即获取到与之前相同的session

2. 服务器关闭，客户端不关闭，session是否相同

   服务器关闭，session被销毁，不相同。

   确保数据不丢失的方法：

        1. session 钝化：服务器正常关闭之前，将session序列化到硬盘上
        2. session 活化：服务器启动后，将session文件转为内存中session对象

   > 如果项目直接在Tomcat中使用的话，Tomcat默认已经完成对session的钝化和活化，但是Idea不支持。
   >
   > 1. Tomcat部署项目，其对session的钝化和活化：
   >
   > 可将项目放在Tomcat/webapps/下加载，启动tomcat试验上述问题，在正常关闭服务器时，会在`Tomcat/work/Catalina/localhost/`项目名称的路径中看到session钝化后的本地文件。
   >
   > 重新启动Tomcat，则会默认加载本地钝化的文件，加载后文件会被删除，此时原有session已被加载到内存中，再次访问，会是之前的session对象。
   >
   > 2. Idea 部署项目失败的原因
   >
   > Idea在重新启动时，会将Idea运行文件存储目录下的work文件夹删除，然后再重新创建一个新的work目录
   >
   > *注意，此处是Idea运行文件中的work*
   >
   > 正常服务器会直接部署在Tomcat中，所以基本不存在以上这个情况。

3. 关于session 销毁

   1. 服务器关闭

   2. session对象调用`invalidate()`

   3. session默认是30分钟。可在`Tomcat/conf/web.xml`中配置：

      ```xml
      <session-config>
      <session-timeout>30</session-timeout>
      </session-config>
      ```

      也可在自己的项目中配置

#### session的特点

1. 用于存储一次会话多次请求的数据，存储在服务器端
2. **可以存储任意类型，任意大小的数据**



### cookie与session 的区别

1. cookie存在客户端，session存在服务器端

2. cookie有数据大小限制，session没有大小限制

3. cookie存在客户端，数据相对不安全

   session存在服务器端，数据安全



### 案例：用户登录包含验证码

需求：

1. 访问带有验证码的jsp页面

1. 用户输入用户名、密码、验证码
2. 验证错误则显示错误信息，否则显示登录成功

实现：

1. jsp 界面编写(省略，[生成验证码](#vertifyCode))

2. 将生成的验证码字符串保存到session中(`session.setAttribute()`)

3. 在验证的servlet中从session中取出验证码当前的字符串，与parameter中的值进行比较

4. 如果比较通过，在判断用户名、密码是否正确，再选择跳转哪个界面

   

```flow
st=>start: 打开界面
e=>end: 结束
op1=>operation: 生成验证码，保存到session中
op3=>operation: 从session中取出验证码
op4=>operation: 提示验证码错误
op5=>operation: 提示用户名/密码错误
op6=>operation: 登陆成功
sub1=>subroutine: My Subroutine|invalid
cond=>condition: 验证验证码
cond1=>condition: 验证用户名密码

st->op1->op3->cond->e
cond(yes)->cond1->
cond(no)->op4->e

cond1(no)->op5->e
cond1(yes)->op6->e

```



## JSP

概述：本质上还是servlet，编译时会生成servlet，编译为.class文件。

### 编写java的方式

1. `<%  %>`  相当于写在servlet `service()`方法中。(局部)
2. `<%! %>` 定义声明，转换后在java类的成员位置。(全局)
3. `<%= %>` 直接输出到页面上。转换后是`out.write("内容")`

### 注释

`<!-- -->`：只能注释html

`<%-- --%>`：jsp注释，可注释jsp中所有类型

### 指令

用于配置jsp页面，导入资源文件

格式： `<%@ 指令名称 属性名1=属性值1 属性名2=属性值2 ...%> `

1. page :配置jsp页面

   `<%@ page contentType="text/html;charset=UTF-8" language="java" %>`

   属性值介绍：

   | 属性        | 描述                                                         |
   | ----------- | ------------------------------------------------------------ |
   | contentType | 设置mime、编码格式。设置编码格式时，高级IDE才生效，低版本的需要通过设置pageEncoding属性 |
   | import      | 导包                                                         |
   | errorPage   | 发生错误时的跳转页面                                         |
   | isErrorPage | 是否为错误页面，标注为错误页面可通过页面中的exception内置对象获取错误信息 |
   | isELIgnored | 是否忽略当前页面的EL表达式，如果忽略，不会执行EL，会原样输出 |

2. include ：页面包含，导入页面资源文件

   `<%@  include file="xxx.jsp"%>`

3. taglib：导入资源

   `<%@ taglib prefix="xxx" uri="xxx"$>`

   prefix：自定义前缀

   uri：库链接地址

   如：`<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>`

### 内置对象

在jsp页面中不需要获取和创建，可以直接使用的对象，共9个  

| 变量名      | 类型                | 描述                                      |
| ----------- | ------------------- | ----------------------------------------- |
| request     | HttpServletRequest  | 一次请求访问的多个资源                    |
| response    | HttpServletResponse | 响应对象                                  |
| pageContext | PageContext         | 当前页面共享数据，还可获取其他8个内置对象 |
| session     | HttpSession         | 一次回话的多个请求间共享数据              |
| application | ServletContext      | 所有用户间共享数据                        |
| page        | Object              | 当前页面(Servlet)的对象，this             |
| out         | JspWriter           | 输出对象，输出到页面上                    |
| config      | ServletConfig       | Servlet配置对象                           |
| exception   | Throwable           | 异常对象                                  |

### EL表达式

概念：Expression Language 表达式语言

作用：替换和简化jap页面中java代码的编写

语法：${表达式}   

   例：`${3>4}`  输出false

如果要原样输出 可给page设置isELIgnored，或 添加转义符 `\${表达式}`

1. 运算符

   | 类型       | 写法                                                         |
   | ---------- | ------------------------------------------------------------ |
   | 算数运算符 | +、-、*、/(div)、%(mod)                                      |
   | 比较运算符 | >、<、=、>=、<=、==、!=                                      |
   | 逻辑运算符 | &&(and)、\|\|(or)、!(not)                                    |
   | 空运算符   | 是否为null或长度为零，例：`${empty list}`、`${not empty list}` |

2. 获取值

   EL表达式只能从域中获取值

   语法：

   1. `${域名称.键名}`： 从指定域中获取指定键的值
   2. `${键名}`：依次从最小的域中查找是否有该键对应的值，知道找到为止

   | 域对象(域从小到大排列) | 域名城                      |
   | ---------------------- | --------------------------- |
   | pageScope              | pageContext                 |
   | requestScope           | request                     |
   | sessionScope           | sesson                      |
   | applicationScope       | application(servletContext) |

   例：

   ```jsp
   <%-- 在request域中存储了name="123" --%>
   <%-- 以下两种写法效果相同： --%>
   
   <%-- 1： --%>
   ${requestScope.name}
   
   <%-- 2： --%>
   <%=request.getAttribute("name")==null?"":request.getAttribute("name")%>
   
   ```

3. 获取对象、list、map的值

   语法：`${域名称.键名.属性名}`，本质上是调用对象的getter方法

   1. 获取对象

   ```jsp
   <% 
    User user = new User();
    user.setName("123");
    request.setAttribute("u",user);
   %>
   ${requestScopo.u}
   
   <%-- 
     通过的是对象的属性来获取，即setter、getter 去掉set、get，剩余部分首字母变小写 
   --%>
   
   ${u.name}
   ${requestScopo.u.name}
   ```

   2. 获取list

      语法：`${域名称.键名[index]}`

   3. 获取map 

      语法：`${域名称.键名.key名}`/`${域名称.键名.["key名"]}`

4. 隐式对象

   获取虚拟目录：`${pageContext.request.contextPath}`， 即：`request.getContextPath()`

       可以用在Form表单的Action属性中，动态获取虚拟目录：`<form action="${pageContext.request.contextPath}/xxx"> ...`

### JSTL

1. 概念：JavaServer Pages Tag Library  JSP标准标签库，用于简化和替换jsp页面上的java代码

   

2. 使用：
   1. 导入jar 
   2. 使用taglib指令引入标签库  <%@ tablib %>
   3. 使用标签

3. 标签：[菜鸟教程链接](<http://www.runoob.com/jsp/jsp-jstl.html>)

4. 常用标签简单实例

   ```jsp
   
     <%
       List list = new ArrayList();
       list.add("abc");
       request.setAttribute("list",list);
       request.setAttribute("number",10);
     %>
   
     <%--if表达式，Test属性为必须，接收boolean值，不存在else 标签，如需要，则再定义一个if标签--%>
     <c:if test="true">如果为真显示，为假不显示</c:if>
     <c:if test="${not empty list}">list is not null</c:if>
   
     <%--choose 表达式，类似于java switch --%>
     <c:choose>
       <c:when test="${number==1}">1</c:when>
       <c:when test="${number==2}">2</c:when>
       <c:when test="${number==3}">3</c:when>
       <c:when test="${number==4}">4</c:when>
       <c:otherwise>其他情况~~~</c:otherwise>
     </c:choose>
   
     <%--forEach 表达式，遍历--%>
     <c:forEach begin="0" end="10" var="i" step="1">
       ${i}<br>
     </c:forEach>
   
     <c:forEach var="item" items="${list}" varStatus="s">
       ${item} ${s.index} ${s.count} <br>
     </c:forEach>
   ```

   

## Filter

### 配置

`urlPatterns` 为过滤路径

注：如果`urlPatterns` 为`/*`，那么在访问所有资源之前，都需要过滤

1. web.xm配置：

```xml
<filter>
        <filter-name>demo1</filter-name>
        <filter-class>com.jone.demo.FilterDemo</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>demo1</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

2. 注解配置：

```java
/**
 * 写法：
 * @WebFilter(filterName = "FilterDemo",urlPatterns = "/*")
 * @WebFilter( "/*")//此处默认为urlPatterns
 *
 */
```

### 生命周期

`init`：服务器启动后，会创建过滤器，执行init方法，只执行一次

`doFilter`：每次访问拦截资源时执行

`destroy`：服务器关闭时，Filter销毁，服务器正常关闭时，会调用destroy方法

 1. 执行流程

    即`doFilter`中代码执行顺序：

    1. 请求时，执行放行之前的代码
    2. 放行 ：`chain.doFilter(req, resp);`
    3. 响应时会执行放行之后的代码

    ```java
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
            //处理请求
             ...
    
            // 放行
            chain.doFilter(req, resp);
    
            //响应请求
             ...
        }
    ```

    

### 拦截路径、方式

1. 拦截路径

   | 配置         | 执行                            |
   | ------------ | ------------------------------- |
   | `/index.jsp` | 只有访问index.jsp资源时         |
   | `/user/*`    | 访问user下的资源时              |
   | `*.jsp`      | 后缀名拦截，所访问有jsp文件资源 |
   | `/*`         | 所有资源                        |

2. 拦截方式

   1. 注解设置：设置dispatherTypes

      | dispatherTypes属性值 | 描述                     |
      | -------------------- | ------------------------ |
      | REQUEST              | 浏览器直接请求时，默认值 |
      | FORWARD              | 转发访问资源             |
      | INCLUDE              | 包含访问资源             |
      | ERROR                | 错误跳转资源             |
      | ASYNC                | 异步访问资源             |

      ```java
      @WebFilter(value = "/*",dispatcherTypes = DispatcherType.REQUEST)
      /** 
      * 多个拦截方式配置
      *@WebFilter(value = "/*",dispatcherTypes =  {DispatcherType.REQUEST,DispatcherType.FORWARD })
      */
      public class FilterDemo implements Filter {
      ...
      }
      ```

   2. web.xml配置

      ```xml
      <filter>
              <filter-name>demo1</filter-name>
              <filter-class>com.jone.demo.FilterDemo</filter-class>
          </filter>
          <filter-mapping>
              <filter-name>demo1</filter-name>
              <url-pattern>/*</url-pattern>
              <dispatcher>REQUEST</dispatcher>
              <dispatcher>FORWARD</dispatcher>
          </filter-mapping>
      ```

### 过滤器链

1. 默认执行顺序：

   如果有两个过滤器A、B， A放行->B放行->资源->B响应->A响应

2. 配置执行顺序：

   注解配置：**按类名的逐个比较字符大小，值小的先执行**

   web.xml：按声明顺序，写在上面的先执行

### 案例

#### 登陆过滤器

```java
@WebFilter(value = "/*")
public class LoginFilter implements Filter {
    public void destroy() {
    }

    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        HttpServletRequest request = (HttpServletRequest) req;
        String uri = request.getRequestURI();
        /**
         * 是否要去登陆，或者请求一些不需要登陆的操作 css、照片、js、验证码等
         */
        if (uri.contains("/login.jsp") ||
                uri.contains("/loginServlet") ||
                uri.contains("/css/") ||
                uri.contains("/fonts/") ||
                uri.contains("/js/") ||
                uri.contains("/checkCodeServlet")
        ) {//不需要登陆的情况，直接放行
            chain.doFilter(req, resp);
        }else{
            Object user = request.getSession().getAttribute("user");
            if (user != null) {//登陆过了,放行
                chain.doFilter(req, resp);
            }else{
                request.setAttribute("log_msg","还没有登陆");
                request.getRequestDispatcher("/login.jsp").forward(request,resp);
            }
        }
    }

    public void init(FilterConfig config) throws ServletException {
    }
}
```

#### 敏感词过滤

```java
@WebFilter("/*")
public class ReplaceFilter implements Filter {

    private List<String> list = new ArrayList<String>();


    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {

        ServletRequest request = (ServletRequest) Proxy.newProxyInstance(req.getClass().getClassLoader(), req.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if (method.getName().equals("getParameter")) {
                    String value = (String) method.invoke(req, args);
                    if (value != null && (list.contains(value))) {
                        value ="***";
                        return value;
                    }
                }
                return method.invoke(req, args);
            }
        });
        chain.doFilter(req, resp);
    }

    public void init(FilterConfig config) throws ServletException {
        ServletContext servletContext = config.getServletContext();
        String realPath = servletContext.getRealPath("/WEB-INF/classes/roles.txt");
        try {
            BufferedReader br = new BufferedReader(new FileReader(realPath));
            String line= null;
            while ((line= br.readLine()) !=null) {
                list.add(line);
            }
            br.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void destroy() {
    }
}
```

## Listener

web 三大组件之一。

ServletContextListener: 监听servletContext对象的创建和销毁

- void contextDestroyed(ServletContextEvent sce)：销毁
- void contextInitialized(ServletContextEvent sce)  ：创建



1. 创建`ServletContextListener` 的实现类

2. 配置Listener

   1. web.xml 配置

      ```xml
       <listener>
              <listener-class>com.jone.model.ServletLoadListener</listener-class>
          </listener>
      ```

   2. 注解配置

      ```java
      @WebListener
      ```

      

简单使用

```java
@WebListener
public class ServletLoadListener implements ServletContextListener {

    /***
     * 服务器启动默认创建ServletContext
     * @param sce
     */
    @Override
    public void contextInitialized(ServletContextEvent sce) {

        //1.获取ServletContext对象
        ServletContext servletContext = sce.getServletContext();
        //2.加载资源文件,
        /***
         * init_parameter 配置在web.xml 中，可指定为对应资源文件的路径，里面存放如全局配置等信息
         * 在此处获取文件的流进行加载
         */
        String parameter = servletContext.getInitParameter("init_parameter");
        //3.获取资源文件路径
        String realPath = servletContext.getRealPath(parameter);

        try {
            FileInputStream inputStream = new FileInputStream(realPath);
            //inputStream...
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }

    /***
     * 服务器正常关闭时，销毁ServletContext
     * @param sce
     */
    @Override
    public void contextDestroyed(ServletContextEvent sce) {

    }
}

```

web.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

<!-- web.xml 配置方式-->  
<!--    
        <listener>
        <listener-class>com.jone.model.ServletLoadListener</listener-class>
    </listener>
-->  

<!-- 对应为 存放在src 下的xml配置文件-->
    <context-param>
        <param-name>init_parameter</param-name>
        <param-value>/WEB-INF/classes/applicationContext.xml</param-value>
    </context-param>
  
</web-app>
```



## JQuery

三大版本说明

1. 1.x：兼容ie678 ，使用最为广泛，功能不再新增，最终版：1.12.4。(2016.5)
2. 2.x：不兼容ie678，功能不再新增，最终版：2.2.4。(2016.5)
3. 3.x：不兼容ie678，只兼容最新的浏览器

Jquery 包中包含 `xxx.js`、`xxx.min.js`。

   `xxx.js`：开发版本，有良好的缩进和注释

   `xxx.min.js`：生产版本，没有缩进，体积更小，加载更快

简单案例：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>JQueryDemo</title>
    <script src="js/jquery-3.4.0.js"></script>
</head>
<body>

<div id="div1">div1...</div>
<div id="div2">div2...</div>
<script>
    // 普通js写法
    var div1 = document.getElementById("div1");
    alert(div1.innerHTML);
    // jQuery 写法
     var div2 = $("#div2");
    alert(div2.html());
</script>
</body>
</html>
```

### jQuery对象与js对象的转换

1. js 和jquery对象方法不通用

2. 转换方式

   js   —>  jquery :`$(js对象)`

   jquery   —>  js :`jq对象[索引]`/`jq对象.get(索引)`

   

```jsp
<script>

 /***
     * 获取所有div 元素，更改其内容
     */
    //js方式，返回的是数组，对数组遍历进行更改
    var tags = document.getElementsByTagName("div");
    for (var i = 0; i < tags.length; i++) {
        tags[i].innerHTML = "div tags" ;
    }

    // jquery方式，直接进行操作
    var $tags = $("div");
    $tags.html("div jquery")

    // 注：js 和jquery对象方法不通用，如以下这句话，是不起作用的：
    $tags.innerHTML = "div jquery";
</script>
```



### 选择器

筛选具有相似特征的元素(标签)。

[jQuery 选择器](http://www.runoob.com/jquery/jquery-selectors.html)

#### 基本语法

 1. 事件绑定

    ```jsp
    <body>
      <div id = "div2">
        div2...
      </div>
      <script>
            //点击事件绑定
        $("#div2").click(function () {
            alert("click!")
        });
      </script>
    </body>
    ```

    

 2. 入口函数

    ```jsp
    <header>
    <script>
        /**
         * 页面加载后再执行
         */
    
        //1. js window.onload
        window.onload = function (ev) {
    
        };
        //2. jquery
        $(function () {
    
        });
    
        function fun1() {
            alert("fun1")
        }
    
        function fun2() {
            alert("fun2")
        }
        /***
         * onload 会被覆盖，以最后定义的为准，
         * 如以下这个案例，只会执行fun2，包含fun1的onload方法被覆盖
         */
        window.onload = fun1;
        window.onload = fun2;
    
        /***
         * jquery 入口函数可多次定义，且不会被覆盖
         * 以下 fun1 fun2都会被执行
         */
        $(fun1);
        $(fun2);
        
    </script>
    </header>
    ```

    

 3. 样式控制

    ```jsp
    <script>
            //1.
        $("#div2").css("background-color","red")
        //2. key 可用来被检查，按下command/ctrl 类似变量
        $("#div1").css("backgroundColor","pink")
    </script>
    ```

#### 基本选择器

| 名称                   | 写法                    | 说明                       |
| ---------------------- | ----------------------- | -------------------------- |
| 标签选择器(元素选择器) | $("html标签名")         | 获取所有匹配标签名称的元素 |
| id选择器               | $("#id属性值")          | 获取指定id属性值的元素     |
| 类选择器               | $(".class的属性值")     | 获取指定class属性值的元素  |
| 并集选择器             | $("选择器1,选择器2,….") | 获取多个选择器对应的元素   |

#### 层级选择器

| 名称       | 写法       | 说明                         |
| ---------- | ---------- | ---------------------------- |
| 后代选择器 | $("A B")   | A下所有的B元素，包含所有层级 |
| 子选择器   | $("A > B") | A下子元素中所有的B元素       |

#### 属性选择器

| 名称           | 写法                    | 说明                                     |
| -------------- | ----------------------- | ---------------------------------------- |
| 属性名称选择器 | $("A[属性名]")          | 获取含有指定属性的元素                   |
| 属性选择器     | $("A[属性名='值']")     | 获取含有指定属性并且值为指定值的元素     |
| 复合属性选择器 | $("A[属性名='值']\[]…") | 获取含有多个指定属性并且值为指定值的元素 |



#### KV判断符号

| 符号 | 写法         | 说明     |
| ---- | ------------ | -------- |
| =    | 属性名='**'  | 相等于   |
| !=   | 属性名!='**' | 不等于   |
| ^=   | 属性名^='**' | 以**开头 |
| $=   | 属性名$='**' | 以**结尾 |
| *=   | 属性名*='**' | 包含**   |

[jQuery选择器]([http://www.runoob.com/jquery/jquery-tutorial.html](http://www.runoob.com/jquery/jquery-tutorial.html))基于[CSS选择器](http://www.runoob.com/cssref/css-selectors.html)，还有一些自定义的选择器