# thymeleafexamples-gtvg

## 配置

## 使用文本时的一些处理

1. 我们把一些字段写在.properties文件中，在引用这些字段的时候，我们需要使用`#{}`进行引用

2. 在标签中，我们定义th:text属性来引用字段`<p th:text="#{home.welcome}">Welcome to our grocery store!</p>`

3. 我们在文件中定义了字段，想要引用需要`org.thymeleaf.messageresolver.IMessageResolver`实现

4. 我们一般使用的是Standard Message Resolver `org.thymeleaf.messageresolver.StandardMessageResolver `，这个如果想要找

   /WEB-INF/templates/home.html文件中的message，就会在相同的文件夹中找到相同的名字的

   /WEB-INF/templates/home_en.properties

   /WEB-INF/templates/home_es.properties

   /WEB-INF/templates/home.properties等

5. 为了处理我们的模版，我们需要实现一个process方法

   ```java
   public void process(
               HttpServletRequest request, HttpServletResponse response,
               ServletContext servletContext, ITemplateEngine templateEngine)
               throws Exception;
   ```

   - 首先我们需要创建一个context，一个context必须要实现`org.thymeleaf.context.IContext`，有一个专业的继承`org.thymeleaf.context.IWebContext`，每一个都有自己特定的实现`org.thymeleaf.context.Context`和`org.thymeleaf.context.WebContext`

     我们调用`templateEngine.process("home", ctx, response.getWriter());`方法来执行

     ```java
     WebContext ctx = new WebContext(request, response, servletContext, request.getLocale());
     ctx.setVariable("today", Calendar.getInstance());
     templateEngine.process("home", ctx, response.getWriter());
     ```

   - 如果我们的message中有标签，需要我们在template中使用\<th:utext\>标签

   - 使用setVariable方法添加一个context variable，在前端中我们使用`${}`进行调用，如果我们传入一个对象user,使用${user.name}自动调用get方法获取值

## 标准表达式

### message

基本用法上面已经介绍，我们如果想要在message中添加参数，我们需要在定义message中使用

```
home.welcome=¡Bienvenido a nuestra tienda de comestibles, {0}!
```

```
<p th:utext="#{home.welcome(${session.user.name})}">
  Welcome to our grocery store, Sebastian Pepper!
</p>
```

类似于home.welcome是函数签名，我们按照顺序填入参数

### Variables

```html
<p>Today is: <span th:text="${today}">13 february 2011</span>.</p>
```

相当于`ctx.getVariables().get("today")`

基本的对象

\#ctx : the context object.

\#vars: the context variables.

\#locale : the context locale.

\#httpServletRequest : (only in Web Contexts) the HttpServletRequest object.

\#httpSession : (only in Web Contexts) the HttpSession object.

⚠️:还有其他的在这里就不写了，之后详细些

### selections

```html
<div th:object="${session.user}">
  <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
```

我们使用这种方法来使用对象中的属性

```html
<div th:object="${session.user}">
  <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
```

可以用也可以不用

### link url

绝对路径http://www.thymeleaf.org

相对路径

