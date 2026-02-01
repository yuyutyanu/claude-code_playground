---
name: java-coding-standards
description: Java 17+のコーディング規約。命名規則、イミュータビリティ、Optional、Stream、例外処理、型安全性、プロジェクト構成。
model: sonnet-4-5
---

# Java コーディング規約

## 基本原則

1. **明確さ > 賢さ**: トリッキーなコードより読みやすいコード
2. **イミュータブルをデフォルトに**: 共有可変状態を最小化
3. **防御的プログラミング**: 意味のある例外で早期に失敗

## 命名規則

| 対象 | 規則 | 例 |
|------|------|-----|
| クラス | PascalCase | `UserService`, `OrderRepository` |
| メソッド・フィールド | camelCase | `findByEmail`, `userName` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| パッケージ | すべて小文字 | `com.example.user` |
| Enum値 | UPPER_SNAKE_CASE | `ACTIVE`, `PENDING` |

## プロジェクト構成

```
src/main/java/com/example/app/
├── config/         # 設定クラス（@Configuration）
├── controller/     # RESTコントローラー
├── service/        # ビジネスロジック
├── repository/     # データアクセス
├── domain/         # エンティティ、値オブジェクト
├── dto/            # データ転送オブジェクト
├── exception/      # カスタム例外
└── util/           # ユーティリティ
```

## イミュータビリティ

```java
// record（Java 16+）を優先
public record UserResponse(String id, String name, String email) {}

// 従来のクラスではfinalフィールド + getter のみ
public class User {
    private final String id;
    private final String name;

    public User(String id, String name) {
        this.id = id;
        this.name = name;
    }

    public String getId() { return id; }
    public String getName() { return name; }
}
```

## Optional

```java
// OK: map/flatMapで変換
Optional<String> name = userRepository.findById(id)
    .map(User::getName);

// OK: 存在しない場合に例外
User user = userRepository.findById(id)
    .orElseThrow(() -> new UserNotFoundException(id));

// NG: get()を直接呼ぶ
User user = userRepository.findById(id).get(); // NoSuchElementExceptionの危険

// NG: Optionalをフィールドやパラメータに使う
private Optional<String> name; // やめる
```

## Stream

```java
// 変換・フィルタリングに活用
List<String> activeNames = users.stream()
    .filter(User::isActive)
    .map(User::getName)
    .sorted()
    .toList(); // Java 16+

// 複雑なネストはループにする
// NG: 読みにくい
users.stream()
    .flatMap(u -> u.getOrders().stream()
        .filter(o -> o.getItems().stream()
            .anyMatch(i -> i.getPrice() > 1000)))
    .collect(Collectors.toList());

// OK: ループで明確に
List<Order> result = new ArrayList<>();
for (User user : users) {
    for (Order order : user.getOrders()) {
        if (order.getItems().stream().anyMatch(i -> i.getPrice() > 1000)) {
            result.add(order);
        }
    }
}
```

## 例外処理

```java
// ドメイン固有の非チェック例外
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String id) {
        super("ユーザーが見つかりません: " + id);
    }
}

// 技術的エラーはコンテキストを付けてラップ
try {
    return objectMapper.readValue(json, User.class);
} catch (JsonProcessingException e) {
    throw new DataParseException("ユーザーJSONのパースに失敗", e);
}

// NG: 広範なcatch
catch (Exception e) { ... } // 避ける
```

## 型安全性

```java
// NG: raw型
List list = new ArrayList();

// OK: ジェネリクス
List<User> users = new ArrayList<>();

// 制約付きジェネリクス
public <T extends Comparable<T>> T findMax(List<T> items) {
    return items.stream().max(Comparator.naturalOrder()).orElseThrow();
}
```

## テスト

```java
// JUnit 5 + AssertJ
import static org.assertj.core.api.Assertions.*;

@Test
void ユーザー名を正しくフォーマットする() {
    // Arrange
    var user = new User("1", "田中太郎");

    // Act
    var formatted = userService.formatName(user);

    // Assert
    assertThat(formatted).isEqualTo("田中 太郎");
}
```

## アンチパターン

- Setter の多用（イミュータブルにする）
- `Optional.get()` の直接呼び出し
- `catch (Exception e)` の広範なキャッチ
- raw型の使用
- staticメソッドの多用（テストしにくい）
- Godクラス（責務を分割する）
