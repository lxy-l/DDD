# Domain-Driven Design 理解
>对于领域驱动设计思想的学习与理解。

---
## DDD适用于以下场景:
1. 业务较为复杂，模块概念较多，需要清晰地划分业务边界和解耦业务。
2. 微服务边界划分困难，需要按照领域模型和限界上下文进行拆分。
3. 技术异构的功能，需要按照技术边界进行拆分。
   

## DDD有以下优点和缺点：
### 优点：
- DDD可以更好地理解和沟通业务需求，建立丰富的领域模型和统一的领域语言。
- DDD可以更快地拆分微服务，实现系统架构适应业务的快速变化。
- DDD是一套完整而系统的设计方法，能够从战略设计到战术设计提供标准的设计过程。
- DDD可以降低服务的耦合性，提高软件的质量和可维护性。
- DDD善于处理高复杂度的业务问题，有利于领域知识的传递和传承。
### 缺点：
- DDD需要花费更多的时间和精力去分析和建模领域，可能会影响开发效率。
- DDD需要有较高的业务抽象能力和面向对象编程能力，对开发者要求较高。
- DDD可能会导致过度设计或过度封装，使得代码难以阅读或修改。
---
## 战略设计
>战略设计是从宏观的角度对业务进行领域划分和构建领域模型，梳理出核心域、子域、限界上下文和统一的领域语言。
>对业务进行领域划分，构建领域模型，梳理出限界上下文，聚合，实体，值对象  
>一个领域就是一个问题空间，我们在业务中所遇到的所有的问题与挑战
### 建模方法
1. 四色建模
2. 限界笔纸
3. 事件风暴
4. 用户故事

---
## 战术设计
>战术设计是从微观的角度对领域模型进行细化和实现，梳理出聚合、实体、值对象、工厂、仓储、服务、事件等概念，并定义它们的属性和行为。
>以领域模型为基础，以限界上下文作为微服务划分的边界进行微服务拆分，实现对领域模型对于代码的映射   
>将战略设计进行具体化和细节化，它主要关注的是技术层面的实施
>一个领域就算一个解决问题空间，用来解决在问题空间的所有问题；
```C#
// 使用code blocks语法来封装长格式内容
public class Order {
  // 订单ID，唯一标识一个订单
  private String orderId;
  // 订单状态，枚举类型
  private OrderStatus status;
  // 订单总金额，值对象
  private Money totalAmount;
  // 订单项列表，值对象集合
  private List<OrderItem> items;
  
  // 构造函数，创建一个新的订单
  public Order(String orderId, List<OrderItem> items) {
    this.orderId = orderId;
    this.status = OrderStatus.CREATED; // 初始状态为创建
    this.items = items;
    this.totalAmount = calculateTotalAmount(); // 根据订单项计算总金额
  }
  
  // 计算订单总金额的方法，返回一个Money值对象
  private Money calculateTotalAmount() {
    Money total = new Money(0); // 初始金额为0
    for (OrderItem item : items) { // 遍历每个订单项
      total = total.add(item.getSubtotal()); // 累加每个订单项的小计金额
    }
    return total;
  }
  
  // 支付订单的方法，改变订单状态为已支付，并触发支付事件（省略具体实现）
  public void pay() {
    if (status == OrderStatus.CREATED) { // 只有创建状态的订单才能支付
      status = OrderStatus.PAID; // 改变状态为已支付
      publishPaymentEvent(); // 发布支付事件（省略具体实现）
    } else {
      throw new IllegalStateException("Only created order can be paid"); // 抛出非法状态异常
    }
    
  }
  

```

---


## 分层

### Domain 领域层
>DDD概念中的核心业务层，封装所有业务逻辑，包含entity、value object、domain service、domain event等。

