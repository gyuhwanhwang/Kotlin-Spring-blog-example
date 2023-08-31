# 코프링

# Understanding the Gradle Build

## plugins

- `kotlin-spring` 플러그인
    - 자동으로 클래스와 메서드를 `open` 해준다. 스프링 애너테이션으로
    - `@Configuration` 이나 `@Transactional` 을 붙인 빈들을 `open` 키워드 없이 생성할 때 유용
        - CGLIB 프록시 이용하는 것들
- `Kotlin JPA` 플러그인
    - `@Entity`, `@MappedSuperClass` , `@Embeddable` 이 붙은 클래스에 no-arg 생성자를 만들어준다.

## Configure a Gradle project

### [Apply the plugin](https://kotlinlang.org/docs/gradle-configure-project.html#apply-the-plugin)

- Kotlin Gradle 플러그인을 적용하려면 Gradle 플러그인 DSL의 플러그인 블록을 사용하세요.

```kotlin
// replace `<...>` with the plugin name
plugins {
    kotlin("<...>") version "1.9.10"
}
```

- 프로젝트를 구성할 때 사용 가능한 Gradle 버전과 Kotlin Gradle 플러그인(KGP) 호환성을 확인하세요. 다음 표에는 완전히 지원되는 Gradle 및 Android Gradle 플러그인(AGP)의 최소 및 최대 버전이 나와 있습니다.

### Targeting the JVM

- JVM을 대상으로 하려면 Kotlin JVM 플러그인을 적용하세요.
- 버전은 이 블록에서 리터럴이어야 하며 다른 빌드 스크립트에서 적용될 수 없습니다.

> **Check for JVM target compatibility of related compile tasks**
>

### All-open complier plugin

- Kotlin은 기본적으로 클래스와 해당 멤버를 final로 가지고 있기 때문에 클래스를 열어야 하는 Spring AOP와 같은 프레임워크와 라이브러리를 사용하는 것이 불편합니다.
- `all-open` 컴파일러 플러그인은 Kotlin을 해당 프레임워크의 요구 사항에 맞게 조정하고 특정 애너테이션으이 달린 클래스와 해당 멤버를 명시적인 open 키워드 없이 열도록 만듭니다.
- 예를 들어 Spring을 사용할 때 모든 클래스를 열 필요는 없으며 @Configuration 또는 @Service와 같은 특정 애너테이션이 달린 클래스만 열면 됩니다. All-open을 사용하면 이러한 주석을 지정할 수 있습니다.

- Spring을 사용하는 경우 Spring 애너테이션을 수동으로 지정하는 대신 kotlin-spring 컴파일러 플러그인을 활성화할 수 있습니다. kotlin-spring은 all-open 상단의 래퍼이며 정확히 같은 방식으로 동작합니다.
- 지정된 애너테이션 목록
    - `[@Component](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html)`
        - 메타 애너테이션 지원 덕분에 @Configuration, @Controller, @RestController, @Service 또는 @Repository로 주석이 달린 클래스는 자동으로 열립니다. 이러한 애너테이션은 @Component로 메타 애너테이션이 달려 있기 때문입니다.
    - `[@Async](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/annotation/Async.html)`
    - `[@Transactional](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html)`
    - `[@Cacheable](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/Cacheable.html)`
    - `[@SpringBootTest](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/context/SpringBootTest.html)`
