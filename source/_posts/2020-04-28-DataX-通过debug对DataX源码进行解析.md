---
title: DataX-通过debug对DataX源码进行解析
top: true
cover: true
date: 2020-04-28 16:46:18
categories: DataX
tags: 
  - java
  - 技术
---
> **<font color=green>写在前面</font>**：公司项目上用到了kettle作为项目的底层同步工具，后来kettle无法满足大数据和分布式环境下的数据同步，也把DataX集成进来，我对DataX同步只是停留在使用上，这次花了几天时间调试DataX源码和百度资料，把这几天的收获总结一下，为后面开发插件做铺垫。废话少说，开始上代码！

### 一、本篇教程侧重点导读
> 1. 总体图解DataX；
> 2. 准备json配置文件；
> 3. DataX的主程序入口Engine的解析；
> 4. JobContainer.start()方法做了哪些事？
> 5. JobContainer：init()实例化reader、writer的job对象；
> 6. JobContainer：split()根据用户配置拆分job成多个task；
> 7. JobContainer：schedule()将reader和writer split的结果整合到具体taskGroupContainer中,同时所有任务调度起来；
> 8. AbstractScheduler：统领全局，周期性汇报日志；
> 9. ProcessInnerScheduler：；创建并启动固定的线程数的TaskGroupContainerRunner；
> 10. TaskGroupContainer：分配的Task，创建并执行TaskExecutor；
> 11. TaskExecutor：一个完整task的执行器；
> 12. invokeHooks用法？



### 二、本篇教程用的软件、技术和说明
> 1. jdk版本：1.8.0_202-b08；
> 2. Python版本：2.7.18（官方推荐2.6.X）；
> 3. Maven版本；3.6.0 ；
> 4. DataX是直接拉取的master分支上的源码；
> 5. **<font color=green>本篇文章也是跟着代码流程走的，根据DataX的图解、按照解析步骤往下看，然后自己多调试几遍，基本上都可以理解清楚DataX内部的核心工作流程，另外本篇篇幅较长，需耐心揣摩和调试。</font>**；

### 三、总体图解DataX
1. 先上一张github上官方描述图：
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200428/3.1.png"  align=left/>
 官方图已经把DataX的整个架构说明的很清楚：用户提交一个job，会根据用户配置、核心配置、使用到的插件配置将job拆分成多个task，然后会很公平的将task分组成多个taskGroup，然后在根据taskGroup的数量创建固定数量的线程池去运行task。
2. 再上一张更细分的图解(这张图有点大，加载较慢)：
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200428/3.2.jpg"  align=left/>
 如果要是还没有开始看源码，先看看这张图，知道了运行流程再去看源码，会更好理解，如果已经看完源码在看这张图，对DataX的源码更能加深印象。

### 四、准备json配置文件
````json
{
  "job": {
    "setting": {
      "speed": {
        "channel": 10
      },
      "errorLimit": {
        "record": 10000,
        "percentage": 1
      }
    },
    "content": [
      {
        "reader": {
          "name": "mysqlreader",
          "parameter": {
            "username": "root",
            "password": "root",
            "column": [
              "MP_ID",
              "LOAD_TIME",
              "DATA_TIME",
              "POS_P_E_TOTAL",
              "REV_P_E_TOTAL",
              "GROUP_P_E_TOTAL",
              "GROUP_Q_E_1",
              "GROUP_Q_E_2",
              "QUAD_1_Q_E_TOTAL",
              "QUAD_2_Q_E_TOTAL",
              "QUAD_3_Q_E_TOTAL",
              "QUAD_4_Q_E_TOTAL",
              "DATA_FLAG"
            ],
            "splitPk": "MP_ID",
            "connection": [
              {
                "table": [
                  "dr_e_raw_hour_202004_debug"
                ],
                "jdbcUrl": [
                  "jdbc:mysql://192.168.1.202:3306/test"
                ]
              }
            ]
          }
        },
        "writer": {
          "name": "mysqlwriter",
          "parameter": {
            "username": "root",
            "password": "root",
            "column": [
              "MP_ID",
              "LOAD_TIME",
              "DATA_TIME",
              "POS_P_E_TOTAL",
              "REV_P_E_TOTAL",
              "GROUP_P_E_TOTAL",
              "GROUP_Q_E_1",
              "GROUP_Q_E_2",
              "QUAD_1_Q_E_TOTAL",
              "QUAD_2_Q_E_TOTAL",
              "QUAD_3_Q_E_TOTAL",
              "QUAD_4_Q_E_TOTAL",
              "DATA_FLAG"
            ],
            "connection": [
              {
                "table": [
                  "dr_e_raw_hour_202004_debug"
                ],
                "jdbcUrl": "jdbc:mysql://192.168.1.202:3306/test1"
              }
            ]
          }
        }
      }
    ]
  }
}
````

 **<font color=red>防坑</font>**：这个json没什么特别的，主要是配置了切分规则job.content.reader.parameter.splitPk="MP_ID"和job.setting.speed.channel=10，配置这两个参数主要是一会debug的时候，DataX会根据job拆分成多个task，如果不配置splitPk，那么程序不会并发的去同步数据！

