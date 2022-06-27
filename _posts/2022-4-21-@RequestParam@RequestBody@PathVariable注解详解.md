### [@RequestParam](https://github.com/RequestParam) [@RequestBody](https://github.com/RequestBody) [@PathVariable](https://github.com/PathVariable) 等参数绑定注解详解

#### 1、 [@PathVariable](https://github.com/PathVariable)

当使用[@RequestMapping](https://github.com/RequestMapping) URI template 样式映射时， 即 someUrl/{paramId}, 这时的paramId可通过 [@Pathvariable](https://github.com/Pathvariable)注解绑定它传过来的值到方法的参数上。

示例代码：

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class test {
  @RequestMapping("/pets/{petId}")
  public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
    // implementation omitted
  }
}
```

上面代码把URI template 中变量 ownerId的值和petId的值，绑定到方法的参数上。若方法参数名称和需要绑定的uri template中变量名称不一致，需要在[@PathVariable](https://github.com/PathVariable)(“name”)指定uri template中的名称。

#### 2、[@RequestBody](https://github.com/RequestBody)正确用法

1、[@requestBody](https://github.com/requestBody)注解常用来处理content-type不是默认的application/x-www-form-urlcoded编码的内容，比如说：application/json或者是application/xml等。一般情况下来说常用其来处理application/json类型。[@RequestBody](https://github.com/RequestBody)接受的是一个json格式的字符串，一定是一个字符串。

2、通过[@requestBody](https://github.com/requestBody)可以将请求体中的JSON字符串绑定到相应的bean上，当然，也可以将其分别绑定到对应的字符串上。

例如：

```java
$.ajax({
       url:"/login",
       type:"POST",
       data:'{"userName":"admin","pwd","admin123"}',
       content-type:"application/json charset=utf-8",
       success:function(data){
               alert("request success ! ");
　　      }
});
```

```java
@requestMapping("/login")
public void login(@requestBody String userName,@requestBody String pwd){
　　System.out.println(userName+" ："+pwd);
}
```

这种情况是将JSON字符串中的两个变量的值分别赋予了两个字符串，但是呢假如我有一个User类，拥有如下字段：

- String userName;
- String pwd;

那么上述参数可以改为以下形式：[@requestBody](https://github.com/requestBody) User user 这种形式会将JSON字符串中的值赋予user中对应的属性上。**需要注意的是，JSON字符串中的key必须对应user中的属性名，否则是请求不过去的**。

```java
@requestMapping("/login")
public void login(@requestBody User user){
　　System.out.println(user.getUserName()+" ："+user.getPwd());
}
```

#### 3、[@RequestParam](https://github.com/RequestParam)

RequestParam可以接受简单类型的属性，也可以接受对象类型。

[@RequestParam](https://github.com/RequestParam)有三个配置参数：

- required 表示是否必须，默认为 true，必须。
- defaultValue 可设置请求参数的默认值。
- value 为接收url的参数名（相当于key值）。

```java
package com.day01springmvc.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.ModelAndView;
@Controller
@RequestMapping("hello")
public class HelloController {
    /**
     * 接收普通请求参数
     * http://localhost:8080/hello/show16?name=linuxsir
     * url参数中的name必须要和@RequestParam("name")一致
     * @return
     */
    @RequestMapping("show16")
    public ModelAndView test16(@RequestParam("name")String name){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello2");
        mv.addObject("msg", "接收普通的请求参数：" + name);
        return mv;
    }
    /**
     * 接收普通请求参数
     * http://localhost:8080/hello/show17
     * url中没有name参数不会报错、有就显示出来
     * @return
     */
    @RequestMapping("show17")
    public ModelAndView test17(@RequestParam(value="name",required=false)String name){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello2");
        mv.addObject("msg", "接收普通请求参数：" + name);
        return mv;
    }
    /**
     * 接收普通请求参数
     * http://localhost:8080/hello/show18?name=998 显示为998
     * http://localhost:8080/hello/show18?name 显示为hello
     * @return
     */
    @RequestMapping("show18")
    public ModelAndView test18(@RequestParam(value="name",required=true,defaultValue="hello")String name){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello2");
        mv.addObject("msg", "接收普通请求参数：" + name);
        return mv;
    }
}
/*
    输出结果：
    show16------接收普通请求参数:linuxsir
    show17------接收普通请求参数:null
    show18------接收普通请求参数:hello
*/
```

#### 4、[@Param](https://github.com/Param)

[@Param](https://github.com/Param)是mybatis中的注解，用注解来简化xml配置的时候,[@Param](https://github.com/Param)注解的作用是给参数命名,参数命名后就能根据名字得到参数值,正确的将参数传入sql语句中 。请看下面的示例：

```java
public interface Mapper {
@Select("select s_id id,s_name name,class_id classid from student where  s_name= #{aaaa} and class_id = #{bbbb}") 
    
public Student select(@Param("aaaa") String name,@Param("bbbb")int class_id);  
    
@Delete...... 
@Insert...... 
}
```



[本文链接：https://www.kuangstudy.com/bbs/1374942320650121217](https://www.kuangstudy.com/bbs/1374942320650121217)