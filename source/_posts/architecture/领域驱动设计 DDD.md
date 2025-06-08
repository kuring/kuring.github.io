---
permalink: /architecture/ddd/
date: 2025-06-08
title: 领域驱动设计 DDD
---
# 1 核心概念
## 1.1 战略设计
从业务视角出发，建立业务领域模型，划分领域边界，建立通用语言的限界上下文，限界上下文可以作为微服务设计的参考边界。
战略设计主要流程包括：建立统一语言、领域分解、领域建模。
## 1.2 战术设计
从技术视角出发，侧重于领域模型的技术实现，完成软件开发和落地。
### 1.2.1 核心概念
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20250608163549.png)
### 1.2.2 实体
具有唯一标识符 ID 和生命周期的对象。状态会随时间变化，但标识符不会变化。实体通常会封装跟自身状态相关的核心业务逻辑和不变规则。

### 1.2.3 值对象
描述事物某些特征或者属性的对象。
具有如下特点：
1. 无标识。
2. 不可变。一旦创建后属性值不能再改变。
3. 通常用来描述实体的属性，比如 `Customer` 实体有一个 `Address` 值对象。
4. 可组合。值对象可以包含其他的值对象。`Address` 可能包含 `Street`, `City`, `ZipCode` 等更小的值对象。

**例子：** `Money` (金额 + 货币), `Address` (街道、城市、邮编), `Color` (RGB值), `DateRange` (开始日期 + 结束日期), `ProductInfo` (产品ID、名称、单价 - 用于订单项中)。

### 1.2.4 **聚合 & 聚合根**

聚合将一组**强关联**的实体和值对象**组合在一起，形成一个一致性边界和一个事务边界**。每个聚合有一个**聚合根**。

**聚合根：**是聚合的**唯一入口点**，通常是一个实体。


例子：
`Order` (聚合根) 包含：
- 自身属性 (订单号、下单时间、状态、总金额)
- `OrderItem` (实体) 列表 - 每个订单项有自己的 `ProductId`, `ProductName`, `UnitPrice`, `Quantity`, `LineTotal` (值对象？)
- `ShippingAddress` (值对象)
- `BillingAddress` (值对象)
要修改某个 `OrderItem` 的数量，必须调用 `Order` 聚合根的 `ChangeItemQuantity(OrderItemId, NewQuantity)` 方法。`Order` 会检查新数量是否有效，重新计算该订单项金额和订单总金额。

### 1.2.5 领域服务
当某个操作或业务逻辑**不适合放在实体或值对象内部**时，将其封装在领域服务中。它代表了一个**无状态**的操作或过程。

场景：
- 操作涉及**多个聚合/实体**的协调。(例如：`TransferService` 处理银行转账，需要操作 `源账户` 和 `目标账户` 两个聚合根)。
- 操作本身是一个**无状态的计算或转换**。(例如：复杂的 `RiskAssessmentService` 计算贷款风险)。
- - 需要调用**外部系统或基础设施**（但核心逻辑仍在领域层）。(例如：`NotificationService` 封装发送通知的规则，实际发送动作可能由基础设施层实现)。

**关键特征：**
- **无状态：** 服务本身不持有业务状态。
-  **领域概念：** 服务执行的操作本身是领域专家关心的核心业务概念（如“转账”、“风险评估”）。
- **接口定义在领域层：** 具体实现在领域层或基础设施层（如果需要访问外部资源）。

**与“应用服务”的区别：**
- **应用服务：** 位于应用层，负责协调领域对象、领域服务、仓储、事务、权限、外部调用等，完成一个用户用例（Use Case）。它更偏重流程协调和技术层面。         
- **领域服务：** 位于领域层，封装了核心的、无法放入实体/值对象的领域逻辑。它只关心业务规则。
### 1.2.6 Repository （仓储或资源库）

**只用于聚合根！** 提供类似集合（Collection）的接口，负责聚合的**持久化（保存）和检索（查询）**。