### 五、DataX的主程序入口Engine的解析
1. Engine.entry(final String[] args)的方法解析如下：
````java
/**
 * 说明：
 * entry 函数主要做两件事情，分别是：
 *
 * 1、解析job相关配置生成configuration。
 * 2、依据配置启动Engine。
 */
public static void entry(final String[] args) throws Throwable {
	
	//省略代码

	/**
	 * by 耳东陈
	 * 获取job的配置路径信息
	 */
	String jobPath = cl.getOptionValue("job");

	//省略代码

	/**
	 * by 耳东陈
	 * 划重点-解析配置信息
	 */
	Configuration configuration = ConfigParser.parse(jobPath);

	//省略代码
	
	Engine engine = new Engine();
	/**
     * by 耳东陈
     * 说明：
     * start过程中做了两件事：
     *
     * 1、创建JobContainer对象
     * 2、启动JobContainer对象
     */
	engine.start(configuration);
}
````
2. 划重点-Configuration configuration = ConfigParser.parse(jobPath)代码解析：
````java
/**
 * by 耳东陈
 * 说明：
 * configuration解析包括三部分的配置解析合并解析结果并返回，分别是：
 *
 * 1、解析job的配置信息，由启动参数指定job.json文件。
 * 2、解析DataX自带配置信息，由默认指定的core.json文件。
 * 3、解析读写插件配置信息，由job.json指定的reader和writer插件信息
 */
public static Configuration parse(final String jobPath) {
	/**
	 * by 耳东陈
	 * 加载任务的指定的配置文件，这个配置是有固定的json的固定模板格式的
	 */
	Configuration configuration = ConfigParser.parseJobConfig(jobPath);

	/**
	 * by 耳东陈
	 * 合并conf/core.json的配置文件
	 */
	configuration.merge(
			ConfigParser.parseCoreConfig(CoreConstant.DATAX_CONF_PATH),
			false);
	// todo config优化，只捕获需要的plugin
	/**
	 * by 耳东陈
	 * 固定的节点路径 job.content[0].reader.name
	 */
	String readerPluginName = configuration.getString(
			CoreConstant.DATAX_JOB_CONTENT_READER_NAME);
	String writerPluginName = configuration.getString(
			CoreConstant.DATAX_JOB_CONTENT_WRITER_NAME);

	String preHandlerName = configuration.getString(
			CoreConstant.DATAX_JOB_PREHANDLER_PLUGINNAME);

	String postHandlerName = configuration.getString(
			CoreConstant.DATAX_JOB_POSTHANDLER_PLUGINNAME);

	/**
	 * by 耳东陈
	 * 添加读写插件的列表待加载
	 */
	Set<String> pluginList = new HashSet<String>();
	pluginList.add(readerPluginName);
	pluginList.add(writerPluginName);

	if(StringUtils.isNotEmpty(preHandlerName)) {
		pluginList.add(preHandlerName);
	}
	if(StringUtils.isNotEmpty(postHandlerName)) {
		pluginList.add(postHandlerName);
	}
	try {
		/**
		 * by 耳东陈
		 * parsePluginConfig(new ArrayList<String>(pluginList))加载指定的插件的配置信息，并且和全局的配置文件进行合并
		 */
		configuration.merge(parsePluginConfig(new ArrayList<String>(pluginList)), false);
	}catch (Exception e){
		//吞掉异常，保持log干净。这里message足够。
		LOG.warn(String.format("插件[%s,%s]加载失败，1s后重试... Exception:%s ", readerPluginName, writerPluginName, e.getMessage()));
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e1) {
			//
		}
		configuration.merge(parsePluginConfig(new ArrayList<String>(pluginList)), false);
	}

	return configuration;
}
````

