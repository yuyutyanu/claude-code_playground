---
name: springboot-testing
description: Spring BootのTDD・テストガイド。JUnit 5、Mockito、MockMvc、@SpringBootTest、@DataJpaTest、Testcontainersを使ったテスト戦略。
model: sonnet-4-5
---

# Spring Boot テスト

## テスト層と使い分け

| 層 | アノテーション | 対象 | 速度 |
|----|---------------|------|------|
| ユニット | なし（JUnit + Mockito） | サービス、ユーティリティ | 最速 |
| Web層 | `@WebMvcTest` | コントローラー | 速い |
| データ層 | `@DataJpaTest` | リポジトリ | 中程度 |
| 統合 | `@SpringBootTest` | アプリ全体 | 遅い |

## ユニットテスト（サービス層）

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void IDでユーザーを取得できる() {
        // Arrange
        var user = new User(1L, "田中太郎", "tanaka@example.com");
        when(userRepository.findById(1L)).thenReturn(Optional.of(user));

        // Act
        var result = userService.findById(1L);

        // Assert
        assertThat(result.name()).isEqualTo("田中太郎");
        verify(userRepository).findById(1L);
    }

    @Test
    void 存在しないIDで例外をスローする() {
        when(userRepository.findById(999L)).thenReturn(Optional.empty());

        assertThatThrownBy(() -> userService.findById(999L))
            .isInstanceOf(UserNotFoundException.class)
            .hasMessageContaining("999");
    }
}
```

## Web層テスト（MockMvc）

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void ユーザー一覧を取得する() throws Exception {
        var users = List.of(new UserResponse(1L, "田中", "tanaka@example.com", LocalDateTime.now()));
        when(userService.findAll(any())).thenReturn(new PageImpl<>(users));

        mockMvc.perform(get("/api/users"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.content[0].name").value("田中"));
    }

    @Test
    void バリデーションエラーで400を返す() throws Exception {
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name": "", "email": "invalid"}
                    """))
            .andExpect(status().isBadRequest());
    }
}
```

## データ層テスト（JPA + Testcontainers）

```java
@DataJpaTest
@Testcontainers
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    void メールでユーザーの存在を確認する() {
        userRepository.save(new User("田中太郎", "tanaka@example.com"));

        assertThat(userRepository.existsByEmail("tanaka@example.com")).isTrue();
        assertThat(userRepository.existsByEmail("unknown@example.com")).isFalse();
    }
}
```

## 統合テスト

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class UserIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void ユーザーを作成して取得する() {
        var request = new CreateUserRequest("田中太郎", "tanaka@example.com");

        var createResponse = restTemplate.postForEntity("/api/users", request, UserResponse.class);
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);

        Long userId = createResponse.getBody().id();
        var getResponse = restTemplate.getForEntity("/api/users/" + userId, UserResponse.class);
        assertThat(getResponse.getBody().name()).isEqualTo("田中太郎");
    }
}
```

## テストビルダー

```java
// テストデータの生成を簡潔にする
public class UserTestBuilder {
    private Long id = 1L;
    private String name = "テストユーザー";
    private String email = "test@example.com";

    public static UserTestBuilder aUser() { return new UserTestBuilder(); }
    public UserTestBuilder withName(String name) { this.name = name; return this; }
    public UserTestBuilder withEmail(String email) { this.email = email; return this; }
    public User build() { return new User(id, name, email); }
}

// 使用例
var admin = UserTestBuilder.aUser().withName("管理者").withEmail("admin@example.com").build();
```

## パラメータ化テスト

```java
@ParameterizedTest
@CsvSource({
    "tanaka@example.com, true",
    "invalid-email, false",
    "'', false"
})
void メールアドレスのバリデーション(String email, boolean expected) {
    assertThat(validator.isValidEmail(email)).isEqualTo(expected);
}
```

## カバレッジ設定（JaCoCo）

```xml
<!-- pom.xml -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals><goal>prepare-agent</goal></goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals><goal>report</goal></goals>
        </execution>
    </executions>
    <configuration>
        <rules>
            <rule>
                <limits>
                    <limit>
                        <counter>LINE</counter>
                        <minimum>0.80</minimum>
                    </limit>
                </limits>
            </rule>
        </rules>
    </configuration>
</plugin>
```

## テストコマンド

```bash
# 全テスト実行
./mvnw test

# 特定クラス
./mvnw test -Dtest=UserServiceTest

# カバレッジレポート生成
./mvnw test jacoco:report
# target/site/jacoco/index.html で確認
```

## 原則

- 振る舞いをテストする（実装の詳細ではない）
- テストは高速・独立・決定的に
- ユニットテストを最も多く、統合テストは重要なフローのみ
- Testcontainersで本番に近いDB環境をテスト
- AssertJのフルエントAPIを活用
