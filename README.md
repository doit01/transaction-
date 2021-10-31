https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative-applying-more-than-just-tx-advice


hibernate.connection.handling_mode property toDELAYED_ACQUISITION_AND_RELEASE_AFTER_STATEMENT

解决的方法就简单了（两种）：

    把这两个方法分开到不同的类中；
    把注解加到类名上面
    
    
TransactionDefinition 事务的一些基础信息，如超时时间、隔离级别、传播属性等
TransactionStatus 事务的一些状态信息，如是否一个新的事务、是否已被标记为回滚
public class DefaultTransactionDefinition implements TransactionDefinition, Serializable {
    private int propagationBehavior = PROPAGATION_REQUIRED;
    private int isolationLevel = ISOLATION_DEFAULT;
    private int timeout = TIMEOUT_DEFAULT;
    private boolean readOnly = false;
    //略
}
声明式事务管理建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。最细粒度只能作用到方法级别

@Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义
虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， @Transactional 注解应该只被应用到 public 方法上，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常

默认情况下，只有来自外部的方法调用才会被AOP代理捕获，也就是，类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使用@Transactional注解进行修饰。

明确几点：

    默认配置下 Spring 只会回滚运行时、未检查异常（继承自 RuntimeException 的异常）或者 Error
    @Transactional 注解只能应用到 public 方法才有效

异常的分类:

@Transactional 默认回滚 运行时异常和error,如果想回滚所有的异常，应该加上 rollbackFor = Throwable.class

@Transactional(rollbackFor=Throwable.class)，如果类加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

在@Transactional注解中如果不配置rollbackFor属性,那么事物只会在遇到RuntimeException的时候才会回滚,加上rollbackFor=Exception.class,可以让事物在遇到非运行时异常时也回滚

 
https://images2018.cnblogs.com/blog/1416523/201807/1416523-20180716145710583-1342649521.png
@Transactional注解的全部属性详解

@Transactional属性
  属性	类型	描述
value 	String 	可选的限定描述符，指定使用的事务管理器
propagation 	enum: Propagation 	可选的事务传播行为设置
isolation 	enum: Isolation 	可选的事务隔离级别设置
readOnly 	boolean 	读写或只读事务，默认读写
timeout 	int (in seconds granularity) 	事务超时时间设置
rollbackFor 	Class对象数组，必须继承自Throwable 	导致事务回滚的异常类数组
rollbackForClassName 	类名数组，必须继承自Throwable 	导致事务回滚的异常类名字数组
noRollbackFor 	Class对象数组，必须继承自Throwable 	不会导致事务回滚的异常类数组

noRollbackForClassName 	类名数组，必须继承自Throwable 	不会导致事务回滚的异常类名字数组