### 六、JobContainer.start()方法做了哪些事？
````java
/**
 * jobContainer主要负责的工作全部在start()里面，包括init、prepare、split、scheduler、
 * post以及destroy和statistics
 */
/**
 * by 耳东陈
 * 说明：
 * JobContainer的start方法会执行一系列job相关的操作，如下：
 *
 * 1、执行job的preHandle()操作，暂时不关注。
 * 2、执行job的init()操作，需重点关注。
 * 3、执行job的prepare()操作，暂时不关注。
 * 4、执行job的split()操作，需重点关注。
 * 5、执行job的schedule()操作，需重点关注。
 * 6、执行job的post()和postHandle()操作，暂时不关注。
 */
@Override
public void start() {
	LOG.info("DataX jobContainer starts job.");

	boolean hasException = false;
	boolean isDryRun = false;
	try {
		this.startTimeStamp = System.currentTimeMillis();
		isDryRun = configuration.getBool(CoreConstant.DATAX_JOB_SETTING_DRYRUN, false);
		if(isDryRun) {
			LOG.info("jobContainer starts to do preCheck ...");
			this.preCheck();
		} else {
			/**
			 * by 耳东陈
			 * 拷贝一份新的配置，保证线程安全
			 */
			userConf = configuration.clone();
			LOG.debug("jobContainer starts to do preHandle ...");
			/**
			 * by 耳东陈
			 * 执行preHandle()操作
			 */
			this.preHandle();

			LOG.debug("jobContainer starts to do init ...");
			
			this.init();
			LOG.info("jobContainer starts to do prepare ...");
			/**
			 * by 耳东陈
			 * 执行plugin的prepare
			 */
			this.prepare();

			LOG.info("jobContainer starts to do split ...");
			/**
			 * by 耳东陈
			 * 说明：
			 * DataX的job的split过程主要是根据限流配置计算channel的个数，进而计算task的个数
			 */
			this.totalStage = this.split();

			LOG.info("jobContainer starts to do schedule ...");
			/**
			 * by 耳东陈
			 * 执行任务调度
			 */
			this.schedule();
			LOG.debug("jobContainer starts to do post ...");
			/**
			 * by 耳东陈
			 * 执行后置操作
			 */
			this.post();

			LOG.debug("jobContainer starts to do postHandle ...");
			/**
			 * by 耳东陈
			 * 执行postHandle操作
			 */
			this.postHandle();
			LOG.info("DataX jobId [{}] completed successfully.", this.jobId);

			this.invokeHooks();
		}
	} catch (Throwable e) {
	    //省略代码
	} finally {
		//省略代码
	}
}
````

### 七、JobContainer：init()实例化reader、writer的job对象
````java
/**
 * reader和writer的初始化
 */
 /**
 * by 耳东陈
 * 说明：
 * Job的init()过程主要做了两个事情，分别是:
 *
 * 1、创建reader的job对象，通过URLClassLoader实现类加载。
 * 2、创建writer的job对象，通过URLClassLoader实现类加载。
 */
private void init() {
	this.jobId = this.configuration.getLong(
			CoreConstant.DATAX_CORE_CONTAINER_JOB_ID, -1);

	if (this.jobId < 0) {
		LOG.info("Set jobId = 0");
		this.jobId = 0;
		this.configuration.set(CoreConstant.DATAX_CORE_CONTAINER_JOB_ID,
				this.jobId);
	}

	Thread.currentThread().setName("job-" + this.jobId);

	/**
	 * by 耳东陈
	 * 初始化
	 */
	JobPluginCollector jobPluginCollector = new DefaultJobPluginCollector(
			this.getContainerCommunicator());
	//必须先Reader ，后Writer
	this.jobReader = this.initJobReader(jobPluginCollector);
	this.jobWriter = this.initJobWriter(jobPluginCollector);
}
````

