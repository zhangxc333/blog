# springboot-crud-restful

## 一、spring initializr生成项目

1. 选择web、theymeleaf模块
2. 导入已有的静态资源（.html .js .css）和javabean、dao（不涉及数据库）。

## 二、springmvc自动与扩展配置

### 1、springmvc自动配置

```java
/**
 *
 * 1、springmvc auto-configuration自动配置
 *
 * @author ys
 * @date 2019/6/21
 */
首页跳转至 "/templates/" + index + ".html" 

方法一
@Controller
public class IndexController {

    @RequestMapping({"/", "/index.html"})
    public String index(){
        return "index";
    }
}

方法二
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
        registry.addViewController("/index.html").setViewName("index");
    }
}
  
```



### 2、springmvc扩展配置

```java
/**
 *
 * springboot推荐給容器中添加组件的方式：使用全注解的方式。
 * 1、@Configuration注解的配置类 --> spring配置文件
 * 2、@Bean注解的方法 --> 配置文件中的<bean></bean>标签
 *    将相应的组件返回到ioc容器中
 *
 * @author ys
 * @date 2019/6/21
 */
@Configuration
public class MyConfig{
    /**
     * 2、springmvc扩展配置，
     * 利用@Configuration，給容器中添加自定义的组件。
     */

    //所有的WebMvcConfigurer都会一起作用
    @Bean
    public WebMvcConfigurer webMvcConfigurer(){

        WebMvcConfigurer webMvcConfigurer = new WebMvcConfigurer(){
            //注册视图解析器
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("index");
                registry.addViewController("/index.html").setViewName("index");
                registry.addViewController("/main.html").setViewName("dashboard");
            }

            /**
             * 将自定义的拦截器添加到容器中，并且添加拦截所有路径"/**"
             * 除去"首页/index.html"、"首页/"、"登入页面/user/login"
             * @param registry
             */
            //注册拦截器
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                //同时要除去对静态资源的拦截
                registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**")
                        .excludePathPatterns("/", "/index.html", "/user/login")
                        .excludePathPatterns("/asserts/**");
            }
        };       
        return webMvcConfigurer;
    }


    /**
     *
     * 将返回值添加到ioc容器中。
     * 方法名localeResolver 就是这个组件默认的id。
     *
     * ===方法名应该命名为需要添加组件类的首字母小写（LocaleResolver --> localeResolver）===
     *
     * @return
     */
    @Bean
    public LocaleResolver localeResolver(){

        LocaleResolver localeResolver = new LocaleResolver() {

            @Override
            public Locale resolveLocale(HttpServletRequest httpServletRequest) {

                String l = httpServletRequest.getParameter("l");

                Locale locale = Locale.getDefault();

                if(!StringUtils.isEmpty(l)){
                    String[] split = l.split("_");
                    locale = new Locale(split[0], split[1]);
                }

                return locale;
            }

            @Override
            public void setLocale(HttpServletRequest httpServletRequest, @Nullable HttpServletResponse httpServletResponse, @Nullable Locale locale) {

            }

        };
        return localeResolver;
    }

}

```

## 三、index.html首页国际化的问题

### 1、根据浏览器设置默认的国际化

**编写国际化配置文件**

1. 在类目录下新建resource bundle文件，文件名为messages。类目录下的`messages.properties`，`messages_en_US.properties`，`messages_zh_CN.properties`文件，springboot能够自动识别。

```properties
如果不使用默认的配置，可以在全局配置文件中设置指定的文件

spring.messages.basename=i18n/login
```

2. 将抽取内容填写在国际化配置文件，并在index.html文件中取出。

```html
<h1 th:text="#{login.msg}">Please sign in</h1>
```

**拓展**：

```html
thymeleaf 五种基本表达式（simple expressions）

1. ${...}获取变量值；OGNL。（最常用）
	1）获取对象的属性、调用方法
	2）使用内置的基本对象
	3）使用内置的一些根据
2. *{...} 
3. #{...} 获取国际化信息
4. @{...} 定义URI
5. ~{...} 片段引用表达式
```



### 2、根据请求自定义国际化

自定义国际化组件（LocaleResolver），并返回给容器。

```java
@Configuration
public class MyConfig{
    
    @Bean
    public LocaleResolver localeResolver(){

        LocaleResolver localeResolver = new LocaleResolver() {

            @Override
            public Locale resolveLocale(HttpServletRequest httpServletRequest) {
                
                String l = httpServletRequest.getParameter("l");
                Locale locale = Locale.getDefault();
                
                if(!StringUtils.isEmpty(l)){
                    String[] split = l.split("_");
                    locale = new Locale(split[0], split[1]);
                }
                return locale;
            }

            @Override
            public void setLocale(HttpServletRequest httpServletRequest, @Nullable HttpServletResponse httpServletResponse, @Nullable Locale locale) {

            }

        };
        return localeResolver;
    }

}
```

