1. pom

    ~~~xml
    <dependency>
    	<groupId>org.springframework.session</groupId>
    	<artifactId>spring-session-data-redis</artifactId>
    </dependency>
    ~~~

2. 配置类

    ~~~java
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
    import org.springframework.data.redis.serializer.RedisSerializer;
    
    @Configuration
    public class SessionConfig{
    
        @Bean("springSessionDefaultRedisSerializer")
        public RedisSerializer<Object> defaultRedisSerializer(){
            return valueSerializer();
        }
    
        private RedisSerializer<Object> valueSerializer(){
            return new Jackson2JsonRedisSerializer(Object.class);
        }
    }
    
    
    
    
    @EnableRedisHttpSession
    @SpringBootApplication
    public class DubbocliApplication{
    	public static void main(String[]args){
    		SpringApplication.run(DubbocliApplication.class,args);
    	}
    }
    ~~~

3. redis配置

    ~~~
    ##redis配置
    spring.redis.sentinel.master=T1
    spring.redis.sentinel.nodes=47.101.155.121:16379,47.101.155.121:16380,47.101.155.121:16381
    spring.redis.password=123456
    spring.redis.pool.max-idle=8
    spring.redis.pool.min-idle=0
    spring.redis.pool.max-active=8
    spring.redis.pool.max-wait=-1
    #超时一定要大于0
    spring.redis.timeout=3000
    spring.session.store-type=redis
    ~~~

    

测试

~~~java
@RequestMapping(value="/session")
public Map<String,Object> getSession(HttpServletRequest request){
	request.getSession().setAttribute("username","admin");
	Map<String,Object> map = new HashMap<String,Object>();
	map.put("sessionId",request.getSession().getId());
	return map;
}

@RequestMapping(value="/get")
public String get(HttpServletRequest request){
	String userName=(String)request.getSession().getAttribute("username");
	return userName;
}

~~~