### 八、JobContainer：split()根据用户配置拆分job成多个task
 1. DataX的job的split过程主要是根据限流配置计算channel的个数，进而计算task的个数，主要过程如下：
  ①、adjustChannelNumber的过程根据按照字节限流和record限流计算channel的个数。
  ②、reader的个数根据channel的个数进行计算。
  ③、writer的个数根据reader的个数进行计算，writer和reader实现1:1绑定。
  ④、通过mergeReaderAndWriterTaskConfigs()方法生成reader+writer的task的configuration，至此我们生成了task的配置信息。
 2. adjustChannelNumber()方法是计算needChannelNumber的数量
 <img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200428/8.2.png"  align=left/>
 3. this.doReaderSplit(this.needChannelNumber)根据channel的个数进行计算readerTask，该方法最终会调用ReaderSplitUtil.doSplit(Configuration originalSliceConfig, int adviceNumber)方法，其逻辑是：根据用户配置的splitPk，去数据库查询最大值和最小值，在根据计算出来的channel个数去切分N个sql语句，每个sql语句会被包装成一个task去运行；
 <img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200428/8.3.1.png"  align=left/>
 <img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200428/8.3.2.png"  align=left/>
 4. mergeReaderAndWriterTaskConfigs()方法是生成task配置信息
 <img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200428/8.4.png"  align=left/>
 拆分逻辑至此结束。
 
### 九、JobContainer：schedule()的作用
````java
/**
 * schedule首先完成的工作是把上一步reader和writer split的结果整合到具体taskGroupContainer中,
 * 同时不同的执行模式调用不同的调度策略，将所有任务调度起来
 */
/**
 * by 耳东陈
 * 执行任务调度
 * 说明：
 * Job的schedule的过程主要做了两件事，分别是：
 *
 * 1、将task拆分成taskGroup，生成List<Configuration> taskGroupConfigs。
 * 2、启动taskgroup的对象， scheduler.schedule(taskGroupConfigs)。
 */
private void schedule() {
	/**
	 * 这里的全局speed和每个channel的速度设置为B/s
	 */
	int channelsPerTaskGroup = this.configuration.getInt(
			CoreConstant.DATAX_CORE_CONTAINER_TASKGROUP_CHANNEL, 5);
	int taskNumber = this.configuration.getList(
			CoreConstant.DATAX_JOB_CONTENT).size();

	this.needChannelNumber = Math.min(this.needChannelNumber, taskNumber);
	PerfTrace.getInstance().setChannelNumber(needChannelNumber);

	/**
	 * by 耳东陈
	 * 这里会公平的将task任务分成多个taskGroup，在后面会根据每组任务数量创建线程池，执行task
	 * 通过获取配置信息得到每个taskGroup需要运行哪些tasks任务
	 */
	List<Configuration> taskGroupConfigs = JobAssignUtil.assignFairly(this.configuration,
			this.needChannelNumber, channelsPerTaskGroup);

	LOG.info("Scheduler starts [{}] taskGroups.", taskGroupConfigs.size());

	ExecuteMode executeMode = null;
	AbstractScheduler scheduler;
	try {
		//省略代码
		
		/**
		 * by 耳东陈
		 * 开始调度所有的taskGroup
		 */
		scheduler.schedule(taskGroupConfigs);

		this.endTransferTimeStamp = System.currentTimeMillis();
	} catch (Exception e) {
		//省略代码
	}

	/**
	 * 检查任务执行情况
	 */
	this.checkLimit();
}
````

### 十、AbstractScheduler：统领全局，周期性汇报日志
 1. startAllTaskGroup(configurations);此方法会启动所有的TaskGroup
 2. 进入while循环，循环体内部逻辑为：
  ①、收集任务状态；
  ②、获取报告，然后报告；
    Communication reportCommunication = CommunicationTool.getReportCommunication(nowJobContainerCommunication, lastJobContainerCommunication, totalTasks);
    this.containerCommunicator.report(reportCommunication);
  ③、错误限制检查；
    errorLimit.checkRecordLimit(nowJobContainerCommunication);
  ④、判断所有的taskGroup是不是都已经完成了，执行完成就退出；
    nowJobContainerCommunication.getState() == State.SUCCEEDED
  ⑤、判断进程状态；dealKillingStat()
  ⑥、判断失败状态；dealFailedStat()
  ⑦、刷新上一个作业状态，然后在下一个时间睡眠；
  
