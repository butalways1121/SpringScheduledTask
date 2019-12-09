# SpringScheduledTask
Spring定时任务
**定时任务，即在某个时间点做某件事情，核心是以时间为关注点，即在一个特定的时间点，系统执行指定的一个操作，本文主要介绍了Spring中定时任务的三种创建方式，戳[这里](https://gitee.com/butalways1121/SpringScheduledTask.git)下载源码。**
## 一、静态：基于注解@Scheduled
&emsp;&emsp;基于注解`@Scheduled`默认为单线程，开启多个任务时，任务的执行时机会受上一个任务执行时间的影响。
<!--more-->

&emsp;&emsp;使用SpringBoot基于注解来创建定时任务非常简单，只需几行代码便可完成，所需依赖：
```bash
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.1.RELEASE</version>
	</parent>
<!-- SpringBoot 核心组件 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
定时器代码示例：
```bash
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import java.util.Date;
@Component
@EnableScheduling
public class Job {
    @Scheduled(fixedDelay = 5000)
    public void fixedDelayJob() throws InterruptedException {
        System.out.println("fixedDelay 每隔5秒" + new Date());
    }

    @Scheduled(fixedRate = 3000)
    public void fixedRateJob() {

        System.out.println("fixedRate 每隔3秒" + new Date());
    }

    @Scheduled(cron = "0 0,30 0,8 ? * ? ")
    public void cronJob() {
        System.out.println(new Date() + " ...>>cron....");
    }
}
```
运行结果：
```bash
fixedRate 每隔3秒Mon Dec 09 11:33:11 CST 2019
fixedDelay 每隔5秒Mon Dec 09 11:33:13 CST 2019
fixedRate 每隔3秒Mon Dec 09 11:33:14 CST 2019
fixedRate 每隔3秒Mon Dec 09 11:33:17 CST 2019
fixedDelay 每隔5秒Mon Dec 09 11:33:18 CST 2019
fixedRate 每隔3秒Mon Dec 09 11:33:20 CST 2019
........
```
**总结一下：**
&emsp;&emsp;1、`@EnableScheduling`注解，其作用是自动扫描注解`@Scheduled`的任务并后台执行。

&emsp;&emsp;2、`@Scheduled`注解用于标注某个方法是一个定时任务的方法，该注解有三个属性：`fixedDelay`、`fexRate`和`cron`，都是用来设置调度时间。

&emsp;&emsp;3、`fixedDelay`属性表示该任务执行完后隔多长时间再调用。

&emsp;&emsp;4、`fixedRate`属性表示任务隔多长时间调用一次，不管任务是否执行完成。

&emsp;&emsp;5、`cron`属性是以表达式的形式来表示时间，比如要设置每天什么时候执行，就可以用cron表达式。corn表达式是一个字符串，以5或6个空格隔开，分为6或7个域，每一个域代表一个含义，如下：

| 字段 | 允许值 | 允许的特殊字符 |
|:------- | :------- |:------- |
| 秒（Seconds） | 0~59的整数 | , - * /    四个字符 |
| 分（Minutes） | 0~59的整数 | , - * /    四个字符 |
| 小时（Hours） | 0~23的整数 | , - * /    四个字符 |
| 日期（DayofMonth） | 1~31的整数（但是你需要考虑你月的天数） | ,- * ? / L W C     八个字符 |
| 月份（Month） | 1~12的整数或者 JAN-DEC | 	, - * ? / , - * /    四个字符 |
| 星期（DayofWeek） | 1~7的整数或者 SUN-SAT（1=SUN）,1表示星期天，2表示星期一 | , - * ? / L C #     八个字符 |
| 年(可选，留空)（Year） | 1970~2099 | , - * /    四个字符 |

特殊符号的含义如下：
```bash
*: 表示匹配该域的任意值，假如在Minutes域使用*, 即表示每分钟都会触发事件。

?: 只能用在DayofMonth和DayofWeek两个域。它也匹配域的任意值，但实际不会。因为DayofMonth和DayofWeek会相互影响。例如想在每月的20日触发调度，不管20日到底是星期几，则只能使用如下写法： 13 13 15 20 * ?, 其中最后一位只能用？，而不能使用*，如果使用*表示不管星期几都会触发，实际上并不是这样。

-: 表示范围，例如在Minutes域使用5-20，表示从5分到20分钟每分钟触发一次。

/：表示起始时间开始触发，然后每隔固定时间触发一次，例如在Minutes域使用5/20,则意味着5分钟触发一次，而25，45等分别触发一次。

,: 表示列出枚举值。例如：在Minutes域使用5,20，则意味着在5和20分每分钟触发一次。

L: 表示最后，只能出现在DayofWeek和DayofMonth域，如果在DayofWeek域使用5L,意味着在最后的一个星期四触发。

W: 表示有效工作日(周一到周五)，只能出现在DayofMonth域，系统将在离指定日期的最近的有效工作日触发事件。例如：在 DayofMonth使用5W，如果5日是星期六，则将在最近的工作日：星期五，即4日触发。如果5日是星期天，则在6日(周一)触发；如果5日在星期一 到星期五中的一天，则就在5日触发。另外一点，W的最近寻找不会跨过月份

LW: 这两个字符可以连用，表示在某个月最后一个工作日，即最后一个星期五。

#: 用于确定每个月第几个星期几，只能出现在DayofMonth域。例如在4#2，表示某月的第二个星期三。
```
corn表达式举例如下：
```bash
0 0 2 1 * ? *   表示在每月的1日的凌晨2点执行任务

0 15 10 ? * MON-FRI   表示周一到周五每天上午10:15执行作业

0 15 10 ? 6L 2002-2006   表示2002-2006年的每个月的最后一个星期五上午10:15执行作

0 0 10,14,16 * * ?   每天上午10点，下午2点，4点 

0 0/30 9-17 * * ?   朝九晚五工作时间内每半小时 

0 0 12 ? * WED    表示每个星期三中午12点 

0 0 12 * * ?   每天中午12点触发 

0 15 10 ? * *    每天上午10:15触发 

0 15 10 * * ?     每天上午10:15触发 

0 15 10 * * ? *    每天上午10:15触发 

0 15 10 * * ? 2005    2005年的每天上午10:15触发 

0 * 14 * * ?     在每天下午2点到下午2:59期间的每1分钟触发 

0 0/5 14 * * ?    在每天下午2点到下午2:55期间的每5分钟触发 

0 0/5 14,18 * * ?     在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发 

0 0-5 14 * * ?    在每天下午2点到下午2:05期间的每1分钟触发 

0 10,44 14 ? 3 WED    每年三月的星期三的下午2:10和2:44触发 

0 15 10 ? * MON-FRI    周一至周五的上午10:15触发 

0 15 10 15 * ?    每月15日上午10:15触发 

0 15 10 L * ?    每月最后一日的上午10:15触发 

0 15 10 ? * 6L    每月的最后一个星期五上午10:15触发 

0 15 10 ? * 6L 2002-2005   2002年至2005年的每月的最后一个星期五上午10:15触发 

0 15 10 ? * 6#3   每月的第三个星期五上午10:15触发

```

## 二、动态：基于接口SchedulingConfigurer
&emsp;&emsp;显然，使用`@Scheduled`注解很方便，但缺点是当我们调整了执行周期的时候，需要重启应用才能生效。为了达到实时生效的效果，可以使用接口来完成定时任务:

1、导入依赖：
```bash
<parent>
    <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.1.RELEASE</version>
</parent>
<!-- SpringBoot 核心组件 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--添加MySql依赖 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<!--添加Mybatis依赖 配置mybatis的一些初始化的东西-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>
<!-- 添加mybatis依赖 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.5</version>
    <scope>compile</scope>
</dependency>
```

2、添加数据库记录：
&emsp;&emsp;在本地test数据库中新建查询执行如下脚本内容，添加corn表达式的值：
```bash
DROP TABLE IF EXISTS `cron`;
CREATE TABLE `cron`  (
  `cron_id` varchar(30) NOT NULL PRIMARY KEY,
  `cron` varchar(30) NOT NULL  
);
INSERT INTO `cron` VALUES ('1', '0/10 * * * * ?');
```

3、application.properties文件配置信息：
```bash
server.port=8088
spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

4、创建定时器：
&emsp;&emsp;数据库准备好数据之后，接下来来编写定时任务，注意这里添加的是**TriggerTask**，目的是循环读取我们在数据库设置好的执行周期，以及执行相关定时任务的内容，示例如下：
```bash
import java.util.Date;

import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;
import org.springframework.scheduling.support.CronTrigger;
import org.springframework.stereotype.Component;

import com.mysql.jdbc.StringUtils;

@Component
@Configuration      //主要用于标记配置类，兼备Component的效果。
@EnableScheduling   // 开启定时任务
public class DynamicJob implements SchedulingConfigurer {

    @Mapper
    public interface CronMapper {
        @Select("select cron from cron limit 1")
        public String getCron();
    }

    @Autowired      //注入mapper
    @SuppressWarnings("all")
    CronMapper cronMapper;

    /**
     * 执行定时任务.
     */
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {

        taskRegistrar.addTriggerTask(
                //1.添加任务内容(Runnable)
                () -> System.out.println("执行动态定时任务: " + new Date()),
                //2.设置执行周期(Trigger)
                triggerContext -> {
                    //2.1 从数据库获取执行周期
                    String cron = cronMapper.getCron();
                    System.out.println("cron="+cron);
                    //2.2 合法性校验.
                    if (StringUtils.isNullOrEmpty(cron)) {
                        // Omitted Code ..
                    }
                    //2.3 返回执行周期(Date)
                    return new CronTrigger(cron).nextExecutionTime(triggerContext);
                }
        );
    }

}
```

5、测试
&emsp;&emsp;运行结果如下，可以执行周期是在数据库设置的10s一次：
```bash
cron=0/10 * * * * ?
执行动态定时任务: Mon Dec 09 14:12:40 CST 2019
cron=0/10 * * * * ?
执行动态定时任务: Mon Dec 09 14:12:50 CST 2019
cron=0/10 * * * * ?
执行动态定时任务: Mon Dec 09 14:13:00 CST 2019
cron=0/10 * * * * ?
执行动态定时任务: Mon Dec 09 14:13:10 CST 2019
cron=0/10 * * * * ?
.......
```
&emsp;&emsp;接着，修改数据库corn的值为`0/6 * * * * ?`，表示定时任务周期为6s，控制台打印：
```bash
cron=0/6 * * * * ?
执行动态定时任务: Mon Dec 09 14:14:24 CST 2019
cron=0/6 * * * * ?
执行动态定时任务: Mon Dec 09 14:14:30 CST 2019
cron=0/6 * * * * ?
执行动态定时任务: Mon Dec 09 14:14:36 CST 2019
cron=0/6 * * * * ?
.......
```
**注：如果在数据库修改时格式出现错误，则定时任务会停止，即使重新修改正确也只能重新启动项目才能恢复。**
## 三、基于注解设定多线程定时任务
&emsp;&emsp;基于注解创建多线程任务代码如下：
```bash
import java.time.LocalDateTime;

import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;


@Component
@EnableScheduling // 1.开启定时任务
@EnableAsync // 2.开启多线程
public class MultiThreadJob {

	@Async
	@Scheduled(fixedDelay = 1000) // 间隔1秒
	public void first() throws InterruptedException {
		System.out.println(
				"第一个定时任务开始 : " + LocalDateTime.now().toLocalTime() + "\r\n线程 : " + Thread.currentThread().getName());
		System.out.println();
		Thread.sleep(1000 * 10);
	}

	@Async
	@Scheduled(fixedDelay = 2000) // 间隔2秒
	public void second() {
		System.out.println(
				"第二个定时任务开始 : " + LocalDateTime.now().toLocalTime() + "\r\n线程 : " + Thread.currentThread().getName());
		System.out.println();
	}
}
```
**其中，`@Async`注解非常重要，在Spring中，基于`@Async`标注的方法，称之为异步方法；这些方法将在执行的时候，将会在独立的线程中被执行，调用者无需等待它的完成，即可继续其他的操作。**

启动程序，允许结果如下：
```bash
第二个定时任务开始 : 14:22:32.584
线程 : SimpleAsyncTaskExecutor-2
第一个定时任务开始 : 14:22:32.584
线程 : SimpleAsyncTaskExecutor-1
第一个定时任务开始 : 14:22:33.558
线程 : SimpleAsyncTaskExecutor-3
第二个定时任务开始 : 14:22:34.559
线程 : SimpleAsyncTaskExecutor-4
第一个定时任务开始 : 14:22:34.559
线程 : SimpleAsyncTaskExecutor-5
第一个定时任务开始 : 14:22:35.560
线程 : SimpleAsyncTaskExecutor-6
```
&emsp;&emsp;以上结果可以看出第一个定时任务和第二个定时任务执行时间互不影响，两者每次执行的时间间隔分别是1s和2s。
