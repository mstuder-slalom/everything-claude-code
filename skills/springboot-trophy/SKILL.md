---
name: springboot-trophy
description: Spring Boot trophy testing with real database testing using Testcontainers, MockMvc for controller integration tests, and minimal external service mocking.
---

# Spring Boot Trophy Testing

Trophy testing methodology for Spring Boot applications using JUnit 5, Testcontainers for real databases, and MockMvc for integration testing.

## When to Activate

- Testing Spring Boot REST controllers
- Verifying JPA repository operations
- Testing service layer with real dependencies
- Validating Spring Security configurations

## Test Organization

```
src/
├── main/java/com/example/myapp/
│   ├── controller/
│   ├── service/
│   ├── repository/
│   └── model/
└── test/java/com/example/myapp/
    ├── integration/              # Integration tests (PRIMARY)
    │   ├── UserControllerTest.java
    │   └── OrderServiceTest.java
    ├── unit/                     # Unit tests (selective)
    │   └── PriceCalculatorTest.java
    └── e2e/                      # E2E tests (critical paths)
        └── CheckoutFlowTest.java
```

## Base Test Configuration

```java
// src/test/java/com/example/myapp/BaseIntegrationTest.java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@ActiveProfiles("test")
public abstract class BaseIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    protected TestRestTemplate restTemplate;

    @Autowired
    protected ObjectMapper objectMapper;
}
```

## Controller Integration Tests

```java
@WebMvcTest(UserController.class)
@AutoConfigureMockMvc
class UserControllerTest extends BaseIntegrationTest {
    /**
     * Scenarios from: openspec/changes/auth/specs/registration/spec.md
     */

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    @DisplayName("WHEN valid registration data submitted THEN user is created")
    void testSuccessfulRegistration() throws Exception {
        // Arrange
        var request = new UserRegistrationRequest(
            "newuser@example.com",
            "SecurePass123!",
            "New User"
        );

        // Act & Assert
        mockMvc.perform(post("/api/users/register")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.email").value("newuser@example.com"))
            .andExpect(jsonPath("$.id").exists());

        // Verify in real database
        assertThat(userRepository.findByEmail("newuser@example.com")).isPresent();
    }

    @Test
    @DisplayName("WHEN duplicate email submitted THEN registration rejected")
    void testDuplicateEmailRejected() throws Exception {
        // Arrange - create existing user in real database
        userRepository.save(new User("existing@example.com", "password", "Existing"));

        var request = new UserRegistrationRequest(
            "existing@example.com",
            "SecurePass123!",
            "New User"
        );

        // Act & Assert
        mockMvc.perform(post("/api/users/register")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors.email").exists());
    }
}
```

## Service Integration Tests

```java
@SpringBootTest
@Transactional
class OrderServiceTest extends BaseIntegrationTest {
    /**
     * Scenarios from: openspec/changes/orders/specs/service/spec.md
     */

    @Autowired
    private OrderService orderService;

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private ProductRepository productRepository;

    private User testUser;
    private Product testProduct;

    @BeforeEach
    void setUp() {
        testUser = userRepository.save(new User("test@example.com", "pass", "Test"));
        testProduct = productRepository.save(new Product("Test Product", BigDecimal.valueOf(99.99)));
    }

    @Test
    @DisplayName("WHEN order created with valid items THEN total calculated correctly")
    void testOrderCreation() {
        // Arrange
        var items = List.of(
            new OrderItemRequest(testProduct.getId(), 2),
            new OrderItemRequest(testProduct.getId(), 1)
        );

        // Act
        Order order = orderService.createOrder(testUser.getId(), items);

        // Assert - verify in real database
        Order savedOrder = orderRepository.findById(order.getId()).orElseThrow();
        assertThat(savedOrder.getItems()).hasSize(2);
        assertThat(savedOrder.getTotal()).isEqualByComparingTo(BigDecimal.valueOf(299.97));
    }

    @Test
    @DisplayName("WHEN order cancelled THEN items restored to inventory")
    void testOrderCancellation() {
        // Arrange - create order
        Order order = orderService.createOrder(testUser.getId(),
            List.of(new OrderItemRequest(testProduct.getId(), 5)));

        int originalStock = productRepository.findById(testProduct.getId())
            .orElseThrow().getStockQuantity();

        // Act
        orderService.cancelOrder(order.getId());

        // Assert - inventory restored
        int newStock = productRepository.findById(testProduct.getId())
            .orElseThrow().getStockQuantity();
        assertThat(newStock).isEqualTo(originalStock + 5);
    }
}
```