### 十一、ProcessInnerScheduler：；创建并启动固定的线程数的TaskGroupContainerRunner；
ProcessInnerScheduler的startAllTaskGroup方式是重写了AbstractScheduler的startAllTaskGroup方法，其逻辑很明朗
````java
/**
 * by 耳东陈
 * 说明：
 * TaskGroup的Schedule方法做的事情如下：
 *
 * 1、为所有的TaskGroup创建TaskGroupContainerRunner。
 * 2、通过线程池提交TaskGroupContainerRunner任务，执行TaskGroupContainerRunner的run()方法。
 * 3、在run()方法内部执行this.taskGroupContainer.start()方法。
 */
@Override
public void startAllTaskGroup(List<Configuration> configurations) {
	/**
	 * by 耳东陈
	 * 根据taskGroup的数量启动固定的线程数
	 */
	this.taskGroupContainerExecutorService = Executors
			.newFixedThreadPool(configurations.size());

	/**
	 * by 耳东陈
	 * 每个taskGroup启动一个TaskGroupContainerRunner
	 */
	for (Configuration taskGroupConfiguration : configurations) {
		/**
		 * by 耳东陈
		 * 创建TaskGroupContainerRunner并提交线程池运行
		 */
		/**
		 * by 耳东陈
		 * 在TaskGroupContainerRunner的run()方法内部执行this.taskGroupContainer.start()方法。
		 */
		TaskGroupContainerRunner taskGroupContainerRunner = newTaskGroupContainerRunner(taskGroupConfiguration);
		this.taskGroupContainerExecutorService.execute(taskGroupContainerRunner);
	}

	/**
	 * by 耳东陈
	 * 等待所有任务执行完后会关闭，执行该方法后不会再接收新任务
	 */
	this.taskGroupContainerExecutorService.shutdown();
}
````

### 十二、TaskGroupContainer：分配的Task，创建并执行TaskExecutor；
TaskGroupContainer的start方式是在上一步骤中执行`this.taskGroupContainerExecutorService.execute(taskGroupContainerRunner);`的时候会启动每个TaskGroupContainerRunner线程，在这个run方法里面会被调度起来
TaskGroupContainer的内部主要做的事情如下：
 1. 根据TaskGroupContainer分配的Task任务列表，创建TaskExecutor对象。
 2. 创建TaskExecutor对象，用以启动分配该TaskGroup的task。
 3. 至此，已经成功的启动了Job当中的Task任务。

核心逻辑在于while循环：
````java
/**
 * by 耳东陈
 *  下面实现主要分为以下几个步骤：
 *    循环检测所有任务的执行状态
 *       1）判断是否有失败的task，如果有则放入失败对立中，并查看当前的执行是否支持重跑和failOver，如果支持则重新放回执行队列中；如果没有失败，则标记任务执行成功，并从状态轮询map中移除
 *       2）如果发现有失败的任务，则汇报当前TaskGroup的状态，并抛出异常
 *       3）查看当前执行队列的长度，如果发现执行队列还有通道，则构建TaskExecutor加入执行队列，并从待运行移除
 *       4）检查执行队列和所有的任务状态，如果所有的任务都执行成功，则汇报taskGroup的状态并从循环中退出
 *       5）检查当前时间是否超过汇报时间检测，如果是，则汇报当前状态
 *       6）当所有的执行完成从while中退出之后，再次全局汇报当前的任务状态
 */
