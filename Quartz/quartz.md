## Quartz学习

### 1.Quartz CronTrigger 最完整配置说明
CronTrigger配置格式:

格式: [秒] [分] [小时] [日] [月] [周] [年]


|序号|说明 |是否必填|允许填写的值|允许的通配符 |
|:--:|:--:|:--:|:--:|:--:|
|1	| 秒	 |是	 |0-59 	|  , - * /|
|2	| 分	 |是	 |0-59 	|  , - * /|
|3	|小时	 |是	 |0-23	|  , - * /|
|4	| 日	 |是	 |1-31	| , - * ? / L W|
|5	|月	 |是	 |1-12 or JAN-DEC	|  , - * /|
|6	| 周	| 是	| 1-7 or SUN-SAT	|  , - * ? / L #|
|7	| 年	 |否	 |empty 或 1970-2099	| , - * /|

通配符说明:

\* 表示所有值. 例如:在分的字段上设置 "*",表示每一分钟都会触发。

? 表示不指定值。使用的场景为不需要关心当前设置这个字段的值。例如:要在每月的10号触发一个操作，但不关心是周几，所以需要周位置的那个字段设置为"?" 具体设置为 0 0 0 10 * ?

\- 表示区间。例如 在小时上设置 "10-12",表示 10,11,12点都会触发。

, 表示指定多个值，例如在周字段上设置 "MON,WED,FRI" 表示周一，周三和周五触发

/ 用于递增触发。如在秒上面设置"5/15" 表示从5秒开始，每增15秒触发(5,20,35,50)。 在月字段上设置'1/3'所示每月1号开始，每隔三天触发一次。

L 表示最后的意思。在日字段设置上，表示当月的最后一天(依据当前月份，如果是二月还会依据是否是润年[leap]), 在周字段上表示星期六，相当于"7"或"SAT"。如果在"L"前加上数字，则表示该数据的最后一个。例如在周字段上设置"6L"这样的格式,则表示“本月最后一个星期五" 

W 表示离指定日期的最近那个工作日(周一至周五). 例如在日字段上设置"15W"，表示离每月15号最近的那个工作日触发。如果15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发.如果15号正好在工作日(周一至周五)，则就在该天触发。如果指定格式为 "1W",它则表示每月1号往后最近的工作日触发。如果1号正是周六，则将在3号下周一触发。(注，"W"前只能设置具体的数字,不允许区间"-").

\# 序号(表示每月的第几个周几)，例如在周字段上设置"6#3"表示在每月的第三个周六.注意如果指定"#5",正好第五周没有周六，则不会触发该配置(用在母亲节和父亲节再合适不过了)


<br/>

### Quartz Scheduler 任务参数与任务状态

@DisallowConcurrentExecution  (简单来说:不允许任务还没结束,新开线程执行任务)

此标记用在实现Job的类上面,意思是不允许并发执行,按照我之前的理解是 不允许调度框架在同一时刻调用Job类，后来经过测试发现并不是这样，而是Job(任务)的执行时间[比如需要10秒]大于任务的时间间隔[Interval（5秒)],那么默认情况下,调度框架为了能让 任务按照我们预定的时间间隔执行,会马上启用新的线程执行任务。否则的话会等待任务执行完毕以后 再重新执行！（这样会导致任务的执行不是按照我们预先定义的时间间隔执行）

测试代码，这是官方提供的例子。设定的时间间隔为3秒,但job执行时间是5秒,设置@DisallowConcurrentExecution以后程序会**等任务执行完毕以后再去执行,否则会在3秒时再启用新的线程执行**

org.quartz.threadPool.threadCount = 5 这里配置框架的线程池中线程的数量,要多配置几个,否则@DisallowConcurrentExecution不起作用
```xml
org.quartz.scheduler.instanceName = MyScheduler
org.quartz.threadPool.threadCount = 5
org.quartz.jobStore.class =org.quartz.simpl.RAMJobStore

```

