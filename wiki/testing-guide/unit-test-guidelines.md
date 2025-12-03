# 单元测试指南

## 1. 测试框架

推荐使用：
- **JUnit 5** - Java 单元测试框架
- **Mockito** - Mock 框架
- **AssertJ** - 断言库

---

## 2. 测试原则

### 2.1 FIRST 原则

- **F**ast - 测试应该快速运行
- **I**ndependent - 测试之间相互独立
- **R**epeatable - 测试结果可重复
- **S**elf-Validating - 测试自动验证结果
- **T**imely - 测试应及时编写

### 2.2 AAA 模式

```java
@Test
void testCalculateTotalAmount() {
    // Arrange（准备）
    Order order = new Order();
    order.addItem(new OrderItem("Product A", 2, 100.0));

    // Act（执行）
    double total = orderService.calculateTotal(order);

    // Assert（断言）
    assertEquals(200.0, total, 0.01);
}
```

---

## 3. 测试场景覆盖

### 3.1 正常场景

测试基本功能在正常输入下的行为

```java
@Test
@DisplayName("正常情况：计算单个商品订单金额")
void testSingleItemOrder() {
    // 测试代码
}
```

### 3.2 边界场景

测试边界值和特殊情况

```java
@Test
@DisplayName("边界情况：数量为 0")
void testZeroQuantity() {
    // 测试代码
}

@Test
@DisplayName("边界情况：最大金额")
void testMaxAmount() {
    // 测试代码
}
```

### 3.3 异常场景

测试错误输入和异常处理

```java
@Test
@DisplayName("异常情况：负数数量应抛出异常")
void testNegativeQuantity() {
    assertThrows(IllegalArgumentException.class, () -> {
        new Order(-1, 100.0);
    });
}
```

---

## 4. Mock 使用

### 4.1 什么时候使用 Mock

- 依赖外部系统（数据库、API）
- 测试错误处理逻辑
- 隔离单元测试

### 4.2 Mock 示例

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @InjectMocks
    private OrderService orderService;

    @Test
    void testGetOrderById() {
        // Given
        Order mockOrder = new Order(1, "Product A", 100.0);
        when(orderRepository.findById(1)).thenReturn(Optional.of(mockOrder));

        // When
        Order result = orderService.getOrderById(1);

        // Then
        assertNotNull(result);
        assertEquals(1, result.getId());
        verify(orderRepository, times(1)).findById(1);
    }
}
```

---

## 5. 数据库测试

### 5.1 测试数据库选择

**推荐：** 使用内存数据库（H2）进行单元测试

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

### 5.2 测试数据准备

```java
@BeforeEach
void setUp() {
    // 准备测试数据
    jdbcTemplate.execute("DELETE FROM employees");
    jdbcTemplate.execute("INSERT INTO employees VALUES (1, 'John', 50000)");
    jdbcTemplate.execute("INSERT INTO employees VALUES (2, 'Jane', 60000)");
}

@AfterEach
void tearDown() {
    // 清理测试数据
    jdbcTemplate.execute("DELETE FROM employees");
}
```

---

## 6. 覆盖率要求

- **行覆盖率：** ≥ 80%
- **分支覆盖率：** ≥ 75%
- **关键业务逻辑：** 100%

**使用 JaCoCo 生成覆盖率报告：**

```xml
<!-- pom.xml -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

---

## 7. 最佳实践

### 7.1 测试命名

```java
// ❌ 不推荐
@Test
void test1() { }

// ✅ 推荐
@Test
@DisplayName("TC-001: 单个商品订单金额计算")
void testSingleItemOrderCalculation() { }
```

### 7.2 断言清晰

```java
// ❌ 不推荐
assertTrue(result == expected);

// ✅ 推荐
assertEquals(expected, result, "订单金额计算错误");
```

### 7.3 一个测试一个断言（尽量）

```java
// ❌ 不推荐（多个断言）
@Test
void testOrder() {
    assertEquals(100, order.getAmount());
    assertEquals("Pending", order.getStatus());
    assertEquals(1, order.getItemCount());
}

// ✅ 推荐（拆分为多个测试）
@Test
void testOrderAmount() {
    assertEquals(100, order.getAmount());
}

@Test
void testOrderStatus() {
    assertEquals("Pending", order.getStatus());
}
```

---

**文档版本:** v1.0
**最后更新:** 2025-01-25
