## Quartz学习
(笔记内容来源: https://www.cnblogs.com/daxin/archive/2013/05/27/3101972.html)
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





































