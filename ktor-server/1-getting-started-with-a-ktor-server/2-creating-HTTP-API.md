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
- `ktor-server-netty` добавляет в наш проект движок Netty, что позволяет нам использовать функциональные возможности
  сервера, не полагаясь на внешний контейнер приложений.
- `ktor-server-content-negotiation` и `ktor-serialization-kotlinx-json` предоставит удобный механизм преобразования
  объектов Kotlin в сериализованную форму, такую как JSON, и наоборот. Мы будем использовать его для форматирования
  выходных данных нашего API и для использования пользовательского ввода, структурированного в JSON. Чтобы
  использовать `ktor-serialization-kotlinx-json`, мы также должны применить `plugin.serialization` плагин.

```groovy
plugins {
    kotlin("plugin.serialization").version("1.6.20")
}
```

- `logback-classic` предоставляет реализацию SLF4J, позволяющую нам видеть хорошо отформатированные журналы в консоли.
- `ktor-server-test-host` и `kotlin-test-junit` позволяет нам тестировать части нашего приложения Ktor, не используя в
  процессе весь стек HTTP. Мы будем использовать это для определения модульных тестов для нашего проекта.

### Конфигурации: application.conf и logback.xml

Сгенерированный проект также включает файлы конфигурации `application.conf` и, расположенные в
папке:`logback.xml resources`

- `application.conf` представляет собой файл конфигурации в формате HOCON. Ktor использует этот файл для определения
  порта, на котором он должен работать, а также определяет точку входа нашего приложения.

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
- `logback.xml` устанавливает базовую структуру ведения журнала для нашего сервера. Если вы хотите узнать больше о
  регистрации в Ktor, ознакомьтесь с разделом «**Logging topic**».

### Исходный код

В `application.conf` точкой входа нашего приложения является `com.example.ApplicationKt.module`. Это
соответствует `Application.module()` функции в `Application.kt`, которая является модулем приложения:

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
- `configureSerialization` — это функция, определенная в `plugins/Serialization.kt`, которая
  устанавливает `ContentNegotiation` и включает `json` сериализатор:
   ```kotlin
   fun Application.configureSerialization() {
       install(ContentNegotiation) {
           json()
       }
   }
   ```

# Маршруты клиентов

Во-первых, давайте займемся `Customer` стороной нашего приложения. Нам нужно создать модель, которая определяет данные,
связанные с клиентом. Нам также необходимо создать ряд конечных точек, чтобы позволить клиентам добавляться,
перечисляться и удаляться.

### Создайте модель клиента

В нашем случае клиент должен хранить некоторую основную информацию в виде текста: у клиента должен быть, `id` по
которому мы можем его идентифицировать, имя и фамилия, а также адрес электронной почты. Простой способ смоделировать это
в Kotlin — использовать класс данных:

1. Создайте новый пакет с именем `models` внутри `com.example`.
2. Создайте `Customer.kt` файл в `models` пакете и добавьте следующее:
    ```kotlin
    import kotlinx.serialization.Serializable
    
    @Serializable
    data class Customer(val id: String, val firstName: String, val lastName: String, val email: String)
    ```
   Обратите внимание, что мы используем `@Serializable` аннотацию из `kotlinx.serialization`. Вместе с интеграцией с
   Ktor это позволит нам автоматически генерировать JSON-представление, необходимое для наших ответов API, как мы вскоре
   увидим.

### Создайте хранилище клиентов

Чтобы не усложнять код, в этом уроке мы будем использовать хранилище в памяти (то есть изменяемый список `Customers`) —
в реальном приложении мы будем хранить эту информацию в базе данных, чтобы она не потеряны после перезапуска нашего
приложения. Мы можем добавить эту строку сразу после объявления класса данных в файле `Customer.kt`:

```kotlin
val customerStorage = mutableListOf<Customer>()
```

Теперь, когда у нас есть четко определенный `Customer` класс и хранилище для объектов наших клиентов, пришло время
создать конечные точки и предоставить их через наш API!

### Определение маршрутизации для клиентов

Мы хотим отвечать на запросы `GET`, `POST` и `DELETE` на `/customer`. Таким образом, давайте определим наши маршруты с
соответствующими методами HTTP.

1. Создайте новый пакет с именем `routes` внутри `com.example`.
2. Создайте `CustomerRoutes.kt` файл с именем в `routes` пакете и заполните его следующим:
    ```kotlin
    import io.ktor.server.routing.*
    
    fun Route.customerRouting() {
        route("/customer") {
            get {
    
            }
            get("{id?}") {
    
            }
            post {
    
            }
            delete("{id?}") {
    
            }
        }
    }
    ```
   В данном случае мы используем `route` функцию для группировки всего, что попадает под `/customer`. Затем мы создаем
   блок для каждого метода HTTP. Это всего лишь один из подходов к структурированию наших маршрутов — когда мы
   рассмотрим `Order` маршруты в следующей главе, мы увидим другой подход.

   Обратите также внимание на то, что у нас фактически есть две записи для `get`: одна без параметра пути, а другая
   с `{id?}`. Мы будем использовать первую запись для вывода списка всех клиентов, а вторую — для отображения
   конкретного.

##### Список всех клиентов

Чтобы перечислить всех клиентов, мы можем вернуть `customerStorage` список с помощью `call.respond` функции в Ktor,
которая может взять объект Kotlin и вернуть его сериализованным в указанном формате. Для `get` обработчика это выглядит
так:

