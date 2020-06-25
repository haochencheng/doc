### Spring 事务
####    什么是事务？
事务是逻辑上的一组操作，要么都执行，要么都不执行.
####    事物的特性（ACID）：
1. 原子性
-  事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
2. 一致性
- 执行事务前后，数据保持一致
3. 隔离性
- 并发访问数据库时，一个用户的事物不被其他事物所干扰，各并发事务之间数据库是独立的；
4. 持久性
- 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

###     Spring事务管理接口：

- PlatformTransactionManager： （平台）事务管理器
- TransactionDefinition： 事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)
- TransactionStatus： 事务运行状态

### 事物属性
1. 隔离级别
2. 传播行为
3. 回滚规则
4. 是否只读
5. 事物超时

####    TransactionDefinition 事物定义接口

其中定义了事物传播属性Propagation和事物隔离级别Isolation

```
public interface TransactionDefinition{

    int getIsolationLevel();    // 返回事务的传播行为
    int getPropagationBehavior();  // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getTimeout();     // 返回事务必须在多少秒内完成
    boolean isReadOnly();   // 返回是否优化为只读事务。
}
```

####    事务隔离级别 isolation
- TransactionDefinition.ISOLATION_DEFAULT
 

    这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。

- TransactionDefinition.ISOLATION_READ_UNCOMMITTED


    该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用该隔离级别。
    

- TransactionDefinition.ISOLATION_READ_COMMITTED


    该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
    

- TransactionDefinition.ISOLATION_REPEATABLE_READ


    该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
    
- TransactionDefinition.ISOLATION_SERIALIZABLE


    所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。
    
####    事务传播行为
所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：

- TransactionDefinition.PROPAGATION_REQUIRED


    如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
    
- TransactionDefinition.PROPAGATION_REQUIRES_NEW


    创建一个新的事务，如果当前存在事务，则把当前事务挂起。
    
- TransactionDefinition.PROPAGATION_SUPPORTS


    如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
    
- TransactionDefinition.PROPAGATION_NOT_SUPPORTED


    以非事务方式运行，如果当前存在事务，则把当前事务挂起。

- TransactionDefinition.PROPAGATION_NEVER


    以非事务方式运行，如果当前存在事务，则抛出异常。
    
- TransactionDefinition.PROPAGATION_MANDATORY


    如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常
    
- TransactionDefinition.PROPAGATION_NESTED


    如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。
    
####    事务超时
所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。

####    并发事务带来的问题
- 脏读（Dirty read）:       
当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。


- 丢失修改（Lost to modif y）       
指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。
例如：事务1读取某表中的数据A=20，事务2也读取A=20，事务1修改A=A-1，事务2也修改A=A-1，最终结果A=19，事务1的修改被丢失。


- 不可重复读（Unrepeatableread）        
指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。


- 幻读（Phantom read）       幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

####    不可重复度和幻读区别：

不可重复读的重点是修改，幻读的重点在于新增或者删除。



    