---
layout: post
title:  "NestJS - Cron Job"
date:   2022-05-19
categories: node
permalink: /:categories/nestjs-cronjob
---

<p style="text-align: justify;">This post has some notes and the code I did while studying nestjs. I started those notes in my <a href="https://fabiana2611.github.io/node/nestjs">first post</a> about nestjs with simple use of the components, the <a href="https://fabiana2611.github.io/node/nestjs-advanced">second post</a> about how to use the components and the third post regarding <a href="https://fabiana2611.github.io/node/nestjs-security">security</a> with nestjs. Now, I am going a step forward to use scheduled tasks with nestjs. More detail about it you can see in the <a href="https://docs.nestjs.com/techniques/task-scheduling/">nestjs page</a>. The code used in this post you can find inside the <a href="https://github.com/fabiana2611/nestjs2/tree/master/tasks">tasks project</a>.</p>

<p>If you want some tips to calculate the time here is the <a href="https://crontab.guru/#0_18,20,22_*_*_*">crontab guru</a>.</p>


<h3>Cron Job</h3>

<blockquote>Task scheduling allows you to schedule arbitrary code (methods/functions) to execute at a fixed date/time, at recurring intervals, or once after a specified interval. </blockquote>

<p>Packages to handle the tasks:</p>
- cron: from linux
- Schedule: from Nestjs
- node-cron: from Node.js
- Agenda, Node-Schedule: MIT-licensed scheduler for Node

<p>A comparing between some of them you can see <a href="https://blog.logrocket.com/comparing-best-node-js-schedulers/">here</a>.</p>

{% highlight ruby %}
$ npm install --save @nestjs/schedule
$ npm install --save-dev @types/cron
{% endhighlight %}

<p style="text-align: justify;">It's necessary to declare it inside your module and add the annotation inside your task to define when it should be executed. The annotation can be done using a value or a enum. Alternatively you can add parameters such as name and time zone. Also it's possible attribute intervals of execution or timeouts.</p>

{% highlight ruby %}
# Declaring the module
@Module({
  imports: [ ScheduleModule.forRoot() ],
})
export class AppModule {}

# Inside the task (Use annotation or the value)
// @Cron(CronExpression.EVERY_30_SECONDS)
@Cron('30 * * * * *')
firstCron() {}

# Using attributes inside the annotation
@Cron('* * 0 * * *', {
    name: 'secondCron',
    timeZone: 'Europe/Paris',
})
secondCron() {}

# By intervals
@Interval(10000)
thridCron() {}

# By Timeout
@Timeout(5000)
fourthCron() {}

# Means of the @Cron definition
('* | * | * | * | * | *')
(second | minutes | hours | day of month | months | day of week)
{% endhighlight %}

<h3>Dynamic cron job</h3>

<p style="text-align: justify;">It's possible handle the cron jobs dynamically using the SchedulerRegistry which will use the name define inside the annotation of the the task.</p>

{% highlight ruby %}
constructor(private schedulerRegistry: SchedulerRegistry) {}

const job = this.schedulerRegistry.getCronJob('secondCron');

job.stop();
console.log(job.lastDate());
{% endhighlight %}

<p style="text-align: justify;">The same way, the SchedulerRegistry is used to register a new job dynamically.</p>

{% highlight ruby %}
addCronJob(name: string, seconds: string) {
  const job = new CronJob(`${seconds} * * * * *`, () => {
    this.logger.warn(`time (${seconds}) for job ${name} to run!`);
  });

  this.schedulerRegistry.addCronJob(name, job);
  job.start();
  this.logger.warn(`job ${name} added for each minute at ${seconds} seconds!`,);
}
{% endhighlight %}

<p style="text-align: justify;">The cron allow delete and list all the jobs.  </p>

<h3>Stackoverflow Scenarios</h3>

<h4><u>Scenario One</u></h4>

<p>The first scenario is: how to execute a job only three times and finish it.</p>

<p>The solution used here is using the timeout. You have to take the period the job has to execute and multiply by how many rounds you want to execute. Also, the example is using the dynamic mode to have the values as parameters. </p>

<p>So, considering the intervals of execution is each 2 seconds and you want this job execute 3 times, the limit of time used will be 2 * 3 * 1000. The job will stop after 6 seconds after the start.</p>

<p>That scenario was get in <a href="https://stackoverflow.com/questions/70077725/add-counter-to-nestjs-cron-job/72310582#72310582">stackoverflow</a>.<p>

{% highlight ruby %}
private static time = 2;
private static quantity = 3;
private static limit = TaskService.time * TaskService.quantity;
private nameTask = '### TESTE  3 Times ###';

# Should be called when the job process is started
threeTimes() {
  this.addCronJon(
    this.nameTask,
    TaskService.time.toString(),
    TaskService.quantity,
  );
}

# Create job dynamically
addCronJon(name: string, seconds: string, limit?: number) {
    const job = new CronJob(`*/${seconds} * * * * *`, () => {
      this.logger.warn(`time (${seconds} for job ${name} to run!`);
    });

    this.schedulerRegistry.addCronJob(name, job);
    job.start();

    this.logger.warn(
      `job ${name} added for each minute at ${seconds} seconds!`,
    );
  }

# The method will handle when the job should be interrupted
@Timeout(TaskService.limit * 1000 + 1000)
threeTimesTimeout() {
    this.schedulerRegistry.deleteCronJob(this.nameTask);
    this.logger.warn(`job ${this.nameTask} deleted!`);
}

# console
[Nest] 9:08:29 AM     LOG [NestApplication] Nest application successfully started +2ms
[Nest] 9:08:30 AM    WARN [TaskService] job MY_TASK added for each minute at 2 seconds!
[Nest] 9:08:32 AM    WARN [TaskService] time (2 for job MY_TASK to run!
[Nest] 9:08:34 AM    WARN [TaskService] time (2 for job MY_TASK to run!
[Nest] 9:08:36 AM    WARN [TaskService] time (2 for job MY_TASK to run!
[Nest] 9:08:36 AM    WARN [TaskService] job MY_TASK deleted!

{% endhighlight %}

<h3>References</h3>
<ul>
  <li><a href="https://fabiana2611.github.io/node/nestjs">NestJS - using the main components</a></li>
  <li><a href="https://fabiana2611.github.io/node/nestjs-advanced">NestJS - components part II</a></li>
  <li><a href="https://fabiana2611.github.io/node/nestjs-security">NestJs - Security</a></li>
  <li><a href="https://docs.nestjs.com/techniques/task-scheduling/">Nestjs page</a></li>
  <li><a href="https://github.com/fabiana2611/nestjs2/tree/master/tasks">GitHub - tasks project</a></li>
  <li><a href="https://crontab.guru/#0_18,20,22_*_*_*">crontab guru</a></li>
  <li><a href="https://github.com/agenda/agenda#readme">Agenda</a></li>
</ul>
