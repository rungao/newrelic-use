## NewRelic监控


[TOC]

### 监控介绍

1. APM 

   应用性能管理（Application Performance Management），主要针对具体业务进行监测、优化，提高应用的可靠性与质量,APM提供商主要有听云、OneAMP、NewRelic等。

2. NewRelic公司

   New Relic 公司(创始人和CEO的名字)，美国旧金山一家创业公司，主要做做软件分析，深入到应用程序内部，告诉你在产品的表象看不到的事情，2008年创建，两年后被3.75亿美元收购APM业务，15年2月已经超过1.1亿用户，目前公司在纽交所已经上市。

3. APM能监控什么？

   区别于service的监控，APM更关注的是业务性能的监控，如网址开发语言使用的是PHP，可监控PHP的执行时间、错误率、使用的MYSQL、Redis、Monogdb分析，web的执行时间等。

   - 服务性能监控
   - 数据库性能监控管理（Redis、Monogdb、Mysql）
   - HTTP状态及错误分析
   - 异常捕获
   - 页面监控
   - 数据报表
   - 准备定位问题，降低MTTR

4. NewRelic提供的服务

   - AMP监控（支持PHP、GO、Nodejs、Java、Python等），主要有脚本分析、错误捕获、

     数据库（Mysql、Redis、Monogdb）性能分析、外部流量、CPU内存分析等

   - BROWSER监控（页面访问时间、JS错误、浏览器分布、地区分析等）

   - SERVERS监控（CPU、内存、进程、网络、磁盘），支持邮箱、短信设置报警

###安装监控

**以下监控安装脚本均以centos系统为例，其他系统请查看官方文档**

1. 安装PHP扩展监控

   ```shell
   sudo rpm -Uvh http://yum.newrelic.com/pub/newrelic/el5/x86_64/newrelic-repo-5-3.noarch.rpm
   sudo yum install newrelic-php5
   sudo newrelic-install install 
   #注意php需要预先加入到系统环境变量中 ln -s /opt/app/php5/bin/php /usr/bin/
   #输入license
   vim /opt/app/php/etc/php.ini
   newrelic.enable = true;
   newrelic.license = xxxx;
   #不设置多台机器不会显示多个
   newrelic.appname = xxx; 
   #检查扩展是否安装成功
   php -m | grep newrelic
   #重启apache/nginx服务
   #重启服务后去AMP后台看到项目名称后即可，后续改PHP相关的所有执行脚本都会被监控且上报到AMP平台
   ```

2. 安装SERVICES监控

   ```shell
   rpm -Uvh https://download.newrelic.com/pub/newrelic/el5/i386/newrelic-repo-5-3.noarch.rpm
   yum install newrelic-sysmond
   nrsysmond-config --set license_key=87a9b40ecc504523aede1899c8efa072572ffed7
   /etc/init.d/newrelic-sysmond start
   # 编辑监控机器名称
   vim /etc/newrelic/nrsysmond.cfg
       ;hostname = xxx
   #加入到开机自启动
   vim /etc/rc.local 
   /etc/init.d/newrelic-sysmond start
   ```

   ​

### APM使用

#### 1. 名词说明

- labels 对服务器列表设置Categroty:Label，能实现类似监控平台的分组功能，提供搜索功能，服务器多可使用此功能
- page views ppm：页面每分钟的综合访问量
- throughput rpm：吞度量
- errors：应用的平均错误率（mysql错误、php错误、redis错误等）
- apdex：应用程序性能指数，默认为0.5，表示小于0.5s满足，0.5-2.0可容忍，>2.0不能接受，在监控页面可自行配置
- cpm：

#### 2. overview概览页面

- web transactions time 应用程序执行时间图，可根据php/mysql/externel services/redis筛选，可以根据日期筛选

- apdex score 应用程序平均得分

- error rate 错误率分析

- throughout 吞吐量分析

- recent events 最新事件（服务器停机、服务器报警、应用错误、应用报警、说明、部署等）

**注：默认显示的最近30分钟内的信息，如果未发现错误需要自行调整筛选时间，推荐进入系统后先把时间选成一天查看**

#### 3. transactions 脚本执行分析

- web事件分析（请求php次数分组，rpm分析），表格模式能详细展示每个php的请求时间、平均请求时间
- no-web事件分析

#### 4. databases数据库监控

支持redis/monogdb/redis监控，自动收集业务中使用到的database并记录，mysql支持记录慢日志、错误日志，支持对topN的语句进行分组浏览

该页面主要关注mysql、redis的使用是否有问题，是否有语句查询次数过多？查询语句太慢？有哪些表有慢查询等。

#### 5.  External services外部访问监控

内部应用访问外部API的接口监控，如访问50bang接口、抓取业务等，可对所有的接口域名、IP分组浏览，按请求时间平均排序，可自动定位到源脚本目录。

该页面主要是针对外部API接口连通率及访问时间进行监控，可发现接口访问慢，长期有问题的接口。

#### 6. error analytics错误分析

该功能是APM中最重要的功能，以上的功能都是为了提高业务质量，而错误分析是为了解决业务中潜在的错误和bug。

##### 错误分析分类#####