## 四、首页登入

### 1、登入和主页跳转

==重定向：可以理解为重新发送请求，请求的内容为`redirect:`后面的url。==

```java
@Controller
public class LoginController {
	/**
	* 获取的参数根据业务逻辑添加。
	* @RequestParam（）参数用于表单数据的提交并验证 
	* Map<String, Object> map，用于封装特定的信息
	* 
	* HttpSession session，session添加属性，在拦截器中数据条件判断
	* ========另外session携带的信息，可以在以后的公共页面上使用。===========
	*/
    @PostMapping(value = "/user/login")
    public String loginController(@RequestParam("username") String username,
                                  @RequestParam("password") String password,
                                  Map<String, Object> map,
                                  HttpSession session){
        if(!StringUtils.isEmpty(username) && password.equals("f")){
            session.setAttribute("username",username);
            //登入成功后，为防止表单重复提交数据，最好的方法使用重定向。
            return "redirect:/main.html";
        }else {
            map.put("msg", "用户名密码错误");
            return "index";
        }
    }
}
```

```html
<!--//添加用户名密码错误信息-->
<p style="color:red;" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```

**注意**：

`return "redirect:/main.html";`和`return "index";`存在本质上的区别，springboot检测到`redirect:`就认定为重定向，会重新发送url请求；而`return "index";`是跳转到页面也就是`/templates/index.html`。

### 2.权限设置（拦截器）

新建自己的拦截器，并且添加到ioc容器中。

1、新建拦截器过程中，实现`HandlerInterceptor`，重写`preHandle()`方法。

​	`preHandle()`方法返回值为boolean类型，没法使用重定向的方法。在这里采用原生servlet方式的方法，即==request.getRequestDispatcher("/index.html").forward(request, response);==

```Java
/**
 * 添加登入拦截器,而后注入到容器中
 * @author ys
 * @date 2019/6/22
 */
public class LoginHandlerInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object username = request.getSession().getAttribute("username");
        if(username == null){
            //未登入
            //这里的“msg”与主页中的用户密码消息相同，不需要额外编写主页。
            request.setAttribute("msg", "没有登入权限");
            //原始的servlet方式
            //将请求发送到指定页面（结合重定向理解）
            request.getRequestDispatcher("/index.html").forward(request, response);
            return false;
        }else{

            return true;
        }

    }

}
```



2、添加到拦截器，拦截器也属于webMvcConfigurer下。

拦截器的添加过程中要注意三点：

* 1、将自定义的拦截器添加到容器中，`.addInterceptor(new LoginHandlerInterceptor())`
* 2、并且添加拦截所有路径。`.addPathPatterns("/**")`
* 3、除去"首页/index.html"、"首页/"、"登入页面/user/login"，`.excludePathPatterns("/", "/index.html", "/user/login")`
* 4、除去对静态资源的拦截(静态支援所在目录)，`.excludePathPatterns("/asserts/**");`

```java
@Configuration
public class MyConfig{

    //所有的WebMvcConfigurer都会一起作用
    @Bean
    public WebMvcConfigurer webMvcConfigurer(){


        WebMvcConfigurer webMvcConfigurer = new WebMvcConfigurer(){
            //注册视图解析器
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("index");
                registry.addViewController("/index.html").setViewName("index");
                registry.addViewController("/main.html").setViewName("dashboard");
            }

            /**
             * 1、将自定义的拦截器添加到容器中，
             * 2、并且添加拦截所有路径"/**"
             * 3、除去"首页/index.html"、"首页/"、"登入页面/user/login"
             * @param registry
             */
            //注册拦截器
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                //同时要除去对静态资源的拦截
                registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**")
                        .excludePathPatterns("/", "/index.html", "/user/login")
                        .excludePathPatterns("/asserts/**");
            }
        };
        return webMvcConfigurer;
    }
}
```



## 五、公共页面抽取

同一网站下页面之间有很多公共部分，可以采用抽取公共页面的思路。即可以简化单个页面的逻辑管理，便于编写特定的代码，也可以保证整个网站页面风格一致。

**thymeleaf公共页面抽取**

```html
1、抽取公共片段
<div th:fragment="copy">
</div>

2、引入公共片段
<div th:insert="~{footer :: copy}"></div>
~{templatename::selector}：模板名::选择器
~{templatename::fragmentname}:模板名::片段名

3、三种th属性
th:insert=""
th:replace=""
th:include=""
```