```java
@PersistJobDataAfterExecution
@DisallowConcurrentExecution
public class StatefulDumbJob implements Job {

    /*
     * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     * 
     * Constants.
     * 
     * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     */

    public static final String NUM_EXECUTIONS = "NumExecutions";

    public static final String EXECUTION_DELAY = "ExecutionDelay";

    /*
     * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     * 
     * Constructors.
     * 
     * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     */

    public StatefulDumbJob() {
    }

    /*
     * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     * 
     * Interface.
     * 
     * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     */

    /**
     * <p>
     * Called by the <code>{@link org.quartz.Scheduler}</code> when a <code>{@link org.quartz.Trigger}</code>
     * fires that is associated with the <code>Job</code>.
     * </p>
     * 
     * @throws JobExecutionException
     *           if there is an exception while executing the job.
     */
    public void execute(JobExecutionContext context)
        throws JobExecutionException {
        System.err.println("---" + context.getJobDetail().getKey()
                + " executing.[" + new Date() + "]");

        JobDataMap map = context.getJobDetail().getJobDataMap();

        int executeCount = 0;
        if (map.containsKey(NUM_EXECUTIONS)) {
            executeCount = map.getInt(NUM_EXECUTIONS);
        }

        executeCount++;

        map.put(NUM_EXECUTIONS, executeCount);

        long delay = 5000l;
        if (map.containsKey(EXECUTION_DELAY)) {
            delay = map.getLong(EXECUTION_DELAY);
        }

        try {
            Thread.sleep(delay);
        } catch (Exception ignore) {
        }

        System.err.println("  -" + context.getJobDetail().getKey()
                + " complete (" + executeCount + ").");

    }

}
```


```java

public class MisfireExample {

    
    public void run() throws Exception {
        Logger log = LoggerFactory.getLogger(MisfireExample.class);

        log.info("------- Initializing -------------------");

        // First we must get a reference to a scheduler
        SchedulerFactory sf = new StdSchedulerFactory();
        Scheduler sched = sf.getScheduler();

        log.info("------- Initialization Complete -----------");

        log.info("------- Scheduling Jobs -----------");

        // jobs can be scheduled before start() has been called

        // get a "nice round" time a few seconds in the future...
        Date startTime = nextGivenSecondDate(null, 15);

        // statefulJob1 will run every three seconds
        // (but it will delay for ten seconds)
        JobDetail job = newJob(StatefulDumbJob.class)
            .withIdentity("statefulJob1", "group1")
            .usingJobData(StatefulDumbJob.EXECUTION_DELAY, 10000L)
            .build();
    
        SimpleTrigger trigger = newTrigger() 
            .withIdentity("trigger1", "group1")
            .startAt(startTime)
            .withSchedule(simpleSchedule()
                    .withIntervalInSeconds(3)
                    .repeatForever())
            .build();
        
        Date ft = sched.scheduleJob(job, trigger);
        log.info(job.getKey() +
                " will run at: " + ft +  
                " and repeat: " + trigger.getRepeatCount() + 
                " times, every " + trigger.getRepeatInterval() / 1000 + " seconds");

        log.info("------- Starting Scheduler ----------------");

        // jobs don't start firing until start() has been called...
        sched.start();

        log.info("------- Started Scheduler -----------------");
        
        try {
            // sleep for ten minutes for triggers to file....
            Thread.sleep(600L * 1000L); 
        } catch (Exception e) {
        }

        log.info("------- Shutting Down ---------------------");

        sched.shutdown(true);

        log.info("------- Shutdown Complete -----------------");

        SchedulerMetaData metaData = sched.getMetaData();
        log.info("Executed " + metaData.getNumberOfJobsExecuted() + " jobs.");
    }



    public static void main(String[] args) throws Exception {

        MisfireExample example = new MisfireExample();
        example.run();
    }

}
```


@PersistJobDataAfterExecution 

 此标记说明在执行完Job的execution方法后保存JobDataMap当中固定数据,在默认情况下 也就是没有设置 @PersistJobDataAfterExecution的时候 每个job都拥有独立JobDataMap

 否则改任务在重复执行的时候具有相同的JobDataMap
```java
@PersistJobDataAfterExecution
@DisallowConcurrentExecution
public class BadJob1 implements Job {

    public BadJob1() {
    }

    public void execute(JobExecutionContext context)
        throws JobExecutionException {
        JobKey jobKey = context.getJobDetail().getKey();
        JobDataMap dataMap = context.getJobDetail().getJobDataMap();
        
        int denominator = dataMap.getInt("denominator");
        System.out.println("---" + jobKey + " executing at " + new Date() + " with denominator " + denominator);

        denominator++;
        dataMap.put("denominator", denominator);
    }

}
```

