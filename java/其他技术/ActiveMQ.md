~~~xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
<dependency>
	<groupId>org.apache.activemq</groupId>
	<artifactId>activemq-pool</artifactId>
</dependency>
~~~



~~~
##activemq配置
spring.activemq.broker-url=tcp://47.101.155.121:61616
#true表示使用连接池
spring.activemq.pool.enabled=true
#连接池最大连接数
spring.activemq.pool.max-connections=5
#空闲的连接过期时间，默认为30秒
spring.activemq.pool.idle-timeout=30000
#强制的连接过期时间，与idleTimeout的区别在于：idleTimeout是在连接空闲一段时间失效，而expiryTimeout不管当前连接的情况，只要达到指定时间就失效。默认为0，never
spring.activemq.pool.expiry-timeout=0
~~~



**生产者**

~~~java
//配置
import org.apache.activemq.command.ActiveMQQueue;
import org.apache.activemq.command.ActiveMQTopic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.jms.Destination;

@Configuration
public class ActivemqConfig{

    @Bean
    public Destination queueTest(){
        return new ActiveMQQueue("queue.test");
    }

    @Bean
    public Destination topicTest(){
        return new ActiveMQTopic("topic.test");
    }
}
~~~



~~~java
import com.sesame.dubbocli.util.JsonResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.jms.Destination;
import java.util.HashMap;
import java.util.Map;

@RestController
public class TestProducer{
    @Autowired
    private JmsMessagingTemplate JmsMessagingTemplate;
    @Autowired
    private Destination queueTest;
    @Autowired
    private Destination topicTest;

    @RequestMapping("sendMsg")
    public JsonResult sendMsg(String msg){
        JmsMessagingTemplate.convertAndSend(this.queueTest,msg);
        System.out.println("消息发送成功:"+msg);
        return new JsonResult(msg);
    }

    @RequestMapping("/sendMap")
    public void sendMap(){
        Map map=new HashMap();
        map.put("mobile","13888888888");
        map.put("content","王总喜提兰博基尼");
        JmsMessagingTemplate.convertAndSend(this.topicTest,map);
        System.out.println("map发送成功");
    }
}

~~~



**消费者**

~~~java
//配置
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.config.DefaultJmsListenerContainerFactory;
import org.springframework.jms.config.JmsListenerContainerFactory;
import javax.jms.ConnectionFactory;

@Configuration
public class ActivemqConfig{

    /**
	 *JmsListener注解默认只接收queue消息,如果要接收topic消息,需要设置containerFactory
	 */
    @Bean
    public JmsListenerContainerFactory<?> topicListenerContainer(
        						ConnectionFactory activeMQConnectionFactory){
        DefaultJmsListenerContainerFactory topicListenerContainer = 
            new DefaultJmsListenerContainerFactory();
        topicListenerContainer.setPubSubDomain(true);
        topicListenerContainer.setConnectionFactory(activeMQConnectionFactory);
        return topicListenerContainer;
    }
}
~~~



~~~java
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;
import java.util.Map;

@Component
public class TestConsumer{

    @JmsListener(destination="queue.test")
    public void readMsg(String text){
        System.out.println("接收到的消息："+text);
    }

    @JmsListener(destination="topic.test",containerFactory="topicListenerContainer")
    public void readMap(Map map){
        System.out.println("1号topic接受消息"+map);
    }

    @JmsListener(destination="topic.test",containerFactory="topicListenerContainer")
    public void readMap2(Map map){
        System.out.println("2号topic接受消息"+map);
    }
}

~~~

