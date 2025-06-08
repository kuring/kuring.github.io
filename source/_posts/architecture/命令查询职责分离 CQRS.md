---
permalink: /architecture/cqrs/
date: 2025-06-08
title: 命令查询职责分离 CQRS
---
CQRS（**Command Query Responsibility Segregation**，命令查询职责分离）是一种在**复杂业务系统中常用的架构模式**，其核心思想是将“读操作”（Query）和“写操作”（Command）进行**职责分离**，从而提升系统的可维护性、扩展性和性能。
- **命令（Command）**：负责处理数据的更新操作（如创建、修改、删除），通常涉及业务逻辑和事务管理。
- **查询（Query）**：仅用于读取数据，专注于高效的数据检索和展示，不包含业务逻辑。

# 1 具体示例
命令模型：
```java
// 1. 定义命令类
public class CreateOrderCommand {
    private String orderId;
    private String productId;
    private int quantity;

    // 构造函数、Getter/Setter
}

// 2. 命令处理服务
@Service
public class OrderCommandService {
    @Autowired
    private OrderRepository orderRepository;
    @Autowired
    private InventoryRepository inventoryRepository;
    @Autowired
    private EventPublisher eventPublisher;

    public void handleCreateOrder(CreateOrderCommand command) {
        // 1. 创建订单
        Order order = new Order(command.getOrderId(), command.getProductId(), command.getQuantity());
        order.setStatus("Pending");
        orderRepository.save(order);

        // 2. 扣减库存
        Inventory inventory = inventoryRepository.findByProductId(command.getProductId());
        inventory.deduct(command.getQuantity());
        inventoryRepository.save(inventory);

        // 3. 发布事件
        eventPublisher.publish(new OrderCreatedEvent(command.getOrderId(), command.getProductId(), command.getQuantity()));
    }
}
```

事件模型（Event Model）
```java
// 领域事件：订单创建事件
public class OrderCreatedEvent {
    private String orderId;
    private String productId;
    private int quantity;

    // 构造函数、Getter/Setter
}
```

查询模型（Query Model）
```java
// 1. 查询服务接口
public interface OrderQueryService {
    OrderView getOrderStatus(String orderId);
}

// 2. 查询视图类（订单状态缓存）
public class OrderView {
    private String orderId;
    private String status;
    private LocalDateTime createdAt;

    // Getter/Setter
}

// 3. 查询服务实现
@Service
public class OrderQueryServiceImpl implements OrderQueryService {
    @Autowired
    private RedisTemplate<String, OrderView> redisTemplate;

    @Override
    public OrderView getOrderStatus(String orderId) {
        return redisTemplate.opsForValue().get("order:" + orderId);
    }

    // 订阅事件并更新缓存
    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        OrderView orderView = new OrderView();
        orderView.setOrderId(event.getOrderId());
        orderView.setStatus("Pending");
        orderView.setCreatedAt(LocalDateTime.now());
        redisTemplate.opsForValue().set("order:" + event.getOrderId(), orderView);
    }
}
```

事件总线（Event Bus）
```java
// 简化的事件发布接口
public interface EventPublisher {
    void publish(Event event);
}

// 简化的事件监听接口
public interface EventListener {
    void onEvent(Event event);
}
```

### 1.1.1 **工作流程**
1. **用户下单**
    - 客户端调用命令服务 `OrderCommandService.handleCreateOrder()`。
    - 命令服务创建订单并扣减库存，发布 `OrderCreatedEvent`。
2. **更新查询视图**
    - 查询服务 `OrderQueryServiceImpl` 监听到 `OrderCreatedEvent`。
    - 将订单状态缓存到 Redis，生成 `OrderView`。
3. **查询订单状态**
    - 客户端调用查询服务 `OrderQueryService.getOrderStatus()`。
    - 查询服务从 Redis 快速返回订单状态。

