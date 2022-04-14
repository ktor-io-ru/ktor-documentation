# Cоздание HTTP-API

**Пример
кода**: [tutorial-http-api](https://github.com/ktorio/ktor-documentation/tree/main/codeSnippets/snippets/tutorial-http-api)

**Используемые плагины**: [Routing](https://ktor.io/docs/routing-in-ktor.html)
,  [ContentNegotiation](https://ktor.io/docs/serialization.html), kotlinx.serialization

В этом руководстве мы собираемся создать HTTP API, который может служить серверной частью для любого приложения, будь то
мобильное, веб-приложение, настольный компьютер или даже служба B2B. Мы увидим, как определяются и структурируются
маршруты, как плагины сериализации помогают упростить утомительные задачи и как мы можем тестировать части нашего
приложения как вручную, так и автоматически.

На протяжении всего руководства мы создадим простой JSON API, который позволит нам запрашивать информацию о клиентах
нашего фиктивного бизнеса, а также о заказах, которые мы в настоящее время хотим выполнить. Мы создадим удобный способ
перечисления всех клиентов и заказов в нашей системе, получим информацию по отдельным клиентам и заказам, а также
обеспечим функциональность для добавления новых записей и удаления старых записей.

# Предпосылки

- Установите IntelliJ IDEA Ultimate .

  Если вы используете IntelliJ IDEA Community или другую IDE, вы можете создать проект Ktor с помощью веб-генератора
  проектов .

- Убедитесь, что плагин Ktor установлен и включен.

# Создание нового проекта Ktor

Чтобы создать базовый проект для нашего приложения с помощью подключаемого модуля Ktor, откройте IntelliJ IDEA и
выполните следующие действия:

1. На экране приветствия щелкните Новый проект . В противном случае в главном меню выберите Файл | Новый | Проект .

2. В мастере создания нового проекта выберите Ktor из списка слева. На правой панели укажите следующие параметры:
   ![](../../../../../Pictures/post2/tutorial_http_api_new_project.png)

    - **Name** : укажите название проекта.
    - **Location**: укажите каталог для вашего проекта.
    - **Build system**: Убедитесь, что в качестве системы сборки выбран **Gradle Kotlin**.
    - **Website**: оставьте **example.com** значение по умолчанию в качестве домена, используемого для создания названия
      пакета.
    - **Artifact**: в этом поле отображается сгенерированное имя артефакта.
    - **Ktor version**: выберите последнюю версию Ktor.
    - **Engine**: Оставьте **Netty** по умолчанию.
    - **Configuration in**: Выберите файл **HOCON**, чтобы указать параметры сервера в специальном файле конфигурации.
    - **Add sample code**: отключите этот параметр, чтобы пропустить добавление примера кода для плагинов.

   Нажмите «**Next**».
3. На следующей странице добавьте плагины **Routing** , **ContentNegotiation** и **kotlinx.serialization**:
   ![](../../../../../Pictures/post2/tutorial_http_api_new_project_plugins.png)
   Нажмите «**Create**» и подождите, пока IntelliJ IDEA создаст проект и установит зависимости.

# Изучение проекта

Чтобы посмотреть на структуру сгенерированного проекта, давайте вызовем представление **Project** :
![](../../../../../Pictures/post2/tutorial_http_api_project_structure.png)

- Файл `build.gradle.kts` содержит зависимости, необходимые для сервера Ktor и плагинов.
- В `main/resources` папке находятся файлы конфигурации.
- Папка `main/kotlin` содержит сгенерированный исходный код.

### Зависимости

Во-первых, давайте откроем `build.gradle.kts` файл и рассмотрим добавленные зависимости:

```groovy
dependencies {
    implementation("io.ktor:ktor-server-core:$ktor_version")
    implementation("io.ktor:ktor-server-netty:$ktor_version")
    implementation("io.ktor:ktor-server-content-negotiation:$ktor_version")
    implementation("io.ktor:ktor-serialization-kotlinx-json:$ktor_version")
    implementation("ch.qos.logback:logback-classic:$logback_version")
    testImplementation("io.ktor:ktor-server-test-host:$ktor_version")
    testImplementation("org.jetbrains.kotlin:kotlin-test-junit:$kotlin_version")
}
```

Давайте кратко рассмотрим эти зависимости одну за другой:

- `ktor-server-core` добавляет основные компоненты Ktor в наш проект.
- `ktor-server-netty` добавляет в наш проект движок Netty, что позволяет нам использовать функциональные возможности сервера, не полагаясь на внешний контейнер приложений.
- `ktor-server-content-negotiation` и `ktor-serialization-kotlinx-json` предоставит удобный механизм преобразования объектов Kotlin в сериализованную форму, такую как JSON, и наоборот. Мы будем использовать его для форматирования выходных данных нашего API и для использования пользовательского ввода, структурированного в JSON. Чтобы использовать `ktor-serialization-kotlinx-json`, мы также должны применить `plugin.serialization` плагин.
```groovy
plugins {
    kotlin("plugin.serialization").version("1.6.20")
}
```
- `logback-classic` предоставляет реализацию SLF4J, позволяющую нам видеть хорошо отформатированные журналы в консоли.
- `ktor-server-test-host` и `kotlin-test-junit` позволяет нам тестировать части нашего приложения Ktor, не используя в процессе весь стек HTTP. Мы будем использовать это для определения модульных тестов для нашего проекта.

### Конфигурации: application.conf и logback.xml

Сгенерированный проект также включает файлы конфигурации `application.conf` и, расположенные в папке:`logback.xml resources`

- `application.conf` представляет собой файл конфигурации в формате HOCON. Ktor использует этот файл для определения порта, на котором он должен работать, а также определяет точку входа нашего приложения.

   ```
   ktor {
       deployment {
           port = 8080
           port = ${?PORT}
       }
       application {
           modules = [ com.example.ApplicationKt.module ]
       }
   }
   ```

   Если вы хотите узнать больше о настройке сервера Ktor, ознакомьтесь с разделом справки «**Configuration**».
- `logback.xml` устанавливает базовую структуру ведения журнала для нашего сервера. Если вы хотите узнать больше о регистрации в Ktor, ознакомьтесь с разделом «**Logging topic**».

### Исходный код

В `application.conf` точкой входа нашего приложения является `com.example.ApplicationKt.module`. Это соответствует `Application.module()` функции в `Application.kt`, которая является модулем приложения:

```kotlin
fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

fun Application.module() {
    configureRouting()
    configureSerialization()
}
```

Этот модуль, в свою очередь, вызывает следующие функции расширения:
- `configureRouting` это функция, определенная в `plugins/Routing.kt`, которая в настоящее время ничего не делает:
   ```kotlin
    fun Application.configureRouting() {
        routing {
        }
    }
   ```
   Мы определим маршруты для клиентов и заказов в следующих главах.
- `configureSerialization` — это функция, определенная в `plugins/Serialization.kt`, которая устанавливает `ContentNegotiation` и включает `json` сериализатор:
   ```kotlin
   fun Application.configureSerialization() {
       install(ContentNegotiation) {
           json()
       }
   }
   ```