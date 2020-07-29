# Spring Framework 学习笔记

## 创建 bean 的细节

### 创建 bean 的三种方式

- 使用默认构造函数创建，类中无默认构造函数则无法创建 bean 对象

  ```xml
  <bean id="accountService" class="com.yartang.service.impl.AccountServiceImpl"/>
  ```

- 使用普通工厂（或某个类）中的方法创建 bean 对象

  ```xml
  <bean id="myFactory" class="com.yartang.factory.MyFactory"></bean>
  <bean id="accountService" factory-bean="myFactory" factory-method="getAccountService"></bean>
  ```

- 使用普通工厂（或某个类）中的静态方法创建 bean 对象

  ```xml
  <bean id="accountService" class="com.yartang.factory.MyFactory" factory-method="getAccountServiceStatic"></bean>
  ```

### bean 对象的作用范围

bean 标签的 scope 属性

作用：指定 bean 对象的生命周期

|   scope   |        Description        |
| :-------: | :-----------------------: |
| singleton |       单例（默认）        |
| prototype |           多例            |
|  request  | 作用于 web 应用的请求范围 |
|  session	| 作用于 web 应用的会话范围|
|global-session|作用于集群环境的会话范围|

```xml
<bean id="accountService" class="com.yartang.service.impl.AccoutnServiceImpl" scope="prototype"></bean>
```

### bean 对象的生命周期

| 单例对象   | 多例对象                                      |
| :----------: |: ---------------------------------------------: |
| 与容器相同 | 使用对象时，对象出生；由 jvm 的 GC 将对象回收 |

示例：

```xml
<beans id="accountService" class="com.yartang.service.impl.AccountServiceImpl" scope="singleton" init-method="init" destroy-method="destroy"></beans>
```

## Spring 的依赖注入 (Dependency Injection)

1. IOC 的作用：降低程序间的耦合（依赖关系）

2. 依赖关系的管理：交给 Spring 维护—当前类需要的对象由 Spring 提供，只需在配置文件中说明

3. 依赖关系的维护：依赖注入

### 依赖注入

#### 依赖注入的数据类型

基本类型和 String，其他 bean 类型（在配置文件中或注解配置过的 bean ），复杂类型/集合类型

#### 依赖注入的方式

##### 使用构造函数提供

使用的标签：*constructor-arg*

标签出现位置：bean 标签内部

标签的属性：

- type: 指定要注入的数据的数据类型，该数据类型也是构造函数汇总某个或者某些参数的类型

- index:  指定要注入的数据给构造函数中指定索引位置的参数赋值，且索引是从 0 开始

- name: 指定构造函数中参数的名称

  **以上三个标签用于指定构造函数中的参数**

- value:  用于给基本类型或者 String 类型赋值

- ref：用于给其他的 bean 类型赋值

示例：

```xml
<!-- 使用构造器依赖注入示例 -->
<bean id="accountService" class="com.yartang.service.impl.AccountServiceImpl">
    <constructor-arg name="id" value="1"></constructor-arg>
    <constructor-arg name="name" value="123"></constructor-arg>
    <constructor-arg name="date" ref="date"></constructor-arg>
</bean>
<!-- 调用默认构造函数，所以打印当前时间 
<bean id="date" class="java.util.Date"></bean> -->
<!-- 可以使用构造函数，改变 bean 对象属性 -->
<bean id="date" class="java.util.Date">
        <constructor-arg name="date" value="1000000000000"></constructor-arg>
</bean>
```

特点：

- 在获取 bean 对象时，必须注入数据，否则无法创建对象；
- 改变了 bean 对象的实例化方式，创建对象时，用不到某些数据，也必须提供

##### 使用 set 方法提供

使用的标签：property

标签出现位置：bean 标签内部\

标签的属性：

- name: 注入时调用的 set 方法名称 
- value:  用于给基本类型或者 String 类型赋值
- ref：用于给其他的 bean 类型赋值

示例;

```xml
<bean id="accountService" class="com.yartang.service.impl.AccountServiceImpl">
    <property name="name" value="123"/>
    <property name="date" ref="date"/>
</bean>

<bean id="date" class="java.util.Date">
    <constructor-arg name="date" value="1000000000000"></constructor-arg>
</bean>
```

特点：

- 创建对象没有明确的限制，直接使用默认构造函数；
- 如果某个类成员必须有值，无法保证该成员有值

##### 使用注解提供

##### 复杂类型/集合类型 注入

## AOP

- 作用：在运行期间，不修改源码增强已有方法
- 优势：减少重复代码， 提高开啊效率，维护方便
- 术语：
  * 连接点 ( Joinpoint ) ：连接业务方法和增强方法的点，通过代理增强业务方法，使业务形成完整的逻辑
  * 切入点 (Pointcut) ：被增强的业务方法
  * 通知 / 增强 (Advice) ：
  * 切面 (Aspect) ：切入点和通知的结合

### 配置AOP

1. 将通知 Bean 也交给 Spring 配置

2. 使用 aop:config 表明开始配置

3. 使用 aop:aspect 配置切面

   - id : 给切面提供唯一的标识
   - ref : 指定通知类的 Bean 的id

4. 在 aop:aspect 内部配置通知的类型

   * aop:before / aop:after-returning / aop:after-throwing / aop:after

     * method : 指定通知类中的某个方法为前置通知方法

     * pointcut : 用于指定切入点的表达式，指定业务层中需要增强的方法

       切入点表达式：execution( 表达式 )

       访问修饰符 返回类型 全限定类名.方法名( 参数列表 )

       **实际开发的写法**  * com.yartang.service.impl. * . *(..)

       实例：

       ```java
       public void com.yartang.service.impl.AccountServiceImpl.saveAccount()
       ```

   - 示例：

    ```xml
   <aop:config>
       <aop:aspect id="logAdvice" ref="logger">
            <aop:before method="printLog" pointcut="execution(public 	void com.yartang.service.impl.AccountServiceImpl.saveAccount())"/>
       </aop:aspect>
   </aop:config>
    ```

   - aop:pointcut 指定切入点表达式：

     - 在 aop:aspect 内时，只能当前 aspect 能使用

     - 在 aop:aspect 外时， 都能使用

       ```xml
       <aop:pointcut id="pt1" expression="execution(public void com.yartang.service.impl.AccountServiceImpl.saveAccount())"/>
       ```

   - 环绕通知：相当于在代码段中配置通知的位置

     ```xml
     <aop:around method="aroundPrintLog" pointcut-ref="pt1"/>
     ```

     ```java
     public Object aroundPrintLog(ProceedingJoinPoint pjp) {
             Object rs = null;
             try  {
                 Object[] args = pjp.getArgs();
                 System.out.println("环绕通知-前置");
                 rs = pjp.proceed(args);
                 System.out.println("环绕通知-后置");
                 return rs;
             } catch (Throwable t) {
                 System.out.println("环绕通知-异常");
                 throw new RuntimeException(t);
             } finally {
                 System.out.println("环绕通知-最终");
             }
         }
     ```

## Spring 事务控制

### 步骤

1. 配置事务管理器

   ```xml
   <bean id="transctionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"/>
   ```

2. 配置事务通知

   

3. 配置 AOP 的切入点表达式 