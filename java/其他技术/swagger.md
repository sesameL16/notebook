~~~xml
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.9.2</version>
</dependency>
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.9.2</version>
</dependency>
~~~



**配置类**

~~~java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * describe:swagger配置类
 *
 * @author lqs
 * @date 2019/05/22
 */
@EnableSwagger2
@Configuration
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                //是否开启swagger
                .enable(true)
                //忽略指定类型的参数
                .ignoredParameterTypes()
                .select()
                //@ApiIgnore 这样,该接口就不会暴露在 swagger2 的页面下
                //这里采用包含注解的方式来确定要显示的接口
            //.apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                //这里采用包扫描的方式来确定要显示的接口        		   	
            .apis(RequestHandlerSelectors.basePackage("cn.bejwt.youth.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title("青春风华网站项目")
                //创建人
                .contact(new Contact("lqs","http://www.bejwt.cn",null))
                //版本号
                .version("1.0.0")
                //描述
                .description("本接口文档描述青春风华网站的APP接口服务的对外接口")
                .build();
    }
}

~~~



Spring Boot使用过Shiro或Spring Security框架，需要将相应的URL访问权限放开，以Shiro为例，添加匿名访问过滤器

~~~java
filterChainDefinitionMap.put("/api/v1/**", "anon"); //API接口

// swagger接口文档
filterChainDefinitionMap.put("/v2/api-docs", "anon");
filterChainDefinitionMap.put("/webjars/**", "anon");
filterChainDefinitionMap.put("/swagger-resources/**", "anon");
filterChainDefinitionMap.put("/swagger-ui.html", "anon");
~~~



**Swagger使用的注解及其说明**

[@Api](https://my.oschina.net/u/2396174)：用在类上，说明该类的作用。

@ApiOperation：注解来给API增加方法说明。

@ApiImplicitParams : 用在方法上包含一组参数说明。

@ApiImplicitParam：用来注解来给方法入参增加说明。

@ApiResponses：用于表示一组响应

@ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息

  l  **code**：数字，例如400

  l  **message**：信息，例如"请求参数没填好"

  l  **response**：抛出异常的类  

@ApiModel：描述一个Model的信息（一般用在请求参数无法使用@ApiImplicitParam注解进行描述的时候）

  l  **@ApiModelProperty**：描述一个model的属性

 

注意：@ApiImplicitParam的参数说明：

| **paramType**：指定参数放在哪个地方 | header：请求参数放置于Request Header，使用@RequestHeader获取  query：请求参数放置于请求地址，使用@RequestParam获取  path：（用于restful接口）-->请求参数的获取：@PathVariable  body：（不常用）  form（不常用） |
| ----------------------------------- | ------------------------------------------------------------ |
| name：参数名                        |                                                              |
| dataType：参数类型                  |                                                              |
| required：参数是否必须传            | true \| false                                                |
| value：说明参数的意思               |                                                              |
| defaultValue：参数的默认值          |                                                              |



~~~java
import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
 
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
 
/**
 * 一个用来测试swagger注解的控制器
 * 注意@ApiImplicitParam的使用会影响程序运行，如果使用不当可能造成控制器收不到消息
 * 
 * @author SUNF
 */
@Controller
@RequestMapping("/say")
@Api(value = "SayController|一个用来测试swagger注解的控制器")
public class SayController {
    
    @ResponseBody
    @RequestMapping(value ="/getUserName", method= RequestMethod.GET)
    @ApiOperation(value="根据用户编号获取用户姓名", notes="test: 仅1和2有正确返回")
    @ApiImplicitParam(paramType="query", name = "userNumber", value = "用户编号", required = true, dataType = "Integer")
    public String getUserName(@RequestParam Integer userNumber){
        if(userNumber == 1){
            return "张三丰";
        }
        else if(userNumber == 2){
            return "慕容复";
        }
        else{
            return "未知";
        }
    }
    
    @ResponseBody
    @RequestMapping("/updatePassword")
    @ApiOperation(value="修改用户密码", notes="根据用户id修改密码")
    @ApiImplicitParams({
        @ApiImplicitParam(paramType="query", name = "userId", value = "用户ID", required = true, dataType = "Integer"),
        @ApiImplicitParam(paramType="query", name = "password", value = "旧密码", required = true, dataType = "String"),
        @ApiImplicitParam(paramType="query", name = "newPassword", value = "新密码", required = true, dataType = "String")
    })
    public String updatePassword(@RequestParam(value="userId") Integer userId, @RequestParam(value="password") String password, 
            @RequestParam(value="newPassword") String newPassword){
      if(userId <= 0 || userId > 2){
          return "未知的用户";
      }
      if(StringUtils.isEmpty(password) || StringUtils.isEmpty(newPassword)){
          return "密码不能为空";
      }
      if(password.equals(newPassword)){
          return "新旧密码不能相同";
      }
      return "密码修改成功!";
    }
}

~~~