**关键特征：**
- **聚合根入口：** 只负责聚合根的持久化和加载。加载时，会重建整个聚合（包含内部实体和值对象）。保存时，保存整个聚合的变更。
- **领域层接口：** 仓储的**接口定义在领域层**，因为它表达的是领域模型需要什么样的持久化能力（如 `IOrderRepository` 定义 `Add(Order order)`, `GetById(OrderId id)`, `Save(Order order)` 等方法）。
- **基础设施层实现：** 仓储的**具体实现在基础设施层**（如 `SqlOrderRepository`, `MongoOrderRepository`）。它知道如何操作数据库、缓存、文件系统等。
- **解耦：** 领域层**只依赖于仓储接口**，完全不知道底层存储细节（SQL, NoSQL, File）。这符合**依赖倒置原则（DIP）**。
- **查询分离：** 仓储通常只提供基于聚合根ID的简单查询。复杂的查询（跨聚合、报表）建议使用单独的**查询层**（如CQRS模式中的Query Side），避免污染领域模型和仓储。
### 1.2.7 工厂
负责封装**复杂对象（尤其是聚合）创建逻辑**的对象或方法。

**适用场景：**
 - 对象的创建过程很复杂，涉及多个步骤、规则校验、依赖组合，不适合放在构造函数中（构造函数应尽量简单）。
- 需要解耦创建逻辑和使用逻辑。
- 需要根据条件创建不同的实现（结合抽象工厂模式）。

 **形式：**
- **独立工厂类：** `OrderFactory.createOrder(customer, items, ...)`
- **聚合根上的工厂方法：** `Order.createDraft(customer, ...)` (静态方法)

**作用：** 将复杂的构造逻辑集中管理，保持客户端代码和领域对象（尤其是聚合根）的简洁。

### 1.2.8 领域事件
表示在领域中发生的、**对业务有重要意义**的事件。

**关键特征：**
- **过去时：** 事件名通常是过去时态 (如 `OrderPlaced`, `PaymentReceived`, `InventoryLow`)。
- **包含信息：** 包含事件发生时相关的数据 (如 `OrderPlaced` 事件包含 `OrderId`, `CustomerId`, `OrderItems`, `TotalAmount`, `Timestamp`)。
- **由聚合根发布：** 通常由聚合根在其状态发生重要变更后发布。发布事件是聚合根业务操作的一部分。
-  **轻量级通知：** 事件本身不包含“如何处理”的逻辑，它只是一个通知。

**作用：**
- **解耦限界上下文：** **最重要的作用！** 一个上下文内的聚合根发布事件，其他上下文可以订阅这些事件并触发本地操作，实现松耦合的集成。这是实现最终一致性的基础。(例如：`订单上下文` 发布 `OrderConfirmed` 事件，`库存上下文` 订阅该事件并扣减库存，`物流上下文` 订阅该事件并安排发货)。
- **驱动内部流程：** 同一个聚合或限界上下文内，事件可以触发后续步骤。(例如：`Order` 支付成功后发布 `PaymentReceived` 事件，触发自身状态变更为 `已支付` 并发布 `OrderPaid` 事件)。 
- **审计追踪：** 记录系统中发生的重要业务事实。    
- **CQRS 读模型更新：** 更新用于查询的读模型。

**实现：** 通常需要基础设施支持（事件总线、消息队列）来实现可靠的事件发布和订阅。
# 2 分层架构

> 严格分层架构：某层只能与直接位于的下层发生耦合。
> 松散分层架构：允许上层与任意下层发生耦合。

在领域驱动设计（DDD）中采用的是松散分层架构，层间关系不那么严格。每层都可能使用它下面所有层的服务，而不仅仅是下一层的服务。

![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20250608172035.png)
## 2.1 应用层
负责编排、转发、校验等，该层应该尽可能的做薄，它知道“**做什么**”（流程步骤），但不知道“**怎么做**”（具体业务规则怎么做在领域层）。

职责：
- **协调者：** 编排领域对象（实体、值对象、聚合根）、领域服务、资源库等，完成一个**具体的用户用例 (Use Case)** 或**系统任务**。它代表一个业务场景的完整流程。
- **事务管理：** 通常负责定义和管理事务边界（确保一个用例内的操作要么全成功，要么全失败）。
-  **安全认证：** 执行权限检查（用户是否有权执行此操作？）。
- **基础验证：** 执行简单的、不依赖领域上下文的验证（如 ID 是否存在）。
- **发布领域事件：** 接收领域层产生的事件，并负责将其发布到事件总线/消息队列（通常委托给基础设施层）。
- **返回结果：** 将执行结果（通常是 DTO）返回给用户界面层，或处理异步事件。

