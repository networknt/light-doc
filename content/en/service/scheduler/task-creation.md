---
title: "Task Creation"
date: 2021-10-07T10:16:02-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

The scheduler supports several time units for task scheduling: MILLISECONDS, SECONDS, MINUTES, HOURS, DAYS. 

For each time unit, Kafka streams punctuate is called at each time unit. Depending on the times in the frequency, the punctuate method will decide when to send a scheduled task. 

For example, for the SECONDS time unit, we might schedule a task every 10 seconds. The punctuate method will be called every second, but we only want to send a task at 0, 10, 20, 30 seconds. 

To do the calculation, we have to round up the current time passed into the punctuate method and the start timestamp in the task definition to the next timestamp rounded by the time unit. 

Once we have two rounded-up numbers, we can get the current and start time difference. We also know how many milliseconds for the task frequency from the task definition. 

```
long period = TimeUtil.oneTimeUnitMillisecond(timeUnit) * next.value.getFrequency().getTime();

```

With the rounded-up period in milliseconds and the frequency in milliseconds, we can calculate when to schedule a task with mod calculation. 

```
if((current-start) % frequenc == 0) schedule a task.
```

To control the time for the task execution in the target topic, we need to set the task start time to the current time, and the task execution streams can use this start time to compare with the processing time to detect if the task was too old to be executed. 

For example, if the controller is restarted with all nodes, there might be a flood of tasks coming in, and it should skip old messages and only execute the new tasks. 


