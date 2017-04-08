JSP语法

# 1. JSP页面基本构成

![](http://upload-images.jianshu.io/upload_images/1124873-7057a746e8ac4833.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.1 JSP运行的4个关键阶段
当浏览器向Web应用服务器请求一个JSP页面时，Web应用服务器将其转换为一个Servlet文件（即一个.java文件），然后将这个Java文件编译为一个字节码文件（即一个.class文件）。最后Web应用服务器加载转换后的Servlet实例，处理客户端的请求，并返回HTML格式的响应回应给客户端浏览器。
![](http://upload-images.jianshu.io/upload_images/1124873-08932cab100f3b0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 2. JSP指令标识
JSP中包含了page、include和taglib共3个指令标识。

- 语法结构：
```
<%@ 指令名 属性1="属性值1" 属性2="属性值2" …%>
```

## 2.1 页面指令page
定义与整个JSP页面相关的属性，比如JSP页面的编码、内容类型及引用的类库等。

- language属性：
指定当前页面中使用的语言。
- contentType属性：
设置JSP页面的MIME类型和字符编码，浏览器会根据该属性指定的类型和编码显示网页内容。
- pageEncoding属性：
设置JSP页面的编码格式，在JSP页面中所有代码都使用该属性指定的字符集。
- import属性：
导入JSP页面中的类包，导入后在JSP页面中可以通过嵌入Java代码的方法使用这些类包。
- buffer属性：
设置out对象使用的缓冲区大小，默认为8 KB。
- autoFlush属性：
指定当缓冲区已满时自动将缓冲区中的内容输出到客户端，默认值为true。如果设置为false，当缓冲区已满时将抛出JSP Buffer overflow异常。
- isErrorPage属性：
将当前JSP页面设置为错误处理页面，以处理另一个JSP页面的错误，即异常处理。
- errorPage属性：
指定当前页面出现异常时调用的另一个页面（即错误处理页面），在错误处理页面中必须将isErrorPage属性值设置为true。
session属性：
指定当前JSP页面是否支持session。
- isELIgnored属性：
指定是否禁用EL表达式。
- isThreadSafe属性：
指定JSP页面是否是线程安全的。

## 2.2 文件包含指令include
include指令是静态包含，即被包含文件中所有内容会被原样包含到该JSP页面中。即使被包含文件中有JSP代码，在包含时也不会被编译执行。将两个页面组合成一个页面后编译处理，最后返回结果页面。

include指令的语法格式如下：
```
<%@include file="path"%>
```

![include指令包含文件的过程](http://upload-images.jianshu.io/upload_images/1124873-18d47e7fca9c57bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.3 用引用标签库指令taglib
taglib指令用于声明一个标签的引用，在JSP页面之中声明了哪个标签库的引用，即可在JSP页面中调用哪个标签。该指令的语法格式如下：
```
<%@taglibprefix="tagPrefix"uri="tagURI" %>  
```
- taglib属性：声明指令为taglib指令。 
- prefix属性：指定标签库的前缀。   
- uri属性：指定标签库文件的位置。

# 3. JSP脚本
JSP中的脚本标识包括JSP表达式（Ex-pression）、声明标识（Declaration）和脚本程序（Scriptlet）。通过这些标识，在JSP页面中可以如编写Java程序一样来声明变量、定义方法或执行各种表达式的运算。

## 3.1 JSP表达式
JSP表达式用于在页面中输出信息，它可以插入到网页的文本中用于输出文本内容，也可以插入到HTML标记中用于动态设置属性值。JSP表达式的语法格式如下：
```
<%= 表达式%>
```

## 3.2 声明标识
在一个JSP页面中也可以如编写Java文件一样定义成员变量及成员方法，此种方式即声明标识的应用。使用声明标识在JSP页面中定义变量或方法是全局的，同时要求遵循Java语言的语法。声明标识的语法格式如下：
```
<%! 声明变量或方法的代码 %>
```

# 4. JSP动作标识
在JSP规范中定义了一些标准的动作，用于为请求处理阶段提供信息。如操作JavaBean、包含其他文件和执行请求转发等，这些动作在请求处理阶段按照在页面中出现的顺序执行。由于JSP动作标识基于XML语法实现，所以在JSP页面中只需要遵循XML语法调用即可。
JSP动作标识的通用语法格式如下：
```
<标识名 属性1="值1" 属性2="值2"…/>
```
或：
```
<标识名 属性1="值1" 属性2="值2"…>   
  <子标识名 属性1="值1" 属性2="值2"…/>  
   … <!--多个子标识-->
</标识名>
```

## 4.1 包含动作标识<jsp：include>
<jsp：include>动作标识用于包含其他页面，被包含的页面可以是动态页面或静态页面。<jsp：include>包含的原理是将被包含的页面编译处理后将结果包含在页面中，包含页面的过程如图4.9所示。

![](http://upload-images.jianshu.io/upload_images/1124873-19c1a18bc2273b72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当浏览器第1次请求一个使用<jsp：include>包含其他页面的页面时，Web容器首先会编译被包含的页面。然后将编译处理后的返回结果包含在页面中，之后编译包含页面，最后将两个页面组合的结果回应给浏览器。

## 4.2 请求转发的动作标识<jsp：forward>
<jsp：forward>动作标识将当前请求转发到其他Web资源（HTML页面、JSP页面和Servlet等），在执行请求转发之后当前页面将不再执行，而是执行该标识指定的目标页面。执行请求转发的基本流程如图所示。
<jsp：forward>动作标识的语法格式如下：
```
<jsp:forward page="url"/>
```
或：
```
<jsp:forward page="url">
    子动作标识<jsp:param>
</jsp:forward>
```

page属性指定请求转发的目标页面，该属性值可以是一个指定文件路径的字符串或表示文件路径的JSP表达式。

## 4.3 子动作标识<jsp：param>
JSP的动作标识<jsp：param>可以作为其他标识的子标识，用于为其他标识传递参数，其语法格式如下：
```
<jsp:param name="参数名" value="参数值" />
```
通过<jsp：param>动作标识指定的参数将以“参数名=值”形式添加到请求中，其功能与在文件名后面直接加“?参数名=参数值”相同。

## 4.4 动作标识 <jsp：useBean>
使用<jsp：useBean>动作标识可以在JSP页面中创建一个Bean实例，并且通过属性设置可以将该实例保存在JSP中的指定范围内。如果在指定的范围内已经存在指定的Bean实例，那么将使用这个实例，而不会重新创建。通过<jsp：useBan>标识创建的Bean实例可以在Scriptlet中应用。

该标识的语法格式如下：
```
<jsp:useBean
       id="变量名"
       scope="page|request|session|application"
        {
        class="package.className"|type="数据类型"|class="package.className"
        beanName="package.className" 
        }
/>
<jsp:setProperty name="变量名" property="*"/>
```
也可以在标识体内嵌入子标识或其他内容：
```
<jsp:useBean
  id="变量名" 
  scope="page|request|session|application" >  
  <jsp:setProperty 
    name="变量名" 
    property="*"/>
 </jsp:useBean>
```
<jsp:useBean>标识中各属性用法

- id：定义一个变量名，程序中将使用该变量名引用所创建的Bean实例
- type：指定id属性所定义变量的类型
- scope：定义Bean实例的范围，默认值为page，其他可选值为rquest、session和application
- class：指定一个完整的类名，与beanName属性不能同时存在。若未设置type属性，则必须设置class属性
- beanName：指定一个完整的类名，与class属性不能同时存在。设置该属性时必须设置type属性，其属性值可以是一个表示完整类名的表达式

## 4.5 动作标识<jsp：getProperty>
<jsp：getProperty>属性用来从指定的Bean中读取指定的属性值并输出到页面中，该Bean必须具有getXXX()方法。

<jsp：getProperty>标识的语法格式如下：
```
<jsp:getProperty 
  name="Bean实例名" 
  property="propertyName"
/>
```

## 4.6 动作标识 <jsp：setProperty>
<jsp：setProperty>标识通常情况下与<jsp：useBean>标识一起使用，它调用Bean中的setXXX()方法将请求中的参数赋值给由<jsp：use-Bean>标识创建的JavaBean中对应的简单属性或索引属性。该标识的语法格式如下：

```
<jsp:setProperty
	name="Bean实例名" 
	{
		property="*" 
		|property="propertyName" 
		| property="propertyName" param="parameterName"
		| property="propertyName" value="值" 
	}
/>
```
---
![](http://upload-images.jianshu.io/upload_images/1124873-98f096df9a20e728.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