- 물론 동일한 프로젝트에서 kotlin-allopen과 kotlin-spring을 모두 사용할 수 있습니다.
- [start.spring.io](http://start.spring.io/) 서비스에서 생성된 프로젝트 템플릿을 사용하는 경우 kotlin-spring 플러그인이 기본적으로 활성화됩니다.

## Compiler options

- Kotlin의 주요 기능 중 하나는 null 안전성입니다. 이는 런타임에 유명한 NullPointerException이 발생하는 대신 컴파일 타임에 null 값을 깔끔하게 처리합니다.
- 이는 Optional과 같은 래퍼 비용을 지불하지 않고도 Null 허용 여부 선언과 "값 또는 값 없음" 의미 체계 표현을 통해 애플리케이션을 더욱 안전하게 만듭니다.
- Java는 유형 시스템에서 널 안전성을 표현하는 것을 허용하지 않지만 Spring Framework는 org.springframework.lang 패키지에 선언된 도구 친화적인 주석을 통해 전체 Spring Framework API의 널 안전성을 제공합니다.
- 기본적으로 Kotlin에서 사용되는 Java API의 유형은 null 검사가 완화되는 플랫폼 유형으로 인식됩니다. JSR 305 주석에 대한 Kotlin 지원 Spring null 허용 여부 애너테이션은 Kotlin 개발자에게 전체 Spring Framework API에 대한 null 안전성을 제공하며 컴파일 타임에 null 관련 문제를 처리할 수 있다는 이점이 있습니다.
- This feature can be enabled by adding the `-Xjsr305` compiler flag with the `strict` options.

```kotlin
tasks.withType<KotlinCompile> {
  kotlinOptions {
    freeCompilerArgs += "-Xjsr305=strict"
  }
}
```

## Dependencies

- 이러한 Spring Boot 웹 애플리케이션에는 2개의 Kotlin 특정 라이브러리가 필요하며(표준 라이브러리는 Gradle을 통해 자동으로 추가됨) 기본적으로 구성됩니다.
    - `kotlin-reflect` is Kotlin reflection library
    - `jackson-module-kotlin` Kotlin 클래스 및 데이터 클래스의 직렬화/역직렬화 지원 추가(단일 생성자 클래스를 자동으로 사용할 수 있으며 보조 생성자 또는 정적 팩토리가 있는 클래스도 지원됨)
- 최신 버전의 H2에서는 `user`와 같은 예약된 키워드를 올바르게 이스케이프하려면 특별한 구성이 필요합니다.
    - src/main/resources/application.properties

    ```kotlin
    spring.jpa.properties.hibernate.globally_quoted_identifiers=true
    spring.jpa.properties.hibernate.globally_quoted_identifiers_skip_column_definitions=true
    ```

- Spring Boot Gradle 플러그인은 Kotlin Gradle 플러그인을 통해 선언된 Kotlin 버전을 자동으로 사용합니다.

# ****Understanding the generated Application****

- Java와 비교하면 세미콜론이 없고, 빈 클래스에 대괄호가 없고(@Bean 주석을 통해 Bean을 선언해야 하는 경우 일부를 추가할 수 있음) runApplication 최상위 함수를 사용한다는 점을 알 수 있습니다.
- `runApplication<BlogApplication>(*args)`는 `SpringApplication.run(BlogApplication::class.java, *args)`에 대한 Kotlin 관용적 대안이며 다음 구문을 사용하여 애플리케이션을 사용자 정의하는 데 사용할 수 있습니다

```kotlin
fun main(args: Array<String>) {
  runApplication<BlogApplication>(*args) {
    setBannerMode(Banner.Mode.OFF)
  }
}
```

# ****Writing your first Kotlin controller****

- 여기서는 기존 Spring 유형에 Kotlin 함수나 연산자를 추가할 수 있는 Kotlin 확장을 사용하고 있습니다.
- 여기서는 `model.addAttribute("title", "Blog")` 대신 `model["title"] = "Blog"`를 작성할 수 있도록 org.springframework.ui.set 확장 함수를 가져옵니다.

```kotlin
import org.springframework.stereotype.Controller
import org.springframework.ui.Model
import org.springframework.ui.set
import org.springframework.web.bind.annotation.GetMapping

@Controller
class HtmlController {

  @GetMapping("/")
  fun blog(model: Model): String {
    model["title"] = "Blog"
    return "blog"
  }

}
```

> Spring Framework KDoc API
https://docs.spring.io/spring-framework/docs/current/kdoc-api/
>

# ****Testing with JUnit 5****

- 이제 Spring Boot에서 기본적으로 사용되는 JUnit 5는 Null을 허용하지 않는 val 속성을 사용할 수 있는 생성자/메서드 매개변수의 자동 연결과 일반 비정적 메서드에서 @BeforeAll/@AfterAll을 사용할 수 있는 가능성을 포함하여 Kotlin에서 매우 편리한 다양한 기능을 제공합니다.
    - [autowiring of constructor/method parameters](https://docs.spring.io/spring/docs/current/spring-framework-reference/testing.html#testcontext-junit-jupiter-di)

## ****Writing JUnit 5 tests in Kotlin****

- 이 예에서는 다양한 기능을 보여주기 위해 통합 테스트를 만들어 보겠습니다.
    - 표현력 있는 테스트 함수 이름을 제공하기 위해 카멜 케이스 대신 백틱 사이에 실제 문장을 사용합니다.
    - JUnit 5에서는 생성자 및 메서드 매개변수를 삽입할 수 있으며 이는 Kotlin 읽기 전용 및 null 허용되지 않는 속성에 적합합니다.
    - 이 코드는 `getForObject` 및 `getForEntity` Kotlin 확장을 활용합니다(가져와야 함).

```kotlin
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class IntegrationTests(@Autowired val restTemplate: TestRestTemplate) {

  @Test
  fun `Assert blog page title, content and status code`() {
    val entity = restTemplate.getForEntity<String>("/")
    assertThat(entity.statusCode).isEqualTo(HttpStatus.OK)
    assertThat(entity.body).contains("<h1>Blog</h1>")
  }

}
```

## ****Test instance lifecycle****

- 때로는 특정 클래스의 모든 테스트 전후에 메서드를 실행해야 하는 경우가 있습니다.
- Junit 4와 마찬가지로 JUnit 5에서는 테스트 클래스가 테스트당 한 번 인스턴스화되기 때문에 기본적으로 이러한 메서드가 정적(Kotlin의 동반 객체로 변환됨, 매우 장황하고 간단하지 않음)이어야 합니다.
    - 개별 테스트 메서드를 독립적으로 실행할 수 있도록 하고 변경 가능한 테스트 인스턴스 상태로 인한 예기치 않은 부작용을 방지하기 위해 JUnit은 각 테스트 메서드를 실행하기 전에 각 테스트 클래스의 새 인스턴스를 만듭니다(정의 참조). 이 "메서드별" 테스트 인스턴스 수명 주기는 JUnit Jupiter의 기본 동작이며 모든 이전 버전의 JUnit과 유사합니다.
- 그러나 Junit 5를 사용하면 이 기본 동작을 변경하고 클래스당 한 번씩 테스트 클래스를 인스턴스화할 수 있습니다. 이는 다양한 방법으로 수행할 수 있습니다. 여기서는 property 파일을 사용하여 전체 프로젝트의 기본 동작을 변경합니다.
- src/test/resources/junit-platform.properties

    ```kotlin
    junit.jupiter.testinstance.lifecycle.default = per_class
    ```

- 이 구성을 사용하면 이제 위의 IntegrationTests 업데이트 버전에 표시된 것처럼 일반 메서드에 @BeforeAll 및 @AfterAll 주석을 사용할 수 있습니다.

# ****Creating your own extensions****

- Java와 같은 추상 메소드와 함께 util 클래스를 사용하는 대신 Kotlin에서는 Kotlin 확장을 통해 이러한 기능을 제공하는 것이 일반적입니다.
- 여기서는 영어 날짜 형식으로 텍스트를 생성하기 위해 기존 LocalDateTime 유형에 format() 함수를 추가하겠습니다.

# ****Persistence with JPA****

- 지연 가져오기가 예상대로 작동하도록 하려면 KT-28525에 설명된 대로 엔터티를 열어야 합니다. 이를 위해 Kotlin allopen 플러그인을 사용할 예정입니다.

```kotlin
plugins {
  ...
  kotlin("plugin.allopen") version "1.8.0"
}

allOpen {
  annotation("jakarta.persistence.Entity")
  annotation("jakarta.persistence.Embeddable")
  annotation("jakarta.persistence.MappedSuperclass")
}
```

- plugin.jpa 는 no-args 를 생성해준다.

- 그런 다음 속성과 생성자 매개변수를 동시에 선언할 수 있는 Kotlin 기본 생성자 간결 구문을 사용하여 모델을 만듭니다.

```kotlin
@Entity
class Article(
    var title: String,
    var headline: String,
    var content: String,
    @ManyToOne var author: User,
    var slug: String = title.toSlug(),
    var addedAt: LocalDateTime = LocalDateTime.now(),
    @Id @GeneratedValue var id: Long? = null)

@Entity
class User(
    var login: String,
    var firstname: String,
    var lastname: String,
    var description: String? = null,
    @Id @GeneratedValue var id: Long? = null)
```

- 여기서는 Article 생성자의 slug 매개변수에 기본 인수를 제공하기 위해 String.toSlug() 확장을 사용하고 있습니다.
- 기본값이 있는 선택적 매개변수는 위치 인수 사용 시 생략이 가능하도록 마지막 위치에 정의됩니다(Kotlin은 명명된 인수도 지원합니다).
- Kotlin에서는 동일한 파일에 간결한 클래스 선언을 그룹화하는 것이 일반적입니다.

<aside>
📌 여기서는 JPA가 불변 클래스 또는 데이터 클래스에 의해 자동으로 생성된 메소드와 작동하도록 설계되지 않았기 때문에 val 속성이 있는 데이터 클래스를 사용하지 않습니다.

다른 Spring Data 플레이버를 사용하는 경우 대부분은 이러한 구성을 지원하도록 설계되었으므로 Spring Data MongoDB, Spring Data JDBC 등을 사용할 때 데이터 클래스 User(val login: String, …)와 같은 클래스를 사용해야 합니다.

</aside>

<aside>
📌 Spring Data JPA를 사용하면 Persistable을 통해 자연 ID(User 클래스의 로그인 속성일 수 있음)를 사용할 수 있지만 KT-6653으로 인해 Kotlin에는 적합하지 않습니다. 따라서 항상 다음과 같은 엔터티를 사용하는 것이 좋습니다. Kotlin에서 생성된 ID.
https://youtrack.jetbrains.com/issue/KT-6653

</aside>

<aside>
📌 여기서는 Optional 기반 CrudRepository.findById의 null 허용 변형인 Spring Data와 함께 기본적으로 제공되는 CrudRepository.findByIdOrNull Kotlin 확장을 사용합니다. 자세한 내용은 실수가 아닌 훌륭한 Null is your Friend 블로그 게시물을 읽어보세요.
[Null is your friend, not a mistake](https://medium.com/@elizarov/null-is-your-friend-not-a-mistake-b63ff1751dd5)

</aside>

# ****Implementing the blog engine****

- 형식이 지정된 날짜로 블로그 및 기사 페이지를 렌더링하기 위해 HtmlController를 업데이트합니다.
- HtmlController에는 단일 생성자(암시적 @Autowired)가 있으므로 ArticleRepository 및 MarkdownConverter 생성자 매개 변수는 자동으로 자동 연결됩니다.
- map { it.render() } 로 데이터 변환
- 테스트 코드도 바꿔주자

<aside>
📌 코드를 더 쉽게 읽을 수 있도록 명명된 매개변수를 사용하는 것에 주목하세요.

</aside>

# ****Exposing HTTP API****

- 테스트의 경우 통합 테스트 대신 Mockito와 유사하지만 Kotlin에 더 적합한 @WebMvcTest 및 Mockk를 활용할 예정입니다.
- @MockBean 및 @SpyBean 주석은 Mockito에만 해당되므로 Mockk에 유사한 @MockkBean 및 @SpykBean 주석을 제공하는 SpringMockK를 활용할 것입니다.

```kotlin
testImplementation("org.springframework.boot:spring-boot-starter-test") {
  exclude(module = "mockito-core")
}
testImplementation("org.junit.jupiter:junit-jupiter-api")
testRuntimeOnly("org.junit.jupiter:junit-jupiter-engine")
testImplementation("com.ninja-squad:springmockk:4.0.0")
```

<aside>
📌 `lateinit`

</aside>

- test

<aside>
📌 $는 문자열 보간에 사용되므로 문자열에서 이스케이프해야 합니다.

</aside>

# ****Configuration properties****

- Kotlin에서 애플리케이션 속성을 관리하는 데 권장되는 방법은 읽기 전용 속성을 사용하는 것입니다.

```kotlin
@ConfigurationProperties("blog")
data class BlogProperties(var title: String, val banner: Banner) {
  data class Banner(val title: String? = null, val content: String)
}
```

- 그런 다음 BlogApplication 수준에서 활성화합니다.

```kotlin
@SpringBootApplication
@EnableConfigurationProperties(BlogProperties::class)
class BlogApplication {
  // ...
}
```

- IDE에서 이러한 사용자 정의 속성을 인식하기 위해 고유한 메타데이터를 생성하려면 다음과 같이 spring-boot-configuration-processor 종속성을 사용하여 kapt를 구성해야 합니다.

```kotlin
plugins {
  ...
  kotlin("kapt") version "1.8.0"
}

dependencies {
  ...
  kapt("org.springframework.boot:spring-boot-configuration-processor")
}
```

<aside>
📌 kapt가 제공하는 모델의 제한으로 인해 일부 기능(예: 기본값 감지 또는 더 이상 사용되지 않는 항목)이 작동하지 않습니다. 또한 KT-18022로 인해 Maven에서는 주석 처리가 아직 지원되지 않습니다. 자세한 내용은initializr#438을 참조하세요.

</aside>

- In IntelliJ IDEA:
    - Make sure Spring Boot plugin in enabled in menu File | Settings | Plugins | Spring Boot
    - Enable annotation processing via menu File | Settings | Build, Execution, Deployment | Compiler | Annotation Processors | Enable annotation processing
    - Since [Kapt is not yet integrated in IDEA](https://youtrack.jetbrains.com/issue/KT-15040), you need to run manually the command `./gradlew kaptKotlin` to generate the metadata