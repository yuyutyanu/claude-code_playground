---
name: springboot-patterns
description: Spring Boot開発パターン。RESTコントローラー、サービス層、データアクセス、例外ハンドリング、キャッシュ、非同期処理、ログ。
model: sonnet-4-5
---

# Spring Boot パターン

## RESTコントローラー

```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public Page<UserResponse> list(Pageable pageable) {
        return userService.findAll(pageable);
    }

    @GetMapping("/{id}")
    public UserResponse findById(@PathVariable Long id) {
        return userService.findById(id);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public UserResponse create(@Valid @RequestBody CreateUserRequest request) {
        return userService.create(request);
    }

    @PutMapping("/{id}")
    public UserResponse update(@PathVariable Long id, @Valid @RequestBody UpdateUserRequest request) {
        return userService.update(id, request);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

## DTO（record）

```java
// リクエスト
public record CreateUserRequest(
    @NotBlank @Size(max = 100) String name,
    @NotBlank @Email String email
) {}

// レスポンス
public record UserResponse(Long id, String name, String email, LocalDateTime createdAt) {
    public static UserResponse from(User user) {
        return new UserResponse(user.getId(), user.getName(), user.getEmail(), user.getCreatedAt());
    }
}
```

## サービス層

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    @Transactional(readOnly = true)
    public Page<UserResponse> findAll(Pageable pageable) {
        return userRepository.findAll(pageable).map(UserResponse::from);
    }

    @Transactional(readOnly = true)
    public UserResponse findById(Long id) {
        return userRepository.findById(id)
            .map(UserResponse::from)
            .orElseThrow(() -> new UserNotFoundException(id));
    }

    @Transactional
    public UserResponse create(CreateUserRequest request) {
        if (userRepository.existsByEmail(request.email())) {
            throw new DuplicateEmailException(request.email());
        }
        User user = new User(request.name(), request.email());
        return UserResponse.from(userRepository.save(user));
    }
}
```

## リポジトリ

```java
public interface UserRepository extends JpaRepository<User, Long> {

    boolean existsByEmail(String email);

    @Query("SELECT u FROM User u WHERE u.status = :status")
    Page<User> findByStatus(@Param("status") UserStatus status, Pageable pageable);

    // DTO射影（エンティティ全体を取得しない）
    @Query("SELECT new com.example.dto.UserSummary(u.id, u.name) FROM User u")
    List<UserSummary> findAllSummaries();
}
```

## 集約例外ハンドラ

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ProblemDetail handleNotFound(UserNotFoundException ex) {
        ProblemDetail detail = ProblemDetail.forStatus(HttpStatus.NOT_FOUND);
        detail.setTitle("リソースが見つかりません");
        detail.setDetail(ex.getMessage());
        return detail;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex) {
        ProblemDetail detail = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        detail.setTitle("バリデーションエラー");
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        detail.setProperty("errors", errors);
        return detail;
    }
}
```

## キャッシュ

```java
@Configuration
@EnableCaching
public class CacheConfig {}

@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public ProductResponse findById(Long id) {
        return productRepository.findById(id)
            .map(ProductResponse::from)
            .orElseThrow(() -> new ProductNotFoundException(id));
    }

    @CacheEvict(value = "products", key = "#id")
    public void update(Long id, UpdateProductRequest request) {
        // ...
    }
}
```

## 非同期処理

```java
@Configuration
@EnableAsync
public class AsyncConfig {}

@Service
public class NotificationService {

    @Async
    public CompletableFuture<Void> sendEmail(String to, String subject, String body) {
        // メール送信処理
        return CompletableFuture.completedFuture(null);
    }
}
```

## ログ

```java
@Slf4j // Lombokアノテーション
@Service
public class OrderService {

    public OrderResponse createOrder(CreateOrderRequest request) {
        log.info("注文作成開始: userId={}", request.userId());
        try {
            // ...
            log.info("注文作成完了: orderId={}", order.getId());
            return OrderResponse.from(order);
        } catch (Exception e) {
            log.error("注文作成失敗: userId={}", request.userId(), e);
            throw e;
        }
    }
}
```

## リクエストログフィルタ

```java
@Component
public class RequestLoggingFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
            HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
        long start = System.currentTimeMillis();
        chain.doFilter(request, response);
        long duration = System.currentTimeMillis() - start;
        log.info("{} {} {} {}ms", request.getMethod(), request.getRequestURI(),
            response.getStatus(), duration);
    }
}
```

## 設定のベストプラクティス

```yaml
# application.yml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
  jpa:
    open-in-view: false  # 必ずfalseにする
    properties:
      hibernate:
        default_batch_fetch_size: 100
  problems:
    enabled: true  # RFC 7807 Problem Details
```

## 原則

- コンストラクタインジェクションを使う（`@RequiredArgsConstructor`）
- コントローラーは薄く、サービスに委譲
- 読み取り専用クエリには `@Transactional(readOnly = true)`
- `open-in-view: false` を必ず設定
- recordをDTO/値オブジェクトに活用
