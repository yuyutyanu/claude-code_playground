---
name: jpa-patterns
description: JPA/Hibernateの設計パターン。エンティティ設計、リレーション、クエリ最適化、N+1問題対策、トランザクション、Testcontainers。
model: sonnet-4-5
---

# JPA/Hibernate パターン

## エンティティ設計

```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_users_email", columnList = "email", unique = true),
    @Index(name = "idx_users_status_created", columnList = "status, created_at")
})
@EntityListeners(AuditingEntityListener.class)
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserStatus status = UserStatus.ACTIVE;

    @CreatedDate
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    // JPA用のデフォルトコンストラクタ（protected）
    protected User() {}

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

## リレーション

### 遅延読み込みをデフォルトに

```java
@Entity
public class Order {

    @ManyToOne(fetch = FetchType.LAZY)  // デフォルトはEAGER → 必ずLAZYに
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();  // デフォルトでLAZY

    // 便利メソッド（双方向の整合性を保つ）
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
    }

    public void removeItem(OrderItem item) {
        items.remove(item);
        item.setOrder(null);
    }
}
```

## N+1問題の対策

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // JOIN FETCHで関連エンティティを一括取得
    @Query("SELECT o FROM Order o JOIN FETCH o.user JOIN FETCH o.items WHERE o.id = :id")
    Optional<Order> findByIdWithDetails(@Param("id") Long id);

    // EntityGraphでも可
    @EntityGraph(attributePaths = {"user", "items"})
    Optional<Order> findWithDetailsById(Long id);
}
```

### バッチフェッチ

```yaml
# application.yml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 100
```

コレクションのLAZYロード時に、N回ではなくバッチでSQLを発行。

## DTO射影（読み取り専用）

```java
// インターフェース射影
public interface UserSummary {
    Long getId();
    String getName();
}

// リポジトリ
List<UserSummary> findByStatus(UserStatus status);

// コンストラクタ射影（record）
@Query("SELECT new com.example.dto.UserDto(u.id, u.name, u.email) FROM User u WHERE u.status = :status")
List<UserDto> findDtosByStatus(@Param("status") UserStatus status);
```

エンティティ全体を取得する必要がない読み取り操作にはDTO射影を使う。

## トランザクション

```java
@Service
@RequiredArgsConstructor
public class TransferService {

    private final AccountRepository accountRepository;

    @Transactional
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        Account from = accountRepository.findById(fromId)
            .orElseThrow(() -> new AccountNotFoundException(fromId));
        Account to = accountRepository.findById(toId)
            .orElseThrow(() -> new AccountNotFoundException(toId));

        from.withdraw(amount);  // 残高不足なら例外 → ロールバック
        to.deposit(amount);
        // saveは不要（@Transactional内のmanaged entityは自動flush）
    }

    @Transactional(readOnly = true)  // 読み取り専用（DB最適化が有効になる）
    public AccountResponse getBalance(Long id) {
        return accountRepository.findById(id)
            .map(AccountResponse::from)
            .orElseThrow(() -> new AccountNotFoundException(id));
    }
}
```

## バルク操作

```java
// saveAll + バッチ設定
@Transactional
public void importUsers(List<CreateUserRequest> requests) {
    List<User> users = requests.stream()
        .map(r -> new User(r.name(), r.email()))
        .toList();
    userRepository.saveAll(users);  // batch_sizeに従いバッチINSERT
}
```

```yaml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 50
        order_inserts: true
        order_updates: true
```

## ページネーション

```java
// Spring Data Pageableを活用
Page<User> findByStatus(UserStatus status, Pageable pageable);

// 使用
Pageable pageable = PageRequest.of(0, 20, Sort.by("createdAt").descending());
Page<User> page = userRepository.findByStatus(UserStatus.ACTIVE, pageable);
```

## テスト

```java
@DataJpaTest
@Testcontainers
@AutoConfigureTestDatabase(replace = Replace.NONE)
class UserRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    void ステータスでユーザーを検索する() {
        userRepository.save(new User("アクティブ", "active@example.com"));
        var inactive = new User("非アクティブ", "inactive@example.com");
        inactive.setStatus(UserStatus.INACTIVE);
        userRepository.save(inactive);

        var result = userRepository.findByStatus(UserStatus.ACTIVE, Pageable.unpaged());
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getContent().get(0).getName()).isEqualTo("アクティブ");
    }
}
```

## SQLログの有効化（開発時）

```yaml
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

## アンチパターン

- `FetchType.EAGER` をリレーションに使用（N+1の原因）
- `open-in-view: true`（デフォルト）のまま運用（コントローラーでLazy Load発生）
- エンティティをそのままAPIレスポンスに返す（DTOを使う）
- 双方向リレーションの整合性を保たない
- 大量データをfindAll()で全件取得