while (true) {
	//1.判断task状态
	boolean failedOrKilled = false;
	Map<Integer, Communication> communicationMap = containerCommunicator.getCommunicationMap();
	for(Map.Entry<Integer, Communication> entry : communicationMap.entrySet()){
		Integer taskId = entry.getKey();
		Communication taskCommunication = entry.getValue();
		if(!taskCommunication.isFinished()){
			continue;
		}
		TaskExecutor taskExecutor = removeTask(runTasks, taskId);

		//上面从runTasks里移除了，因此对应在monitor里移除
		taskMonitor.removeTask(taskId);

		//失败，看task是否支持failover，重试次数未超过最大限制
		if(taskCommunication.getState() == State.FAILED){
			taskFailedExecutorMap.put(taskId, taskExecutor);
			if(taskExecutor.supportFailOver() && taskExecutor.getAttemptCount() < taskMaxRetryTimes){
				taskExecutor.shutdown(); //关闭老的executor
				containerCommunicator.resetCommunication(taskId); //将task的状态重置
				Configuration taskConfig = taskConfigMap.get(taskId);
				taskQueue.add(taskConfig); //重新加入任务列表
			}else{
				failedOrKilled = true;
				break;
			}
		}else if(taskCommunication.getState() == State.KILLED){
			failedOrKilled = true;
			break;
		}else if(taskCommunication.getState() == State.SUCCEEDED){
			Long taskStartTime = taskStartTimeMap.get(taskId);
			if(taskStartTime != null){
				Long usedTime = System.currentTimeMillis() - taskStartTime;
				LOG.info("taskGroup[{}] taskId[{}] is successed, used[{}]ms",
						this.taskGroupId, taskId, usedTime);
				//usedTime*1000*1000 转换成PerfRecord记录的ns，这里主要是简单登记，进行最长任务的打印。因此增加特定静态方法
				PerfRecord.addPerfRecord(taskGroupId, taskId, PerfRecord.PHASE.TASK_TOTAL,taskStartTime, usedTime * 1000L * 1000L);
				taskStartTimeMap.remove(taskId);
				taskConfigMap.remove(taskId);
			}
		}
	}
	
	// 2.发现该taskGroup下taskExecutor的总状态失败则汇报错误
	if (failedOrKilled) {
		lastTaskGroupContainerCommunication = reportTaskGroupCommunication(
				lastTaskGroupContainerCommunication, taskCountInThisTaskGroup);

		throw DataXException.asDataXException(
				FrameworkErrorCode.PLUGIN_RUNTIME_ERROR, lastTaskGroupContainerCommunication.getThrowable());
	}
	
	//3.有任务未执行，且正在运行的任务数小于最大通道限制
	Iterator<Configuration> iterator = taskQueue.iterator();
	while(iterator.hasNext() && runTasks.size() < channelNumber){
		Configuration taskConfig = iterator.next();
		Integer taskId = taskConfig.getInt(CoreConstant.TASK_ID);
		int attemptCount = 1;
		TaskExecutor lastExecutor = taskFailedExecutorMap.get(taskId);
		if(lastExecutor!=null){
			attemptCount = lastExecutor.getAttemptCount() + 1;
			long now = System.currentTimeMillis();
			long failedTime = lastExecutor.getTimeStamp();
			if(now - failedTime < taskRetryIntervalInMsec){  //未到等待时间，继续留在队列
				continue;
			}
			if(!lastExecutor.isShutdown()){ //上次失败的task仍未结束
				if(now - failedTime > taskMaxWaitInMsec){
					markCommunicationFailed(taskId);
					reportTaskGroupCommunication(lastTaskGroupContainerCommunication, taskCountInThisTaskGroup);
					throw DataXException.asDataXException(CommonErrorCode.WAIT_TIME_EXCEED, "task failover等待超时");
				}else{
					lastExecutor.shutdown(); //再次尝试关闭
					continue;
				}
			}else{
				LOG.info("taskGroup[{}] taskId[{}] attemptCount[{}] has already shutdown",
						this.taskGroupId, taskId, lastExecutor.getAttemptCount());
			}
		}
		/**
		 * by 耳东陈
		 * 需要新建任务的配置信息
		 */
		Configuration taskConfigForRun = taskMaxRetryTimes > 1 ? taskConfig.clone() : taskConfig;
		/**
		 * by 耳东陈
		 * taskExecutor应该就需要新建的任务
		 */
		TaskExecutor taskExecutor = new TaskExecutor(taskConfigForRun, attemptCount);
		taskStartTimeMap.put(taskId, System.currentTimeMillis());
		taskExecutor.doStart();

		iterator.remove();
		runTasks.add(taskExecutor);

		//上面，增加task到runTasks列表，因此在monitor里注册。
		taskMonitor.registerTask(taskId, this.containerCommunicator.getCommunication(taskId));

		taskFailedExecutorMap.remove(taskId);
		LOG.info("taskGroup[{}] taskId[{}] attemptCount[{}] is started",
				this.taskGroupId, taskId, attemptCount);
	}

	//4.任务列表为空，executor已结束, 搜集状态为success--->成功
	if (taskQueue.isEmpty() && isAllTaskDone(runTasks) && containerCommunicator.collectState() == State.SUCCEEDED) {
		// 成功的情况下，也需要汇报一次。否则在任务结束非常快的情况下，采集的信息将会不准确
		lastTaskGroupContainerCommunication = reportTaskGroupCommunication(
				lastTaskGroupContainerCommunication, taskCountInThisTaskGroup);

		LOG.info("taskGroup[{}] completed it's tasks.", this.taskGroupId);
		break;
	}

	// 5.如果当前时间已经超出汇报时间的interval，那么我们需要马上汇报
	long now = System.currentTimeMillis();
	if (now - lastReportTimeStamp > reportIntervalInMillSec) {
		lastTaskGroupContainerCommunication = reportTaskGroupCommunication(
				lastTaskGroupContainerCommunication, taskCountInThisTaskGroup);

		lastReportTimeStamp = now;

		//taskMonitor对于正在运行的task，每reportIntervalInMillSec进行检查
		for(TaskExecutor taskExecutor:runTasks){
			taskMonitor.report(taskExecutor.getTaskId(),this.containerCommunicator.getCommunication(taskExecutor.getTaskId()));
		}

	}

	Thread.sleep(sleepIntervalInMillSec);
}