1. Entity(Reference Object):由标识定义的实体。
2. Value Object：描述了一个事务的某种特征。用于描述领域的某个方面而本身没有概念标识的对象（值对象）实例化之后表现一些设计元素，对于这些元素，只关心他是什么，而不关心他是谁。当我们关心一个模型的属性时，应把他归类为Value Object。他是不可变得，不需要任何标识
3. Aggregate Root: 聚合是用来封装真正的不变性，而不是简单的将对象组合在一起；
				聚合应尽量设计的小；
				聚合之间的关联通过ID，而不是对象引用；
				聚合内强一致性，聚合之间最终一致性；
4. DomainService:领域层中的服务，负责检查是否满足临界值。例如银行转账功能。应属于领域服务，因为它包含重要业务逻辑。
		1.与必要的账户和总账对象进行交互，执行相应的借入贷出操作
		2.提供结果确认（允许或拒绝）

1.聚合根、实体、值对象的区别？

	从标识的角度：

	聚合根具有全局的唯一标识，而实体只有在聚合内部有唯一的本地标识，值对象没有唯一标识，不存在这个值对象或那个值对象的说法；

	从是否只读的角度：

	聚合根除了唯一标识外，其他所有状态信息都理论上可变；实体是可变的；值对象是只读的；

	从生命周期的角度：

	聚合根有独立的生命周期，实体的生命周期从属于其所属的聚合，实体完全由其所属的聚合根负责管理维护；值对象无生命周期可言，因为只是一个值；

2.聚合根、实体、值对象对象之间如何建立关联？

	聚合根到聚合根：通过ID关联；

	聚合根到其内部的实体，直接对象引用；

	聚合根到值对象，直接对象引用；

	实体对其他对象的引用规则：1）能引用其所属聚合内的聚合根、实体、值对象；2）能引用外部聚合根，但推荐以ID的方式关联，另外也可以关联某个外部聚合内的实体，但必须是ID关联，否则就出现同一个实体的引用被两个聚合根持有，这是不允许的，一个实体的引用只能被其所属的聚合根持有；

	值对象对其他对象的引用规则：只需确保值对象是只读的即可，推荐值对象的所有属性都尽量是值对象；

3.如何识别聚合与聚合根？

	明确含义：一个Bounded Context（界定的上下文）可能包含多个聚合，每个聚合都有一个根实体，叫做聚合根；

	识别顺序：先找出哪些实体可能是聚合根，再逐个分析每个聚合根的边界，即该聚合根应该聚合哪些实体或值对象；最后再划分Bounded Context；

	聚合边界确定法则：根据不变性约束规则（Invariant）。不变性规则有两类：1）聚合边界内必须具有哪些信息，如果没有这些信息就不能称为一个有效的聚合；2）聚合内的某些对象的状态必须满足某个业务规则；

### Infrastructure 基础设施层
>提供公共组件，如：Logging、Trascation、HttpClient,ORM等。

### Application 应用层   
>ApplicationService应该永远返回DTO而不是Entity
>		1.构建领域边界
>		2.降低规则依赖
>		3.通过DTO组合降低成本
	
1. 用来封装业务逻辑
2. 面向用例。
3. 粗粒度。
4. 外部视图看系统。
5. 一个请求对应一个方法。
6. 服务之间不相互调用。
7. 职责一般包括：跨模块协调、DTO转换、事务AOP、权限AOP、日志AOP、异常AOP、邮件、消息队列。
8. 组合多个业务实体、基础设施层的各种组件完成业务服务
Service：
9. 获取输入。
10. 发送消息给领域服务，要求执行
11. 监听确认消息
12. 决定使用基础设施层Service来发送通知

### WebApi/MVC 展现层
>对外提供各种协议形式的服务，并提供Validation参数校验，authenticate权限认证，业务实体组装器Assembler等。




---
# 总结
 1. 领域模型由领域专家和开发人员交流建模

 2. 应该领域层调用应用层。认为领域层是被驱动调用的，还是静态数据驱动思维。

 3. 战略设计和战术设计是相辅相成的，战略设计为战术设计提供了指导和范围，战术设计为战略设计提供了反馈和验证