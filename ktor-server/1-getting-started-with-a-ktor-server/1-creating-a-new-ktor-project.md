# Создание нового проекта

---

Окончательный проект: https://github.com/Egor-Liadsky/-ktor-get-started-sample

---

Ktor — это асинхронная платформа для создания микросервисов,
веб-приложений и многого другого. Вы можете создать и настроить
новый проект Ktor, используя специальный плагин для
IntelliJ IDEA Ultimate или веб-генератор проектов.
В этом руководстве мы покажем вам, как создать,
запустить и протестировать простое приложение Ktor.

---
# Предпосылки

Прежде чем начать этот урок:

- [Установите IntelliJ IDEA Ultimate](https://www.jetbrains.com/help/idea/installation-guide.html?_gl=1*u52pxs*_ga*ODA3NjA4NjYuMTY0OTczMzQ4Mg..*_ga_VCMCSM1ZZ7*MTY0OTg3MTQxOC44LjEuMTY0OTg3Nzc4My42MA..&_ga=2.151919610.2132766609.1649733482-80760866.1649733482)
- Убедитесь, что [плагин Ktor](https://www.jetbrains.com/help/idea/ktor.html?_gl=1*u52pxs*_ga*ODA3NjA4NjYuMTY0OTczMzQ4Mg..*_ga_VCMCSM1ZZ7*MTY0OTg3MTQxOC44LjEuMTY0OTg3Nzc4My42MA..&_ga=2.151919610.2132766609.1649733482-80760866.1649733482) установлен и включен.
---
# Начало создание проекта

В этом разделе описывается настройка проекта с
помощью [подключаемого модуля Ktor](https://plugins.jetbrains.com/plugin/16008-ktor) для Intellij
IDEA Ultimate. Если вы используете IntelliJ IDEA 
Community Edition, используйте [генератор проектов Ktor](https://start.ktor.io/?_gl=1*12p6e8b*_ga*ODA3NjA4NjYuMTY0OTczMzQ4Mg..*_ga_VCMCSM1ZZ7*MTY0OTg3MTQxOC44LjEuMTY0OTg3Nzc4My42MA..&_ga=2.173936736.2132766609.1649733482-80760866.1649733482#/settings?name=ktor-sample&website=example.com&artifact=com.example.ktor-sample&kotlinVersion=1.6.20&ktorVersion=2.0.0&buildSystem=GRADLE_KTS&engine=NETTY&configurationIn=CODE&addSampleCode=true&plugins=) .

Чтобы создать новый проект Ktor, [откройте IntelliJ IDEA](https://www.jetbrains.com/help/idea/run-for-the-first-time.html?_ga=2.114654916.2132766609.1649733482-80760866.1649733482&_gl=1*m5ng6p*_ga*ODA3NjA4NjYuMTY0OTczMzQ4Mg..*_ga_VCMCSM1ZZ7*MTY0OTk1MjA5OC45LjAuMTY0OTk1MjEwMC41OA..)
и выполните следующие действия:

1. На экране приветствия щелкните **New Project**.
    В противном случае в главном меню выберите **File | New | Project**.
2. В мастере создания **нового проекта** выберите **Ktor** из списка слева.
3. На правой панели вы можете указать следующие параметры:
<img width="802" alt="ktor_idea_new_project_settings" src="https://user-images.githubusercontent.com/47715067/163443503-8f7650b6-7020-4a6d-b28e-7f6b49a9ce04.png">

   - **Name**: укажите имя проекта.
   - **Location**: укажите каталог для вашего проекта.
   - **Build system**: выберите желаемую систему сборки. Это может быть Gradle с Kotlin или Groovy DSL, или Maven.
   - **Website**: укажите домен, используемый для создания имени пакета.
   - **Artifact**: в этом поле отображается сгенерированное имя артефакта.
   - **Ktor version**: выберите нужную версию Ktor.
   - **Engine**: выберите движок, используемый для запуска сервера.
   - **Configuration in**: Выберите, следует ли указывать параметры сервера [в коде или в файле HOCON](https://ktor.io/docs/create-server.html).
   - **Add sample code**: оставьте этот параметр включенным, чтобы добавить образец кода для
   подключаемых модулей, добавленных на следующей странице.
   
   В этом руководстве мы оставляем значения по умолчанию для этих параметров. Нажмите Далее, чтобы перейти на следующую страницу.

4. На следующей странице вы можете выбрать набор [плагинов](https://ktor.io/docs/plugins.html) —
строительных блоков, обеспечивающих общую функциональность
приложения Ktor, например, аутентификацию, сериализацию и
кодирование контента, сжатие, поддержку файлов cookie и так далее.
![](../../../../../Pictures/post/ktor_idea_new_project_plugins_list.png)
А пока давайте установим только плагин [**Routing**](https://ktor.io/docs/routing-in-ktor.html)
для обработки входящих запросов. Начните 
вводить **Routing** в верхнем левом поле поиска,
найдите **Routing** в списке и нажмите Добавить.
![](../../../../../Pictures/post/ktor_idea_new_project_plugins.animated.gif)
   Нажмите «**Create**» и подождите, пока IntelliJ IDEA создаст проект, и установит зависимости.

Теперь мы готовы запустить созданное приложение.

---
# Запуск приложения Ktor

Чтобы запустить созданное приложение Ktor, выполните следующие действия:
1. Вызовите представление [**Project**](https://www.jetbrains.com/help/idea/project-tool-window.html), и откройте 
файл **Application.kt**, расположенный по следующему пути:
`src /main /kotlin /com /example/ Application.kt`
![](../../../../../Pictures/post/ktor_idea_new_project_view.png)
2. В файл **Application.kt** автоматически добавляется следующий код:
      ```kotlin
      package com.example
      import io.ktor.server.engine.*
      import io.ktor.server.netty.*
      import com.example.plugins.*
   
      fun main() {
          embeddedServer(Netty, port = 8080, host = "0.0.0.0") {
              configureRouting()
          }.start(wait = true)
      }
      ```
      Основные части этого кода:
      - Функция [`embeddedServer`](https://ktor.io/docs/create-server.html#embedded-server) используется для настройки
      параметров сервера в коде и запуске приложения.
      - `configureRouting` является функцией расширения,
      определяющей [маршрутизацию](https://ktor.io/docs/routing-in-ktor.html). Эта функция объявлена
      в отдельном пакете `plugins` (файл **Routing.kt**).
      ```kotlin
      package com.example.plugins
   
      import io.ktor.server.routing.*
      import io.ktor.http.*
      import io.ktor.server.application.*
      import io.ktor.server.response.*
      import io.ktor.server.request.*
   
      fun Application.configureRouting() {
   
          routing {
              get("/") {
                  call.respondText("Hello World!")
              }
          }
      }
      ```
      Функция `get` внутри `routing` блока получает
   запросы GET, сделанные по пути **/**, и отвечает
   простым текстовым ответом.
3. Чтобы запустить приложение, щелкните на значок
рядом с `main` функцией и выберите «**run ApplicationKt**».
![](../../../../../Pictures/post/ktor_idea_new_project_run_gutter.png)
4. Подождите, пока Intellij IDEA запустит приложение. 
В окне инструмента «**Run**» должно появиться следующее сообщение:

   `[main] INFO  ktor.application - Responding at http://0.0.0.0:8080`
![](../../../../../Pictures/post/ktor_idea_new_project_run_tool_window.png)
   Это означает, что сервер готов принимать запросы по
адресу http://0.0.0.0:8080 . Вы можете щелкнуть на эту ссылку, 
чтобы открыть приложение в браузере по умолчанию:
![](../../../../../Pictures/post/ktor_idea_new_project_browser.png)

---

# Протестируйте приложение Ktor

Теперь протестируем созданное приложение:
1. Откройте файл **ApplicationTest.kt**, 
расположенный по следующему пути:
`src /test /kotlin /com /example/ ApplicationTest.kt` . 
В этом файле функция `testApplication` используется для
выполнения запроса GET к **/** и проверки состояния, а так же содержимого ответа.
2. Чтобы запустить тест, щелкните на значок рядом с `testRoot` 
функцией и выберите «**run ApplicationTest.test**» .
![](../../../../../Pictures/post/ktor_idea_new_project_run_test_gutter.png)
3. Подождите, пока IntelliJ IDEA выполнит тест, 
и отобразит результаты в окне инструмента «**Run**».

---

# Следующие шаги

Создавайте реальные приложения Ktor с помощью руководств:
- [Создание HTTP-API](https://github.com/ktor-io-ru/ktor-documentation/blob/main/ktor-server/1-getting-started-with-a-ktor-server/2-creating-HTTP-API.md)
- [Создание интерактивного веб-сайта](https://github.com/ktor-io-ru/ktor-documentation/tree/main/ktor-server/2-creating-a-website)
- [Создание чата WebSocket](https://github.com/ktor-io-ru/ktor-documentation/blob/main/ktor-server/3-creating-a-WebSocket-chat%EF%BB%BF.md)

-> [Далее](https://github.com/ktor-io-ru/ktor-documentation/blob/main/ktor-server/1-getting-started-with-a-ktor-server/2-creating-HTTP-API.md)