- **Page-relative**, like user/login.html
- **Context-relative**, like /itemdetails?id=3 (context name in server will be added automatically)
- **Server-relative**, like ~/billing/processInvoice (allows calling URLs in another context (= application) in the same server.
- **Protocol-relative URLs**, like //code.jquery.com/jquery-2.0.3.min.js

```html
<!-- Will produce 'http://localhost:8080/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html"
   th:href="@{http://localhost:8080/gtvg/order/details(orderId=${o.id})}">view</a>

<!-- Will produce '/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" th:href="@{/order/details(orderId=${o.id})}">view</a>

<!-- Will produce '/gtvg/order/3/details' (plus rewriting) -->
<a href="details.html" th:href="@{/order/{orderId}/details(orderId=${o.id})}">view</a>
```

- 使用的时候，会把渲染好了的th:href设置到href中
- 使用url参数的时候使用orderId=${o.id} ，如果有多个参数，使用@{/order/process(execId=${execId},execType='FAST')}
- 使用相对路径的时候，会自动在连接前面添加application context name.

###  Literals(字面量)

```html
<p>
  Now you are looking at a <span th:text="'working web application'">template file</span>.
</p>
```

如果我们想要直接应用字符串，使用单引号

```html
<p>The year is <span th:text="2013">1492</span>.</p>
<p>In two years, it will be <span th:text="2013 + 2">1494</span>.</p>
```

引入数字直接写，如果我们写了表达式，会自动计算

```html
<div th:if="${user.isAdmin()} == false"> ...
<div th:if="${variable.something} == null"> ...
```

使用\<th:if\>可以进行判断

使用+可以进行字符串添加

```
th:text="'The name of the user is ' + ${user.name}"
```

```html
<span th:text="|Welcome to our application, ${user.name}!|">
```

使用`||`可以把表达式直接在字符串中进行转换，不用使用+

### Conditional expressions

```html
<tr th:class="${row.even}? 'even' : 'odd'">
  ...
</tr> 
```

就是三木运算符，可以更加灵活的对内容进行修改

### Default expressions (Elvis operator)

```html
<div th:object="${session.user}">
  ...
  <p>Age: <span th:text="*{age}?: '(no age specified)'">27</span>.</p>
</div>
```

这个的含义等同于

```html
<p>Age: <span th:text="*{age}? *{age} : '(no age specified)'">27</span>.</p>
```

## Setting Attribute Values

th:attr 进行属性的修改

```html
<form action="subscribe.html" th:attr="action=@{/subscribe}">
  <fieldset>
    <input type="text" name="email" />
    <input type="submit" value="Subscribe me!" th:attr="value=#{subscribe.submit}"/>
  </fieldset>
</form>
```

```html
<img src="../../images/gtvglogo.png"
     th:attr="src=@{/images/gtvglogo.png},title=#{logo},alt=#{logo}" />
     
<img src="/gtgv/images/gtvglogo.png" title="Logo de Good Thymes" alt="Logo de Good Thymes" />

<img src="../../images/gtvglogo.png"
     th:src="@{/images/gtvglogo.png}" th:title="#{logo}" th:alt="#{logo}" />
```

th:value设置value

```html
<input type="submit" value="Subscribe me!" th:value="#{subscribe.submit}"/>
```

添加属性

```html
<input type="button" value="Do it!" class="btn" th:attrappend="class=${' ' + cssStyle}" />

<input type="button" value="Do it!" class="btn warning" />
```

## 迭代

在后台我们传入了一个list，我么使用

```html
<table> 
	<tr>
        <th>NAME</th>
        <th>PRICE</th>
        <th>IN STOCK</th>
    </tr>
    <tr th:each="prod : ${prods}">
        <td th:text="${prod.name}">Onions</td>
        <td th:text="${prod.price}">2.41</td>
        <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
    </tr>
</table>
```

Any object implementing java.util.Iterable
Any object implementing java.util.Map . When iterating maps, iter variables will be of class java.util.Map.Entry .
Any array
Any other object will be treated as if it were a single-valued list containing the object itself.

这些都可以被迭代

## 判断

```
th:if="${not #lists.isEmpty(prod.comments)}
```

true的条件

value非空

是boolean且为ture

是数字并且不是0

是字符并且不是0

是字符串并且不是“false”, “off” or “no”

不是 a boolean, a number, a character or a String.



是空的时候为false

## switch

```html
<div th:switch="${user.role}">
  <p th:case="'admin'">User is an administrator</p>
  <p th:case="#{roles.manager}">User is a manager</p>
  // 相当于default
  <p th:case="*">User is some other thing</p>
</div>
```

## Template Layout

```html
<div th:fragment="copy">
      &copy; 2011 The Good Thymes Virtual Grocery
</div>
```

首先，这个是定义在footer.html中的一个模版片段

我们在其他的地方如果想要调用这个模版片段

```html
<div th:include="footer :: copy"></div>
```

我们使用"templatename::domselector" 或者 templatename::[domselector]的形式进行

::domselector 或者 this::domselector 表示使用的是当前的这个里面的

```html
<div id="copy-section">
  &copy; 2011 The Good Thymes Virtual Grocery
</div>

<div th:include="footer :: #copy-section"></div>
```

这样也同样可以使用

### 带参数的代码块

```html
<div th:fragment="frag (onevar,twovar)">
    <p th:text="${onevar} + ' - ' + ${twovar}">...</p>
</div>
```

```html
<div th:include="::frag (${value1},${value2})">...</div>

<div th:include="::frag (onevar=${value1},twovar=${value2})">...</div>与参数的顺序无关，只需要指明是哪一个参数
```

如果我们在定义代码段的时候没有定义参数部分，我们必须使用

```html
<div th:include="::frag (onevar=${value1},twovar=${value2})">
    等价于
<div th:include="::frag" th:with="onevar=${value1},twovar=${value2}">
```

th:assert 进行断言，如果not抛出异常

## Local Variables

## Attribute Precedence

即使在同一个标签中，不同的属性执行也是有顺序的

所有的标签都是有优先级的

![image-20180910112753873](/var/folders/4v/0l6shpvn279745vgjnbbw7600000gn/T/abnerworks.Typora/image-20180910112753873.png)即使属性值防反了，也能又通要的效果

## 注释

<!-- -->

```
<!--/* This code will be removed at thymeleaf parsing time! */-->
```

Thymeleaf会绝对的把这之间的代码删掉

所以把<!--/*--><!--*\-->这一对标签进行对代码块的删除

当静态的打开一个页面，注释掉某一个代码块，经过Thymeleaf处理打开页面，有显示代码块

```html
<span>hello!</span>
<!--/*/
  <div th:text="${...}">
    ...
  </div>
/*/-->
<span>goodbye!</span>

相当于

<span>hello!</span>
  <div th:text="${...}">
    ...
</div>
<span>goodbye!</span>
```

## Inlining

使用`[[...]]`可以代替任何的th:text

```html
<body th:inline="text">
...
   <p>Hello, [[${session.user.name}]]!</p>
   <p>Hello, <span th:text="${session.user.name}">Sebastian</span>!</p>
...
</body>
```

inline属性中有三种text , javascript and none

这种写法虽然简单，但是我们如果使用这种写法的时候，静态启动页面的时候我们就不能得到一个页面的原型

### Script inlining

使用javascript或者dart

## Validation and Doctypes

我们想要让我们的模版有效，就要引入dtd文件，标准的有VALIDXML and VALIDXHTML，如果我们仅仅引入标准的文件

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"
```

我们就不能使用th:的东西了

所以我们在我们的页面中引入了

```
<!DOCTYPE html SYSTEM "http://www.thymeleaf.org/dtd/xhtml1-strict-thymeleaf-4.dtd">
```

这个里面内置了标准的文件

```
<html xmlns="http://www.w3.org/1999/xhtml"
	xmlns:th="http://www.thymeleaf.org">
```

同时也要配置这个