包含的内容：
- **应用服务：** 这是这一层的核心组件。每个应用服务方法通常对应一个用户用例或一个原子性系统任务。方法名通常是动词，描述操作（如 `PlaceOrderService.execute(PlaceOrderCommand)`）。
- **命令处理器 / 查询处理器 (CQRS)：** 如果采用 CQRS 模式，应用层会包含处理 `Command` 和 `Query` 的处理器。
- **DTO (输入/输出)：** `Command`, `Query`, `Response` 等对象，用于层间数据传输。

## 2.2 领域层
领域层是绝对的核心，包含了实体、值对象、聚合与聚合根、领域服务、领域事件、仓储接口、工厂接口/实现。

## 2.3 基础设施层

**职责：** 为上层提供**具体的技术实现细节**和**与外部世界的交互能力**。是系统的“工具箱”和“适配器”。

包含的内容：
- **仓储实现：** 提供 `IOrderRepository` 等的具体实现（如 `SqlOrderRepository`, `MongoOrderRepository`），负责与数据库（SQL/NoSQL）、文件系统、缓存等持久化机制交互。
- **外部服务客户端实现：** 调用第三方 API、支付网关、短信服务、邮件服务的具体实现。
- **消息通信：** 实现消息队列（RabbitMQ, Kafka）的生产者/消费者，事件总线 (`IEventBus` 的实现)。
- **文件 I/O：** 读写文件、操作存储（S3, Azure Blob）的具体代码。
- **网络通信：** HTTP 客户端、RPC 客户端的封装。
- **配置管理：** 读取配置文件、环境变量。
- **日志记录实现：** `ILogger` 接口的具体实现（如 Log4Net, Serilog）。
- **身份认证/授权实现：** 与 Auth0、OAuth2 服务器等集成的具体代码。
- **框架集成：** ORM (Entity Framework, Hibernate), Web 框架 (ASP.NET Core, Spring Boot) 的特定配置和粘合代码。
# 3 项目实战
下面是一个订单管理系统的代码示例。

目录结构如下:
```
order-ddd/
├── application/             # 应用层
│   └── OrderApplicationService.java
├── domain/                  # 领域层
│   ├── model/               # 实体、值对象、聚合根
│   │   ├── Order.java
│   │   ├── OrderItem.java
│   │   └── Customer.java
│   ├── repository/          # 仓储接口
│   │   └── OrderRepository.java
│   └── service/             # 领域服务
│       └── OrderDomainService.java
├── infrastructure/          # 基础设施层
│   ├── persistence/         # 数据库持久化实现
│   │   └── OrderRepositoryImpl.java
│   └── event/               # 事件发布等
│       └── OrderEventPublisher.java
└── interfaces/              # 用户接口层（如 REST API）
    └── OrderController.java
```

## 3.1 领域层
### 3.1.1 领域模型层
`Order.java` 聚合根
```java
package com.example.order.domain.model;

import java.util.List;
import java.util.UUID;

public class Order {
    private String orderId;
    private String customerId;
    private List<OrderItem> items;
    private OrderStatus status;

    public Order(String customerId, List<OrderItem> items) {
        this.orderId = UUID.randomUUID().toString();
        this.customerId = customerId;
        this.items = items;
        this.status = OrderStatus.CREATED;
    }

    public void confirm() {
        if (status != OrderStatus.CREATED) {
            throw new IllegalStateException("Only created orders can be confirmed.");
        }
        this.status = OrderStatus.CONFIRMED;
    }

    // Getters
}
```
`OrderItem.java` - 值对象
```java
package com.example.order.domain.model;

public class OrderItem {
    private String productId;
    private int quantity;
    private double price;

    public OrderItem(String productId, int quantity, double price) {
        this.productId = productId;
        this.quantity = quantity;
        this.price = price;
    }

    public double getTotalPrice() {
        return quantity * price;
    }

    // Getters
}
```

`Customer.java` - 实体
```java
package com.example.order.domain.model;

public class Customer {
    private String id;
    private String name;

    public Customer(String id, String name) {
        this.id = id;
        this.name = name;
    }

    // Getters
}
```

`OrderStatus.java` - 枚举
```
package com.example.order.domain.model;

public enum OrderStatus {
    CREATED,
    CONFIRMED,
    CANCELLED
}
```
### 3.1.2 领域仓储层
 `OrderRepository.java` - 领域层仓储接口
