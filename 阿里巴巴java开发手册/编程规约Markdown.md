##### 第一章 编程规约
- 慎用Object的clone方法:因为他是浅拷贝,若想要实现深拷贝,需要重写clone方法来实现属性对象的拷贝.
- 类成员与方法访问控制从严. 对任何类,方法,参数,变量,严控访问范围.过于宽泛的访问范围,不利于模块解耦.
##### 集合处理
1. 关于hashcode和equals方法.
   1. 只要重写equals方法,就必须重写hashcode方法
   2. 如果自定义对象作为map的key,那么必须重写hashcode和equals方法.
2. 在使用工具类Arrays.asList()把数组转换成集合时,不能使用其修改集合相关的方法,它的add/remove/clear方法会抛出unsupportedOperationException异常.
> 说明: asList返回对象是一个arrays内部类,并没有实现集合的修改方法.Arrays.asList体现的是适配器模式,只是转换接口,后台的数据仍是数组.
改变数组的值,集合中的值也会随之更改.
3. 关于泛型???
4. 不要在foreach循环里进行元素的remove/add操作.remove元素请使用Iterator方式,如果并发操作,需要对Iterator对象加锁.
5. 集合初始化时,指定集合初始值的大小.
###### 说明:hashmap使用HashMap(int initialCapacity)初始化.
initialCapacity = (需要存储的元素个数/负载因子)+1.负载因子()默认为0.75.如果暂时无法确定初始值的大小,请设置默认值(16).
6. map的遍历,用entryset,jdk8用map.foreach()方法.
##### 并发处理
1. 线程资源必须通过线程池来提供,不允许在应用中自行显示创建线程.
- 说明:使用线程池的好处是减少在创建和销毁线程上所消耗的时间及系统资源,解决资源不足的问题.如果不使用线程池,有可能造成系统创建大量同类线程而导致消耗完内存或者"过度切换"的问题.
2. 线程池不允许使用Executors创建,而是通过ThreadPoolExecutor的方式创建,这样的处理方法能让编写的工程师更加明确线程池的运行规则,规避资源耗尽的风险.
- 说明:Executors返回的线程池对象的弊端如下:
 - FixedThreadPool和SingleThreadPool:允许的请求队列的长度为Integer.Max_Value,可能会堆积大量的请求,从而导致OOM
- CachedThreadPool和ScheduledThreadPool:允许创建的线程数量为Integer.MAX_VALUE,可能会创建大量的线程,从而导致OOM.
3. SimpleDateFormat 是线程不安全的类,一般不要定义static变量,如果定义为static,必须加锁,或者使用DateUtils工具类.
- 注意线程安全,使用DateUtils.<br>
    private static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>(){
    	@Override
        protected DateFormat initialValue () {
        	return  new SimpleDateFormat("yyyy-MM-dd");
        }
    };<br>
