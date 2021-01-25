**简单的定时任务用 spring的 @Scheduled 注解即可**

~~~java
@Component
public class ScheduledTask {
    
    @Scheduled(fixedRate = 5000)
    //表示每隔5000ms，Spring scheduling会调用一次该方法，不论该方法的执行时间是多少
    public void reportCurrentTime() throws InterruptedException {
        System.out.println(new Date()));
    }

    @Scheduled(fixedDelay = 5000)
    //表示当方法执行完毕5000ms后，Spring scheduling会再次调用该方法
    public void reportCurrentTimeAfterSleep() throws InterruptedException {
        System.out.println(new Date()));
    }

    @Scheduled(cron = "0 0 1 * * *")
    //提供了一种通用的定时任务表达式，这里表示每隔5秒执行一次，更加详细的信息可以参考cron表达式。
    public void reportCurrentTimeCron() throws InterruptedException {
        System.out.println(new Date()));
    }
}


@SpringBootApplication
@EnableScheduling
//告诉Spring创建一个task executor，如果我们没有这个标注，所有@Scheduled标注都不会执行
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
~~~

**cron表达式**

至少6个（也可能7个）按顺序依次为

秒（0~59）

分钟（0~59）

小时（0~23）

天（月）（0~31，但是你需要考虑你月的天数）

月（0~11）

天（星期）（1~7 1=SUN 或 SUN，MON，TUE，WED，THU，FRI，SAT）

年份（1970－2099）

~~~
0 0/30 9-17 * * ?        朝九晚五工作时间内每半小时
0 0 10,14,16 * * ?       每天上午10点，下午2点，4点
0 15 10 L * ?            每月最后一日的上午10:15触发
0 15 10 ? * 6L 2002-2005 2002年至2005年的每月的最后一个星期五上午10:15触发 
~~~





**Quartz 实现动态设置定时任务**

~~~xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
~~~



~~~java
import org.quartz.Scheduler;
import org.quartz.spi.TriggerFiredBundle;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.config.AutowireCapableBeanFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.quartz.AdaptableJobFactory;
import org.springframework.scheduling.quartz.SchedulerFactoryBean;
import org.springframework.stereotype.Component;

/**
*describe:Quartz配置类
*
*@author lqs
*@date 2019/03/20
*/
@Configuration
public class QuartzConfiguration{

	//解决Job中注入SpringBean为null的问题
	@Component("quartzJobFactory")
	private class QuartzJobFactory extends AdaptableJobFactory{
		//这个对象Spring会帮我们自动注入进来,也属于Spring技术范畴.
		@Autowired
		private AutowireCapableBeanFactory capableBeanFactory;
		
		@Override
		protected Object createJobInstance(TriggerFiredBundle bundle)throws Exception{
			//调用父类的方法
			Object jobInstance = super.createJobInstance(bundle);
			//进行注入,这属于Spring的技术,不清楚的可以查看Spring的API.
			capableBeanFactory.autowireBean(jobInstance);
			return jobInstance;
		}
	}
	
	//注入scheduler到spring，在下面quartzManege会用到
	@Bean(name="scheduler")
	public Scheduler scheduler(QuartzJobFactory quartzJobFactory)throws Exception{
		SchedulerFactoryBean factoryBean = new SchedulerFactoryBean();
		factoryBean.setJobFactory(quartzJobFactory);
		factoryBean.afterPropertiesSet();
		Scheduler scheduler = factoryBean.getScheduler();
		scheduler.start();
		return scheduler;
	}

}

~~~



**定时任务管理器**

~~~java
import cn.newcapec.city.smart.modular.system.model.TimingTask;
import lombok.extern.slf4j.Slf4j;
import org.quartz.*;
import org.springframework.stereotype.Component;
import javax.annotation.Resource;
import static org.quartz.JobBuilder.newJob;
import static org.quartz.TriggerBuilder.newTrigger;

/**
*describe:定时任务管理器
*
*@authorlqs
*@date2019/03/20
*/
@Component
@Slf4j
public class QuartzManage{

	@Resource(name="scheduler")
	private Scheduler scheduler;
	