## Repository Integration Tests

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class UserRepositoryTest {
    /**
     * Scenarios from: openspec/changes/users/specs/repository/spec.md
     */

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    @DisplayName("WHEN searching by email THEN correct user returned")
    void testFindByEmail() {
        // Arrange
        User user = userRepository.save(new User("find@example.com", "pass", "Find Me"));

        // Act
        Optional<User> found = userRepository.findByEmail("find@example.com");

        // Assert
        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("Find Me");
    }

    @Test
    @DisplayName("WHEN user has orders THEN eager fetch works correctly")
    void testUserWithOrders() {
        // Arrange
        User user = userRepository.save(new User("orders@example.com", "pass", "Has Orders"));
        // Orders created via service...

        // Act
        User found = userRepository.findByIdWithOrders(user.getId()).orElseThrow();

        // Assert - no N+1 query
        assertThat(found.getOrders()).isNotEmpty();
    }
}
```

## Mocking External Services Only

```java
@SpringBootTest
@Transactional
class PaymentServiceTest extends BaseIntegrationTest {
    /**
     * Mock Stripe (external), use real database
     */

    @Autowired
    private PaymentService paymentService;

    @Autowired
    private OrderRepository orderRepository;

    @MockBean
    private StripeClient stripeClient;  // External service - OK to mock

    @Test
    @DisplayName("WHEN payment succeeds THEN order marked as paid")
    void testSuccessfulPayment() {
        // Arrange
        Order order = orderRepository.save(createTestOrder());

        when(stripeClient.createPaymentIntent(any()))
            .thenReturn(new PaymentIntent("pi_test", "succeeded"));

        // Act
        PaymentResult result = paymentService.processPayment(order.getId(), "pm_test");

        // Assert
        assertThat(result.isSuccessful()).isTrue();

        Order updated = orderRepository.findById(order.getId()).orElseThrow();
        assertThat(updated.getStatus()).isEqualTo(OrderStatus.PAID);
        assertThat(updated.getPaymentId()).isEqualTo("pi_test");
    }

    @Test
    @DisplayName("WHEN payment fails THEN order remains pending")
    void testFailedPayment() {
        // Arrange
        Order order = orderRepository.save(createTestOrder());

        when(stripeClient.createPaymentIntent(any()))
            .thenThrow(new StripeException("Card declined"));

        // Act
        PaymentResult result = paymentService.processPayment(order.getId(), "pm_test");

        // Assert
        assertThat(result.isSuccessful()).isFalse();

        Order updated = orderRepository.findById(order.getId()).orElseThrow();
        assertThat(updated.getStatus()).isEqualTo(OrderStatus.PENDING);
    }
}
```

## Security Integration Tests

```java
@SpringBootTest
@AutoConfigureMockMvc
class SecurityTest extends BaseIntegrationTest {
    /**
     * Scenarios from: openspec/changes/security/specs/auth/spec.md
     */

    @Autowired
    private MockMvc mockMvc;

    @Test
    @DisplayName("WHEN accessing protected endpoint without auth THEN 401 returned")
    void testUnauthorizedAccess() throws Exception {
        mockMvc.perform(get("/api/admin/users"))
            .andExpect(status().isUnauthorized());
    }

    @Test
    @WithMockUser(roles = "USER")
    @DisplayName("WHEN user accesses admin endpoint THEN 403 returned")
    void testForbiddenAccess() throws Exception {
        mockMvc.perform(get("/api/admin/users"))
            .andExpect(status().isForbidden());
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    @DisplayName("WHEN admin accesses admin endpoint THEN access granted")
    void testAdminAccess() throws Exception {
        mockMvc.perform(get("/api/admin/users"))
            .andExpect(status().isOk());
    }
}
```

## Running Spring Boot Trophy Tests

```bash
# Run all tests
./mvnw test

# Run with coverage
./mvnw test jacoco:report

# Run specific test class
./mvnw test -Dtest=UserControllerTest

# Run integration tests only
./mvnw test -Dgroups=integration

# Skip slow tests
./mvnw test -DexcludedGroups=slow

# Run in parallel
./mvnw test -DforkCount=2C
```

## Trophy Test Report (Spring Boot)

```
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.myapp.integration.UserControllerTest
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] Running com.example.myapp.integration.OrderServiceTest
[INFO] Tests run: 4, Failures: 1, Errors: 0, Skipped: 0

========================= Trophy Test Results ==========================
Spec Coverage:
  auth/spec.md: 5/5 scenarios ✅
  orders/spec.md: 3/4 scenarios (1 failed)
  security/spec.md: 3/3 scenarios ✅

Failed Scenarios:
  ❌ orders/spec.md: "Partial refund" - Not implemented

Total: 11/12 scenarios verified (92%)
========================= 11 passed, 1 failed ==========================
```

## Testcontainers Configuration

```java
// src/test/resources/application-test.yml
spring:
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
  test:
    database:
      replace: none

// For faster tests, use singleton containers
@TestConfiguration
public class TestcontainersConfig {

    private static final PostgreSQLContainer<?> postgres;

    static {
        postgres = new PostgreSQLContainer<>("postgres:15")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test")
            .withReuse(true);
        postgres.start();
    }

    @DynamicPropertySource
    static void registerProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
}
```