//6.最后还要汇报一次
reportTaskGroupCommunication(lastTaskGroupContainerCommunication, taskCountInThisTaskGroup);
````

### 十三、TaskExecutor：一个完整task的执行器；

````java
/**
 * by 耳东陈
 * 说明：
 * TaskExecutor是一个完整task的执行器
 * TaskExecutor的启动过程主要做了以下事情：
 *
 * 1、创建了reader和writer的线程任务，reader和writer公用一个channel。
 * 2、先启动writer线程后，再启动reader线程。
 * 3、至此，同步数据的Task任务已经启动了。
 */
public TaskExecutor(Configuration taskConf, int attemptCount) {
	// 获取该taskExecutor的配置
	this.taskConfig = taskConf;
	Validate.isTrue(null != this.taskConfig.getConfiguration(CoreConstant.JOB_READER)
					&& null != this.taskConfig.getConfiguration(CoreConstant.JOB_WRITER),
			"[reader|writer]的插件参数不能为空!");

	// 得到taskId
	this.taskId = this.taskConfig.getInt(CoreConstant.TASK_ID);
	this.attemptCount = attemptCount;

	/**
	 * 由taskId得到该taskExecutor的Communication
	 * 要传给readerRunner和writerRunner，同时要传给channel作统计用
	 */
	this.taskCommunication = containerCommunicator
			.getCommunication(taskId);
	Validate.notNull(this.taskCommunication,
			String.format("taskId[%d]的Communication没有注册过", taskId));
	this.channel = ClassUtil.instantiate(channelClazz,
			Channel.class, configuration);
	/**
	 * by 耳东陈
	 * channel在这里生成，每个taskGroup生成一个channel，
	 * 在generateRunner方法当中生成writer或reader并注入channel
	 */
	this.channel.setCommunication(this.taskCommunication);

	/**
	 * 获取transformer的参数
	 */

	List<TransformerExecution> transformerInfoExecs = TransformerUtil.buildTransformerInfo(taskConfig);

	/**
	 * 生成writerThread
	 */
	writerRunner = (WriterRunner) generateRunner(PluginType.WRITER);
	this.writerThread = new Thread(writerRunner,
			String.format("%d-%d-%d-writer",
					jobId, taskGroupId, this.taskId));
	//通过设置thread的contextClassLoader，即可实现同步和主程序不通的加载器
	this.writerThread.setContextClassLoader(LoadUtil.getJarLoader(
			PluginType.WRITER, this.taskConfig.getString(
					CoreConstant.JOB_WRITER_NAME)));

	/**
	 * 生成readerThread
	 */
	readerRunner = (ReaderRunner) generateRunner(PluginType.READER,transformerInfoExecs);
	this.readerThread = new Thread(readerRunner,
			String.format("%d-%d-%d-reader",
					jobId, taskGroupId, this.taskId));
	/**
	 * 通过设置thread的contextClassLoader，即可实现同步和主程序不通的加载器
	 */
	this.readerThread.setContextClassLoader(LoadUtil.getJarLoader(
			PluginType.READER, this.taskConfig.getString(
					CoreConstant.JOB_READER_NAME)));
}
````

### 十四、invokeHooks用法？
目前发现DataX在运行的最后，调用了此方法，还没有搜到相关资料，记录一下日后更新。
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200428/14.1.png"  align=left/>