引入片段的时候传入参数，这里的传参应该理解为：==引用片段的过程中，将参数传入片段。==

```java

<div th:replace="~{common/bar::#nav_side(activeUri='main')}"></div>
```



## 六、restful-crud实验的架构

| 需求         | 实现方法    | 请求uri   | 请求方式 |
| ------------ | ----------- | --------- | -------- |
| 查询所有员工 | getEmps()   | /emps     | GET      |
| 新增员工     | saveEmp()   | /emp      | POST     |
| 查询某个员工 | getEmp()    | /emp/{id} | GET      |
| 修改员工     | updateEmp() | /emp/{id} | PUT      |
| 删除员工     | deleteEmp() | /emp/{id} | DELETE   |



## 七、查询员工



## 八、新增员工

**注意**：下拉列表提交的数据部门的id值`${dept.id}`，自动封装要想成功，下拉列表名应该为`th:name="department.id"`，即javabean对应属性的级联属性id。

```html
<select class="form-control" th:name="department.id">
	<option th:each="dept:${depts}" th:text="${dept.departmentName}" th:value="${dept.id}">1</option>
</select>
```



## 九、修改员工

**查询某位员工，并来到修改页面**。

1、在员工列表点击查询按钮，查询某位员工。按钮上的uri上添加id的拼接方式为==@{/emp/}+${emp.id}==

```html
<a class="btn btn-sm btn-primary" href="/emp/{id}" th:href="@{/emp/}+${emp.id}">编辑</a>
```

2、查询页面数据的拼接

与输入框直接利用`th:value=""`，单选框是`th:checked="${emp.gender}==1"`，下拉列表是`th:selected="${emp.department.id}==${dept.id}"`。本质上没有取别的，都是按照各自的语法赋值。

```html
<!--单选框 -->
<div class="form-group">
    <label>Gender</label><br/>
    <div class="form-check form-check-inline">
        <input class="form-check-input" type="radio" name="gender"  value="1" th:checked="${emp.gender}==1">
        <label class="form-check-label">男</label>
    </div>
    <div class="form-check form-check-inline">
        <input class="form-check-input" type="radio" name="gender"  value="0" th:checked="${emp.gender}==0">
        <label class="form-check-label">女</label>
    </div>
</div>

<!--下拉列表 -->
<div class="form-group">
    <label>department</label>
    <select class="form-control" th:name="department.id">
        <option th:each="dept:${depts}" th:text="${dept.departmentName}" th:value="${dept.id}"
                th:selected="${emp.department.id}==${dept.id}">1</option>
    </select>
</div>
```

 

## 十、删除员工

**点击删除员工按钮，弹出静态框，点击确定删除员工。**

### 1、引入问题

```html
<button class="btn btn-sm btn-danger" id="btn_delete">删除</button>
```

<button>按钮不在表单中，不可以通过表单发送uri请求；将<button>按钮修改为<a>超链接之后，虽说可以提交uri请求，但是默认的请求方式为GET，不满足删除员工使用DELETE方式的要求。==于是，我们采用js，单击事件的方式提交DELETE请求方式的表单。==

表单请求方式确认后，就需要确定请求uri。由于查询员工的时候，员工的id可以放在<button>按钮上，==所以我们在button按钮上自定义一个属性，用于存放了拼接了id的url。==

==该属性将从按钮，转移到静态框，然后赋值给form表单的action属性。提交form表单，进行删除==

```html
<button class="btn btn-sm btn-danger" id="btn_delete" th:attr="delete_uri=@{/emp/}+${emp.id}">删除</button>
```

**拓展：**

```markdown
thymeleaf添加属性的方式 
th:attr="delete_uri=@{/emp/}+${emp.id}"
```

### 2、容易出错的两个问题

==1、删除按钮不是在页面上原生编写的，而是根据循环拼接出来，单击事件的调用不能直接使用`.click`，而是该使用`$(document).on("click", "#id", function(){});`==

```html
<!-- 使用on绑定事件 -->
<script>
$(document).on("click", "#id", function(){
	//stuff
});
</script>
```

==2、在thymeleaf模板下，对于表单提交的`put`、`delete`请求，利用`th:method=""`能够直接使用（默认只支持`get`、`post`）.==

```html
如下所示的代码，在编译的时候
<form action="/emp" th:method="delete"></form>
会自动生成===============>
<form action="/emp" method="post">
    <input type="hidden" name="_method" value="delete">
</form>
```