- 说明:jdk8 可以用Instant代替Date,LocalDateTime代替calendar,datetimeformatter代替simpledateformat.
4. 在高并发场景中,同步调用应该去考量锁的性能损耗.能用无锁表结构,就不用锁;能锁区块,就不要锁整个方法;能用对象锁,就不要用类锁.
- 使加锁的代码块工作量尽可能小,避免在锁代码块中用RPC方法.
5. 在对多个资源,数据库表,对象同时加锁时,需要保持一致的加锁顺序,否则可能会造成死锁现象.
6. 在并发修改同一记录时,为避免更新丢失,需要加锁.要么在应用层加锁,要么在缓存层加锁,要么在数据层使用乐观锁,使用version作为更新数据.
- 如果每次访问冲突概率小于20%,推荐使用乐观锁,否则使用悲观锁.乐观锁的重试次数不得小于三次.
7. 对于多线程并行处理定时任务的情况,在Timer运行多个TimeTask时,只要其中之一没有捕获抛出的异常,其他任务便会自动停止运行.如果在处理定时任务时,使用ScheduledExecutorService,则没有这个问题.
8. 使用countdownlatch进行异步转同步操作,每个线程退出前必须调用countdown方法,线程执行代码注意catch异常,确保countdown方法被执行,避免主线程无法执行至await方法,知道超时才返回结果.
- 子线程抛出异常堆栈,不能在主线程try-catch到.
9. 避免Random实例被多线程使用,虽然共享该实例是线程安全的,但是会因竞争同一seed导致性能下降.
- Random实例包括Java.util.random的实例或者math.random()的方式
- JDK7之后可以直接用API ThreadLocalRandom.
10. 在并发场景下,通过双重检查锁(double-checked-locking)实现延迟初始化的优化问题隐患,推荐解决方案中较为简单的一种,目标属性声明volatile型.
11. volatie解决多线程内存不可见问题,对于一写多读,可以解决变量同步问题,但是如果多写,同样无法解决线程安全问题.如果是count++操作,使用如下类实现:AtomicInteger count = new AtomicInteger();count.addAndGet(1);如果是jdk8,推荐使用Longadder对象,他比atomiclong性能更好,减少乐观锁的重试次数.
12. HashMap在容量不够进行resize时,由于高并发可能出现死链,导致cpu占用飙升,在开发过程中使用其他数据结构或加锁来规避此风险.
13. ThreadLocal无法解决共享对象的更新问题,ThreadLocal对象建议使用static修饰.这个变量是针对一个线程内所有操作共享的,所以设置为静态变量,所有此类实例共享此静态变量,也就是说,在类第一次被使用时装载,只分配一块存储空间,所有此类的对象(只要是这个线程内定义的)都可以操控这个变量.
14. 控制语句:在高并发场景下,避免使用"等于"判断作为中断或退出的条件.
- 说明:如果并发控制没有处理好,容易产生等值判断被"击穿"的情况,应使用大于或小于的区间判断条件来代替.
- 超过三层的if else逻辑判断代码可以使用卫语句,策略模式,状态模式等来实现.卫语句就是把复杂的条件表达式拆分成多个条件表达式,条件为真时,立刻从方法体重返回给调用方.卫语句的好处是条件表达式之间互相独立,不会干扰.
15.注释:特殊注释,待办事宜(TODO),错误(FIXME).
16.获取当前毫秒数用System.currentTimeMills().不用new Date().getTime();
#### 异常日志
1. java类库中定义的可以通过预检查方式规避RuntimeException不应该通过catch方式处理,
2. 异常不要用来做流程控制,条件控制,不符合一场涉及的初衷:解决运行程序中的各种意外情况,况且异常的处理效率比条件判断方式要低很多.
3. catch时情分请稳定代码和非稳定代码.对于稳定代码不catch处理,对于非稳定代码cat.对大段代码进行try-catch,将使程序无法根据不同的异常做出正确的反应,不利于定位问题,是一种不负责任的表现.
4. 不能再finally块中使用return.
5. 定义区分unchecked/checked异常,避免直接抛出new RuntimeException(),更不允许抛出Exception或者Throwable,应使用具有业务含义的自定义异常.推荐业界已经定义过的自定义异常,如DAOException/ServiceException等.
6. 避免出现重复的代码.
#### 日志规约
1. 应用中日志的命名方式要注意.最好能通过日志文件名就能知道是哪个类型,有什么目的,便于查找.
2. 对trace/debug/info级别的日志输出必须用条件输出形式或者站位符方式.避免string拼接.隐形的toString方法.
3. 避免重复打印日志,必须配置addtivity=false.
4. 记录日志时请思考:这些日志真的有人看吗,看到这些日志能做什么,能不能给问题排查带来好处.
5. 可以使用warn日志级别记录用户输入参数错误的情况,避免当用户投诉时无所适从.,error日志只记录系统逻辑出错,异常等重要的错误信息.
#### 单元测试
- AIR原则:A:automatic,independent,repeatable
- BCDE原则:
    1. Border边界值测试,循环边界,特殊取值,特殊事件点,数据顺序等.
    2. Correct,正确的输入,并得到预期结果
    3. Design,与设计文档相结合,来编写单元测试.
    4. Error,强制错误信息输入(如非法数据,异常流程,非业务允许输入等.)并得到预期的结果.
#### MYSQL数据库
1. 主键索引用PK_字段名,唯一索引用uk_字段名,普通索引用idx_字段名.
2. 如果存储的字符串长度几乎相等,则应使用char定字符串类型.
3. 页面搜索严禁左模糊或者全模糊,如果需要请通过搜索引擎来解决.
- 索引文件具有B-Tree的最左前缀匹配特性,如果左边的值未确定,那么就无法使用此索引.