	public void addJob(TimingTask job) throws SchedulerException,ClassNotFoundException,IllegalAccessException,InstantiationException{
		//通过类名获取实体类，即要执行的定时任务的类
		Class<?> clazz = Class.forName(job.getCallTarget());
		Job jobEntity = (Job)clazz.newInstance();
		//通过实体类和任务名创建JobDetail
		JobDetail jobDetail = newJob(jobEntity.getClass())
			.withIdentity(job.getCallTarget()).build();
		//通过触发器名和cron表达式创建Trigger
		Trigger cronTrigger = new Trigger()
			.withIdentity(job.getCallTarget())
			.startNow()
			.withSchedule(CronScheduleBuilder.cronSchedule(job.getCronExpress()))
			.build();
		//执行定时任务
		scheduler.scheduleJob(jobDetail,cronTrigger);
	}
	
	/**
	*更新jobcron表达式
	*@param quartzJob
	*@throws SchedulerException
	*/
	public void updateJobCron(TimingTask quartzJob) throws SchedulerException{
		TriggerKey triggerKey = TriggerKey.triggerKey(quartzJob.getCallTarget());
		CronTriggert rigger = (CronTrigger)scheduler.getTrigger(triggerKey);
		CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(quartzJob.getCronExpress());
		trigger = trigger.getTriggerBuilder().withIdentity(triggerKey).withSchedule(scheduleBuilder).build();
		scheduler.rescheduleJob(triggerKey,trigger);
	}
	
	/**
	*删除一个job
	*@param quartzJob
	*@throws SchedulerException
	*/
	public void deleteJob(TimingTask quartzJob) throws SchedulerException{
		JobKey jobKey = JobKey.jobKey(quartzJob.getCallTarget());
		scheduler.deleteJob(jobKey);
	}
	
	/**
	*恢复一个job
	*@param quartzJob
	*@throws SchedulerException
	*/
	public void resumeJob(TimingTask quartzJob)throws SchedulerException{
		JobKey jobKey = JobKey.jobKey(quartzJob.getCallTarget());
		scheduler.resumeJob(jobKey);
	}
	
	/**
	*立即执行job
	*@param quartzJob
	*@throws SchedulerException
	*/
	public void runAJobNow(TimingTask quartzJob) throws SchedulerException{
		JobKey jobKey=JobKey.jobKey(quartzJob.getCallTarget());
		scheduler.triggerJob(jobKey);
	}
	
	/**
	*暂停一个job
	*@param quartzJob
	*@throws SchedulerException
	*/
	public void pauseJob(TimingTask quartzJob) throws SchedulerException{
		JobKey jobKey=JobKey.jobKey(quartzJob.getCallTarget());
		scheduler.pauseJob(jobKey);
	}
}

~~~

**任务状态枚举**

~~~java
/**
*任务接口
*Created by es on2018/8/28.
*/
public interface IJob{

	//任务状态
	enum State{
		//启用
		ENABLE(0),
		//禁用
		DISABLE(1);
		
		private int state;
		
		State(int state){
			this.state = state;
		}

		public int getState(){
			return state;
		}
	}
}

~~~



**项目启动时重新激活启用的定时任务**

~~~java
import cn.newcapec.city.smart.modular.system.model.TimingTask;
import cn.newcapec.city.smart.modular.system.service.ITimingTaskService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.apache.commons.collections.CollectionUtils;
import java.util.List;

/**
*describe:项目启动时重新激活启用的定时任务
*
*@author lqs
*@date 2019/03/20
*/
@Component
@Slf4j
public class MyApplicationRunner implements ApplicationRunner{
	
	@Autowired
	private ITimingTaskService timingTaskService;

	/**
	*//项目启动时重新激活启用的定时任务
	*@param applicationArguments
	*@throws Exception
	*/
	@Override
	public void run(ApplicationArguments applicationArguments)throws Exception{
		TimingTask timingTask = new TimingTask();
		timingTask.setDelFlag(0);
		timingTask.setStatus(0);
		List<TimingTask> scheduleJobList = timingTaskService.selectByTimingTask();
		log.info("定时任务开启个数"+scheduleJobList.size());
		if(CollectionUtils.isNotEmpty(scheduleJobList)){
			timingTaskService.initTimingTask(scheduleJobList);
		}
	}
}

~~~



**测试**

~~~java
import lombok.extern.slf4j.Slf4j;
import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import java.util.Date;

/**
*describe:测试任务
*
*@authorlqs
*@date2019/03/21
*/
@Slf4j
public class TaskTest1 implements Job{

	public void run(){
		log.info("测试定时任务1##################"+(newDate()));
	}

	@Override
	public void execute(JobExecutionContext jobExecutionContext)throws JobExecutionException{
		run();
	}
}

~~~