```java
public class JobExceptionExample {

    public void run() throws Exception {

        // First we must get a reference to a scheduler
        SchedulerFactory sf = new StdSchedulerFactory();
        Scheduler sched = sf.getScheduler();

        // jobs can be scheduled before start() has been called

        // get a "nice round" time a few seconds in the future...
        Date startTime = nextGivenSecondDate(null, 2);

        JobDetail job = newJob(BadJob1.class)
            .withIdentity("badJob1", "group1")
            .usingJobData("denominator", "0")
            .build();
        
        SimpleTrigger trigger = newTrigger() 
            .withIdentity("trigger1", "group1")
            .startAt(startTime)
            .withSchedule(simpleSchedule()
                    .withIntervalInSeconds(2)
                    .repeatForever())
            .build();

        Date ft = sched.scheduleJob(job, trigger);
        
        //任务每2秒执行一次 那么在BadJob1的方法中拿到的JobDataMap的数据是共享的.
        //这里要注意一个情况： 就是JobDataMap的数据共享只针对一个BadJob1任务。
        //如果在下面在新增加一个任务 那么他们之间是不共享的 比如下面
        
        JobDetail job2 = newJob(BadJob1.class)
                .withIdentity("badJob1", "group1")
                .usingJobData("denominator", "0")
                .build();
        
        SimpleTrigger trigger2 = newTrigger() 
                .withIdentity("trigger1", "group1")
                .startAt(startTime)
                .withSchedule(simpleSchedule()
                        .withIntervalInSeconds(2)
                        .repeatForever())
                .build();
        
        //这个job2与job执行的JobDataMap不共享
        sched.scheduleJob(job2, trigger2);
        
        sched.start();

        try {
            // sleep for 30 seconds
            Thread.sleep(30L * 1000L);
        } catch (Exception e) {
        }

        sched.shutdown(false);
    }

    public static void main(String[] args) throws Exception {

        JobExceptionExample example = new JobExceptionExample();
        example.run();
    }

}
```

requestRecovery的意思是当任务在执行过程中出现意外 比如服务器down了 那么在重启时候是否恢复任务

```java
JobDetail job = newJob(HelloJob.class)
	.withIdentity("job1", "group1")
	.storeDurably() 
	.requestRecovery()
	.build();
```

<br/>

### Quartz Scheduler当任务中出现异常时的处理策略(JobExecutionExceptions)

问题1 如果任务执行发生错误了怎么办!
Quartz提供了二种解决方法:
- 1 立即重新执行任务
- 2 立即停止所有相关这个任务的触发器

问题2 怎么去执行呢?
Quartz的解决方式是:在你的程序出错时,用Quartz提供的JobExecutionException类相关方法很好的解决
一、立即重新执行该任务

当任务中出现异常时，我们捕获它，然后转换为JobExecutionExceptions异常抛出，同时可以控制调度引擎立即重新执行这个任务。

```java
try {
		int zero = 0;
		int calculation = 4815 / zero;
	} 
	catch (Exception e) {
		_log.info("--- Error in job!");
		JobExecutionException e2 = 
			new JobExecutionException(e);
		// this job will refire immediately
		e2.refireImmediately();
		throw e2;
	}
```

二、取消所有与这个任务关联的触发器

```java
try {
	int zero = 0;
	int calculation = 4815 / zero;
} 
catch (Exception e) {
	_log.info("--- Error in job!");
	JobExecutionException e2 = 
		new JobExecutionException(e);
	// Quartz will automatically unschedule
	// all triggers associated with this job
	// so that it does not run again
	e2.setUnscheduleAllTriggers(true);
	throw e2;
}
```

<br/>

### Quartz Scheduler与Spring集成(一) 基础配置与常见问题
https://www.cnblogs.com/daxin/archive/2013/05/29/3107178.html

常用操作代码:
http://www.quartz-scheduler.org/documentation/quartz-2.3.0/cookbook/MultipleSchedulers.html

<br/>

### Quartz How-To:Defining a Job (with input data)
Job:
```java
public class PrintPropsJob implements Job {

	public PrintPropsJob() {
		// Instances of Job must have a public no-argument constructor.
	}

	public void execute(JobExecutionContext context)
			throws JobExecutionException {

		JobDataMap data = context.getMergedJobDataMap();
		System.out.println("someProp = " + data.getString("someProp"));
	}

}

```

Define job instance:

```java
JobDetail job1 = newJob(MyJobClass.class)
    .withIdentity("job1", "group1")		//标识任务
    .usingJobData("someProp", "someValue")		//input data 
    .build();

```