```kotlin
import com.example.models.*
import io.ktor.http.*
import io.ktor.server.application.*
import io.ktor.server.request.*
import io.ktor.server.response.*
import io.ktor.server.routing.*

fun Route.customerRouting() {
    route("/customer") {
        get {
            if (customerStorage.isNotEmpty()) {
                call.respond(customerStorage)
            } else {
                call.respondText("No customers found", status = HttpStatusCode.OK)
            }
        }
    }
}
```

Для того чтобы это работало, нам нужен `ContentNegotiation` плагин, который уже установлен с `json` сериализатором в
формате `plugins/Serialization.kt`. Что делает согласование контента? Рассмотрим следующий запрос:

```http request
GET http://127.0.0.1:8080/customer
Accept: application/json
```

Когда клиент делает такой запрос, согласование контента позволяет серверу проверить `Accept` заголовок, посмотреть,
может ли он обслуживать этот конкретный тип контента, и если да, вернуть результат.

Поддержка JSON обеспечивается **kotlinx.serialization**. Ранее мы использовали его аннотацию `@Serializable` для
аннотирования нашего `Customer` класса данных, а это означает, что Ktor теперь знает, как сериализовать `Customers` (и
коллекции `Customers`!)

##### Вернуть конкретного клиента

Другой маршрут, который мы хотим поддерживать, — это тот, который возвращает конкретного клиента на основе его
идентификатора (в данном случае `200`):

```http request
GET http://127.0.0.1:8080/customer/200
Accept: application/json
```

В Ktor пути также могут содержать параметры, соответствующие определенным сегментам пути. Мы можем получить доступ к их
значению с помощью оператора индексированного доступа (`call.parameters["myParamName"]`). Добавим в `get("{id?}")`
запись следующий код:

```kotlin
get("{id?}") {
    val id = call.parameters["id"] ?: return@get call.respondText(
        "Missing id",
        status = HttpStatusCode.BadRequest
    )
    val customer =
        customerStorage.find { it.id == id } ?: return@get call.respondText(
            "No customer with id $id",
            status = HttpStatusCode.NotFound
        )
    call.respond(customer)
}
```

Сначала мы проверяем, существует ли параметр `id` в запросе. Если он не существует, мы отвечаем `400 Bad Request` кодом
состояния и сообщением об ошибке, и все готово. Если параметр существует, мы пытаемся найти `find` соответствующую
запись в нашем `customerStorage`. Если мы найдем его, мы ответим объектом. В противном случае мы вернем код
состояния `404` «Not found» с сообщением об ошибке.

##### Создать клиента

Затем мы реализуем опцию для клиента в `POST` JSON-представлении объекта клиента, который затем помещается в наше
хранилище клиента. Его реализация выглядит так:

```kotlin
post {
    val customer = call.receive<Customer>()
    customerStorage.add(customer)
    call.respondText("Customer stored correctly", status = HttpStatusCode.Created)
}
```

`call.receive` интегрируется с настроенным плагином Content Negotiation. Вызов его с общим параметром `Customer`
автоматически десериализует тело запроса JSON в `Customer` объект Kotlin. Затем мы можем добавить клиента в наше
хранилище и ответить кодом состояния `201 Created`.

На этом этапе стоит еще раз подчеркнуть, что в этом руководстве мы также намеренно рассматриваем проблемы, которые могут
возникнуть, например, из-за одновременного доступа к хранилищу нескольких запросов. В производственной среде структуры
данных и код, к которым можно получить одновременный доступ из нескольких запросов/потоков, должны учитывать эти случаи
— то, что выходит за рамки этого руководства.

##### Удалить клиента

Реализация удаления клиента выполняется аналогично процедуре, которую мы использовали для перечисления конкретного
клиента. Сначала мы получаем, `id` а затем модифицируем наш `customerStorage` соответствующим образом:

```kotlin
delete("{id?}") {
    val id = call.parameters["id"] ?: return@delete call.respond(HttpStatusCode.BadRequest)
    if (customerStorage.removeIf { it.id == id }) {
        call.respondText("Customer removed correctly", status = HttpStatusCode.Accepted)
    } else {
        call.respondText("Not Found", status = HttpStatusCode.NotFound)
    }
}
```

Подобно определению нашего `get` запроса, мы удостоверяемся, что `id` не равно нулю. Если `id` отсутствует,
отвечаем `400 Bad Request` ошибкой.

### Зарегистрируйте маршруты

До сих пор мы определяли наши маршруты только внутри функции расширения, `Route` поэтому Ktor еще не знает о наших
маршрутах, и нам нужно их зарегистрировать. Откройте `plugins/Routing.kt` файл и добавьте следующий код:

```kotlin
import com.example.routes.*
import io.ktor.server.application.*
import io.ktor.server.routing.*

fun Application.configureRouting() {
    routing {
        customerRouting()
    }
}
```

Как вы помните, `configureRouting` функция уже вызывается в нашей `Application.module()` функции в` Application.kt`.

Мы завершили реализацию клиентских маршрутов в нашем API. Если вы хотите убедиться, что все работает сразу, вы можете
сразу перейти к главе [Проверка конечных точек HTTP]() вручную. Если вы еще можете терпеть неизвестность, мы можем перейти
к реализации маршрутов, связанных с заказом.