- MysqlError :  MYSQL语句执行不成功的错误记录，非常重要，所有的错误都必须解决
- E_WARNING : PHP WARNING错误，可根据具体错误解决，此类错误建议全部解决
- Error : PHP 错误，必须解决
- E_ERROR : 错误，必须解决
- E_COMPLE_ERROR: 编译错误，必须解决
- Exception : 应用程序报错，可针对解决

#####错误解决#####

错误可根据相关信息进行分组，支持按错误类型、主机、HTTP CODE、脚本名称、UA等信息进行分组流程，根据分组可分析此类错误可能引起的原因。错误支持自动定位到PHP相关文件，且有详细的PHP异常记录信息。

**注意**：

1. 使用MYSQL驱动的默认产生的SQL会被记录，因mysql_query发送错误会产生一条E_WARGING错误
2. 使用PDO-MYSQL的必须设定错误类型为ERMODE_WARNING或ERRMODE_EXCEPTION，默认模式发送错误后不会产生任何错误，除非框架自行记录，newrelic不会记录。

#### 7. Services maps

服务器架构图，提供在线创建/编辑架构图，系统根据使用的服务列表和外部服务，构建简单的服务器拓扑图。

#### 8.REPORTS报表

- SLA报表，提供每日、每周、每月的pageview/loadtime/apdex的报表，并提供下载功能

- availability可用性监控，预先添加一个域名，系统自动进行可用性监控

- scalability可扩展性分析，提供响应时间、CPU应用、DATABASE的吞吐量分析，可按时段进行分析，此功能可分析不同时段业务系统的运行情况

- web transactions report网页事件报告，按时间和脚本名称分析网页加载时间、平均加载时间、apdex评分，可搜索相关的脚本名称，此功能可分析一个请求连接在不同时段的运行情况

- database 数据库相关的性能分析报告，根据时间、表信息分析CURD的性能情况

- background jobs 命令行脚本的执行运行情况分析

####8. Services maps

- Envionment 该系统的环境变量情况，agent扩展的安装及参数配置浏览

  ​

### Servers监控

1. 查看某段时间的Load average系统负载（对当前CPU工作情况的度量，系统负载根据CPU内核决定，如8内核CPU，如果超过8则表示系统运行的队列已满发生排队和拥堵，可使用top/w/uptime查看）

2. 查看某段时间的CPU利用情况

3. 查看某段时间的物理内存、虚拟内存利用

4. 查看某段时间的磁盘IO情况

5. 查看某段时间的网络吞吐量

6. 查看系统进程运行列表，进程占用资源分析，可根据时间查看历史运行情况

7. 查看系统磁盘的使用

与监控系统不同的是该监控支持了进程列表的监控，且能查看历史时间的进程的CPU/MEM的使用情况。



### BROWSER监控

1.  page views页面加载时间

2.  js errors js错误分析，pro版本支持

3.  browser浏览器分析，大部分的web分析都有该功能，区别是此分析是从server端收集的数据

4.  Geo地理分析，国内的支持的不是很好

5.  session traces，pro版本支持 

####applaction settings功能

1. copy系统提供的js代码放在业务需要监控的网页html的头部或者底部，类似于cnzz统计

2. 设置是否开启browser-apm监控，如果是pro版本则可支持browser所有监控

   ​

### SYNTHETICS监控

#### ping监控

免费功能，添加一个域名或者IP地址，选择监控点（国际监控点，对国内的电信/联通目前支持不好），选择监控时间和报警邮件后即可，该功能类似于监控宝之类的业务

#### simple browser模拟浏览监控

模拟浏览器访问并生成访问数据，14天免费试用期限

#### scripted browser 完成的浏览器脚本测试

完整模拟浏览器访问，并生成报告，14天的免费试用期限

####　API接口监控

类似于URL监控，针对API接口的信息进行校验监控,创建监控后可自定义写脚本（nodejs），该功能可以定制化监控线上业务的访问情况，如浏览器名单数据接口是否有问题

### MOBILE监控



### PLIUGINS插件



### 应用监控

#### servers监控

默认当 CPU>80%大于15分钟， DISK IO >90%大于15分钟， MEM > 90%大于5分钟， DISK USE > 90%大于5分钟 ，系统将发生报警，报警支持eamil/mobile组合方式告知

多服务器管理：

1. 新建邮件告知组（channels and groups），将相同业务的人员分组方便邮件告知
2. 新建server policy分组，将机器分到不同的分组中，设置自定义的告警信息以及可自定义发送到指定人的邮箱中

#### 应用监控

默认error rate > 5%大于3分钟，apdex > 0.7 5分钟系统将发生报警，报警告知方式支持email/mobile组合方式

同样应用监控也支持emial分组，自定义选择机器分组



### 接下来需要做的

1. 将所有PDO实例化类增加setAttribute(ATTR_ERRMODE,  ERRMODE_WARNING)-子龙负责

2. 检查测试环境与线上环境是否代码一致

3. 处理测试环境中的报错信息

   （泽峰：安全卫士63，步青：64、107和111，子龙：65和66）

4. 我们部门先试用一个月，遇到报错信息处理并记录