<br/>

### Quartz How-To: Scheduling a Job
```java
// Define job instance
JobDetail job1 = newJob(ColorJob.class)
    .withIdentity("job1", "group1")
    .build();

// Define a Trigger that will fire "now", and not repeat
Trigger trigger = newTrigger()
    .withIdentity("trigger1", "group1")
    .startNow()
    .build();

// Schedule the job with the trigger
sched.scheduleJob(job, trigger);

```


<br/>

### How-To: Update an existing job
```java
   
// Add the new job to the scheduler, instructing it to "replace"
//  the existing job with the given name and group (if any)
JobDetail job1 = newJob(MyJobClass.class)
    .withIdentity("job1", "group1")
    .build();

// store, and set overwrite flag to 'true'     
scheduler.addJob(job1, true);

```

<br/>


### How-To: Updating a trigger
有一些业务场景，我们需要手动去更新任务的触发时间，比如某个任务是每隔10分钟触发一次，现在需要改成每隔20分钟触发一次，这样既就需要手动的更新触发器
官方的例子:
http://www.quartz-scheduler.org/documentation/quartz-2.1.x/cookbook/UpdateTrigger 

Replacing a trigger 替换触发器，通过triggerkey移除旧的触发器，同时添加一个新的进去。

```java
// Define a new Trigger 
Trigger trigger = newTrigger()
    .withIdentity("newTrigger", "group1")
    .startNow()
    .build();

// tell the scheduler to remove the old trigger with the given key, and put the new one in its place
sched.rescheduleJob(triggerKey("oldTrigger", "group1"), trigger);
```

但是有一个地方需要注意：sched.rescheduleJob(triggerKey("oldTrigger", "group1"), trigger); 这个方法返回一个Date.

如果返回 null 说明替换失败，原因就是旧触发器没有找到，所以新的触发器也不会设置进去.


<br/>

### How-To: Using Job Listeners
Quartz Scheduler 可以对Job(任务)建立一个监听器，分别对任务执行 之前-之后-取消 3个状态进行监听。 

实现监听器需要实现JobListener接口，然后注册到Scheduler上就可以了。

一：首先写一个监听器实现类
```java
package com.gary.operation.jobdemo.example1;

import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.quartz.JobListener;

public class MyJobListener implements JobListener {

    @Override
    public String getName() {
        return "MyJobListener";
    }

    /**
     * (1)
     * 任务执行之前执行
     * Called by the Scheduler when a JobDetail is about to be executed (an associated Trigger has occurred).
     */
    @Override
    public void jobToBeExecuted(JobExecutionContext context) {
        System.out.println("MyJobListener.jobToBeExecuted()");
    }

    /**
     * (2)
     * 这个方法正常情况下不执行,但是如果当TriggerListener中的vetoJobExecution方法返回true时,那么执行这个方法.
     * 需要注意的是 如果方法(2)执行 那么(1),(3)这个俩个方法不会执行,因为任务被终止了嘛.
     * Called by the Scheduler when a JobDetail was about to be executed (an associated Trigger has occurred),
     * but a TriggerListener vetoed it's execution.
     */
    @Override
    public void jobExecutionVetoed(JobExecutionContext context) {
        System.out.println("MyJobListener.jobExecutionVetoed()");
    }

    /**
     * (3)
     * 任务执行完成后执行,jobException如果它不为空则说明任务在执行过程中出现了异常
     * Called by the Scheduler after a JobDetail has been executed, and be for the associated Trigger's triggered(xx) method has been called.
     */
    @Override
    public void jobWasExecuted(JobExecutionContext context,
            JobExecutionException jobException) {
        System.out.println("MyJobListener.jobWasExecuted()");
    }

}
```

二：将这个监听器注册到Scheduler
假设有一个任务的key是 job1与 group1
```java
// define the job and tie it to our HelloJob class
        JobDetail job = newJob(HelloJob.class)
            .withIdentity("job1", "group1")
            .build();
``` 

```java
全局注册,所有Job都会起作用
Registering A JobListener With The Scheduler To Listen To All Jobs
sched.getListenerManager().addJobListener(new MyJobListener());
```

 
```java
指定具体的任务
Registering A JobListener With The Scheduler To Listen To A Specific Job
Matcher<JobKey> matcher = KeyMatcher.keyEquals(new JobKey("job1", "group1"));
sched.getListenerManager().addJobListener(new MyJobListener(), matcher);
```
```java
指定一组任务
Registering A JobListener With The Scheduler To Listen To All Jobs In a Group
GroupMatcher<JobKey> matcher = GroupMatcher.jobGroupEquals("group1");
sched.getListenerManager().addJobListener(new MyJobListener(), matcher);
```

