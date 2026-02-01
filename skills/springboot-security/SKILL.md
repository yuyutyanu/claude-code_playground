---
name: springboot-security
description: Spring Securityによる認証・認可、JWT、CSRF、入力バリデーション、セキュリティヘッダー、レート制限のガイド。
model: sonnet-4-5
---

# Spring Boot セキュリティ

## 基本原則

- **デフォルトで拒否**: 明示的に許可したものだけアクセス可能
- **入力をすべて検証**: 外部からのデータは信頼しない
- **最小権限**: 必要最小限のアクセス権を付与
- **設定でセキュア**: コードではなく設定でセキュリティを担保

## Security設定

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable()) // ステートレスAPI（Bearer Token）の場合
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
}
```

## JWT認証フィルタ

```java
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
            HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            if (jwtService.isValid(token)) {
                var auth = jwtService.getAuthentication(token);
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        }
        chain.doFilter(request, response);
    }
}
```

## メソッドレベルセキュリティ

```java
@Service
public class OrderService {

    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public List<Order> getOrders(Long userId) {
        return orderRepository.findByUserId(userId);
    }

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteOrder(Long orderId) {
        orderRepository.deleteById(orderId);
    }
}
```

## 入力バリデーション

```java
public record CreateUserRequest(
    @NotBlank(message = "名前は必須です")
    @Size(max = 100, message = "名前は100文字以内です")
    String name,

    @NotBlank(message = "メールは必須です")
    @Email(message = "有効なメールアドレスを入力してください")
    String email,

    @NotBlank
    @Size(min = 8, max = 100, message = "パスワードは8文字以上100文字以内です")
    String password
) {}
```

## パスワードエンコード

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

// 保存時
String encoded = passwordEncoder.encode(rawPassword);

// 検証時
boolean matches = passwordEncoder.matches(rawPassword, encoded);
```

## セキュリティヘッダー

```java
http.headers(headers -> headers
    .contentSecurityPolicy(csp -> csp.policyDirectives("default-src 'self'"))
    .frameOptions(frame -> frame.deny())
    .xssProtection(xss -> xss.headerValue(XXssProtectionHeaderWriter.HeaderValue.ENABLED_MODE_BLOCK))
    .referrerPolicy(referrer -> referrer.policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.STRICT_ORIGIN_WHEN_CROSS_ORIGIN))
);
```

## レート制限（Bucket4j）

```java
@Component
public class RateLimitFilter extends OncePerRequestFilter {

    private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();

    private Bucket createBucket() {
        return Bucket.builder()
            .addLimit(Bandwidth.classic(100, Refill.intervally(100, Duration.ofMinutes(15))))
            .build();
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
            HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        String ip = request.getRemoteAddr();
        Bucket bucket = buckets.computeIfAbsent(ip, k -> createBucket());
        if (bucket.tryConsume(1)) {
            chain.doFilter(request, response);
        } else {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
        }
    }
}
```

## 秘密情報の管理

```yaml
# application.yml - 環境変数で外部化
spring:
  datasource:
    url: ${DATABASE_URL}
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD}

jwt:
  secret: ${JWT_SECRET}
  expiration: 3600
```

ハードコードしない。Vault等の秘密管理ツールの利用を推奨。

## チェックリスト

- [ ] 認証トークンがhttpOnly Cookieまたは適切なヘッダーで管理されている
- [ ] CSRF対策（ブラウザアプリ）またはCSRF無効化（ステートレスAPI）
- [ ] すべてのエンドポイントに認可ルールがある
- [ ] 入力がBean Validationで検証されている
- [ ] パスワードがBCryptでハッシュ化されている
- [ ] SQLクエリがパラメータ化されている
- [ ] セキュリティヘッダーが設定されている
- [ ] レート制限が重要なエンドポイントに適用されている
- [ ] ログに個人情報・秘密情報が出力されていない
- [ ] `./mvnw dependency:tree` で脆弱な依存関係がない