```java
package com.example.order.domain.repository;

import com.example.order.domain.model.Order;

public interface OrderRepository {
    void save(Order order);
    Order findById(String orderId);
    Order update(Order order);
}
```
### 3.1.3 领域服务层
OrderDomainService.java
```java
package com.example.order.domain.service;

import com.example.order.domain.model.Order;
import com.example.order.domain.model.OrderItem;
import com.example.order.domain.model.OrderStatus;
import com.example.order.domain.repository.OrderRepository;

import java.util.List;

public class OrderDomainService {

    public boolean validateOrderItems(List<OrderItem> items) {
        for (OrderItem item : items) {
            if (item.getQuantity() <= 0 || item.getPrice() <= 0) {
                return false;
            }
        }
        return true;
    }

    public void validateOrderStatusForConfirm(Order order) {
        if (order.getStatus() != OrderStatus.CREATED) {
            throw new IllegalStateException("Order can only be confirmed if it is in CREATED status.");
        }
    }

    public boolean isOrderValidForCreate(String customerId, List<OrderItem> items) {
        return validateOrderItems(items);
    }

    public void confirmOrder(Order order) {
        validateOrderStatusForConfirm(order);
        order.confirm();
    }
}
```
## 3.2 应用层
`OrderApplicationService.java`
```java
package com.example.order.application;

import com.example.order.domain.model.Customer;
import com.example.order.domain.model.Order;
import com.example.order.domain.repository.OrderRepository;
import com.example.order.infrastructure.event.OrderEventPublisher;

import java.util.List;

public class OrderApplicationService {

    private final OrderRepository orderRepository;
    private final OrderEventPublisher eventPublisher;

    public OrderApplicationService(OrderRepository orderRepository, OrderEventPublisher eventPublisher) {
        this.orderRepository = orderRepository;
        this.eventPublisher = eventPublisher;
    }

    public String createOrder(String customerId, List<OrderItem> items) {
        Order order = new Order(customerId, items);
        orderRepository.save(order);
        eventPublisher.publishOrderCreatedEvent(order.getOrderId());
        return order.getOrderId();
    }

    public void confirmOrder(String orderId) {
        Order order = orderRepository.findById(orderId);
        order.confirm();
        orderRepository.update(order);
        eventPublisher.publishOrderConfirmedEvent(orderId);
    }
}
```

## 3.3 基础设施层（infrastructure）
`OrderRepositoryImpl.java` - 模拟数据库操作
```java
package com.example.order.infrastructure.persistence;

import com.example.order.domain.model.Order;
import com.example.order.domain.repository.OrderRepository;

import java.util.HashMap;
import java.util.Map;

public class OrderRepositoryImpl implements OrderRepository {
    private Map<String, Order> db = new HashMap<>();

    @Override
    public void save(Order order) {
        db.put(order.getOrderId(), order);
    }

    @Override
    public Order update(Order order) {
        return db.put(order.getOrderId(), order);
    }

    @Override
    public Order findById(String orderId) {
        return db.get(orderId);
    }
}
```

 `OrderEventPublisher.java` - 发布事件
 ```java
package com.example.order.infrastructure.event;

public class OrderEventPublisher {
    public void publishOrderCreatedEvent(String orderId) {
        System.out.println("Order Created Event Published: " + orderId);
    }

    public void publishOrderConfirmedEvent(String orderId) {
        System.out.println("Order Confirmed Event Published: " + orderId);
    }
}
```

## 3.4 接口层
`OrderController.java` - 简单的控制层
```java
package com.example.order.interfaces;

import com.example.order.application.OrderApplicationService;
import com.example.order.domain.model.OrderItem;

import java.util.List;

public class OrderController {

    private final OrderApplicationService orderService;

    public OrderController(OrderApplicationService orderService) {
        this.orderService = orderService;
    }

    public String createOrder(String customerId, List<OrderItem> items) {
        return orderService.createOrder(customerId, items);
    }

    public void confirmOrder(String orderId) {
        orderService.confirmOrder(orderId);
    }
}
```

# 4 参考资料
- [领域驱动设计：从理论到实践，一文带你掌握DDD！](https://mp.weixin.qq.com/s/x4HjK8t6mPAg1vQWa3PrSg)