```java
可以根据组的名字匹配开头和结尾或包含
GroupMatcher<JobKey> matcher = GroupMatcher.groupStartsWith("g");
GroupMatcher<JobKey> matcher = GroupMatcher.groupContains("g");
sched.getListenerManager().addJobListener(new MyJobListener(), matcher);
```

<br/>

### How-To: Trigger That Executes Every Day
Using CronTrigger
```java
 trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .startNow()
    .withSchedule(dailyAtHourAndMinute(15, 0)) // fire every day at 15:00
    .build();

```

Using SimpleTrigger
```java
trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .startAt(tomorrowAt(15, 0, 0)  // first fire time 15:00:00 tomorrow
    .withSchedule(simpleSchedule()
            .withIntervalInHours(24) // interval is actually set at 24 hours' worth of milliseconds
            .repeatForever())
    .build();
```

Using CalendarIntervalTrigger
```java
trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .startAt(tomorrowAt(15, 0, 0)  // 15:00:00 tomorrow
    .withSchedule(calendarIntervalSchedule()
            .withIntervalInDays(1)) // interval is set in calendar days
    .build();

```

<br/>

### Quartz Trigger Priority 触发器优先级
当多个触发器在一个相同的时间内触发，并且调度引擎中的资源有限的情况下，那么具有较高优先级的触发器先触发。

需要将配置文件中的org.quartz.threadPool.threadCount = 1设置为1，这样能更好的测试出效果。
```java
package com.gary.operation.jobdemo.example14;

import static org.quartz.DateBuilder.futureDate;
import static org.quartz.JobBuilder.newJob;
import static org.quartz.SimpleScheduleBuilder.simpleSchedule;
import static org.quartz.TriggerBuilder.newTrigger;

import java.util.Date;

import org.quartz.JobDetail;
import org.quartz.Scheduler;
import org.quartz.SchedulerFactory;
import org.quartz.Trigger;
import org.quartz.DateBuilder.IntervalUnit;
import org.quartz.impl.StdSchedulerFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class PriorityExample {
    
    public void run() throws Exception {
        Logger log = LoggerFactory.getLogger(PriorityExample.class);

        // First we must get a reference to a scheduler
        SchedulerFactory sf = new StdSchedulerFactory();
        Scheduler sched = sf.getScheduler();

        JobDetail job = newJob(TriggerEchoJob.class)
            .withIdentity("TriggerEchoJob")
            .build();
            
        Date startTime = futureDate(5, IntervalUnit.SECOND);
        
        Trigger trigger1 = newTrigger()
            .withIdentity("Priority7 Trigger5SecondRepeat")
            .startAt(startTime)
            .withSchedule(simpleSchedule().withRepeatCount(1).withIntervalInSeconds(5))
            .withPriority(7)
            .forJob(job)
            .build();

        Trigger trigger2 = newTrigger()
            .withIdentity("Priority5 Trigger10SecondRepeat")
            .startAt(startTime)
            .withPriority(5)
            .withSchedule(simpleSchedule().withRepeatCount(1).withIntervalInSeconds(5))
            .forJob(job)
            .build();
        
        Trigger trigger3 = newTrigger()
            .withIdentity("Priority10 Trigger15SecondRepeat")
            .startAt(startTime)
            .withSchedule(simpleSchedule().withRepeatCount(1).withIntervalInSeconds(5))
            .withPriority(10)
            .forJob(job)
            .build();

        // Tell quartz to schedule the job using our trigger
        sched.scheduleJob(job, trigger1);
        sched.scheduleJob(trigger2);
        sched.scheduleJob(trigger3);

        sched.start();

        log.info("------- Waiting 30 seconds... -------------");
        try {
            Thread.sleep(30L * 1000L); 
            // executing...
        } catch (Exception e) {
        }

        sched.shutdown(true);
    }

    public static void main(String[] args) throws Exception {
        PriorityExample example = new PriorityExample();
        example.run();
    }
}
```



------------------------
(笔记内容大部分来源: https://www.cnblogs.com/daxin/archive/2013/05/27/3101972.html)









