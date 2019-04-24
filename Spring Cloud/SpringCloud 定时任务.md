## SpringCloud 定时任务配置

- SpringCloud本身支持开启定时任务，只需要在启动类上添加注解**@EnableScheduling**,表示当前项目支持定时任务。

- 创建定时任务，在类上添加注解**@Component**，并在需要定时执行的方法上添加注解**@Scheduled(cron = "0/5 * * * * *")**。

  > **cron参数：**
  >
  > 
  >
  > 