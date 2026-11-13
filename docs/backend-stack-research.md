# Выбор backend-стека для лабораторной №2

Дата проверки официальных источников: **13 ноября 2026 года**. Это исследование вариантов, а не решение за пользователя.
Исходные условия: новичок в микросервисах, неделя по несколько часов в день; Windows, около 31 GiB RAM, Docker Linux с 16 372 322 304 байтами памяти (около 15,25 GiB), Python 3.13 и .NET 8 SDK.
Пользователь выбрал 6 сервисов; язык свободный, по методичке достаточно REST или gRPC. Цель: самостоятельные business services с API, логикой, конфигурацией и запуском; БД, безопасность, ошибки, requestID, документация, health и тесты.
Установка Python/.NET не доказывает знание языка. В исследовании ничего не устанавливалось и сервисы не запускались.

## Как читать сравнение

**Факты** ниже привязаны к документации владельцев технологий. **Выводы** — инженерная оценка применительно к заданию, а не измерения и не обещание сроков.
Сравнивается путь до законченного небольшого API, а не только до первого `Hello World`.
«Оптимизированный» здесь означает разумные затраты на изучение, поддержку и работу прототипа; максимальная скорость без профиля нагрузки не определена.

## Поддержка и выбор версий

| Платформа | Подтверждённый статус на дату проверки | Что это меняет |
|---|---|---|
| Python | 3.13 получает security fixes до октября 2029; 3.14 и 3.15 находятся в bugfix-фазе. [Python versions](https://devguide.python.org/versions/) | Вывод: уже установленный 3.13 — допустимая исходная точка; совместимость выбранных библиотек проверяется отдельно. |
| .NET | .NET 8 и 9 завершили поддержку **10.11.2026**; .NET 10 LTS и .NET 11 STS поддерживаются до **14.11.2028**. [Политика .NET](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core) | Вывод: для нового проекта смотреть на .NET 10 LTS; наличие SDK 8 не является достаточным аргументом выбрать 8. |
| Node.js | Официальная таблица отмечает 26, 24 и 22 как LTS; 25 и 20 — EOL, 27 — Alpha. [Node releases](https://nodejs.org/en/about/previous-releases) | Вывод: проверять Node LTS вместе с требованиями фреймворка; Alpha для этой работы не нужна. |
| Go | На странице выпущены 1.27 и 1.26; релиз поддерживается до выхода двух новых major-версий. [Go release policy](https://go.dev/doc/devel/release) | Вывод: использовать актуальный patch одной из поддерживаемых веток, а не искать «Go LTS». |
| Django | 5.2 LTS имеет extended support до апреля 2028; актуальная страница также содержит 6.1. [Django downloads/support](https://www.djangoproject.com/download/) | Вывод: 5.2 LTS — консервативная основа при подтверждённой совместимости DRF и Python. |
| Spring Boot | Текущая документация 4.1.2 требует Java ≥17 и допускает до Java 27; есть отдельные требования Maven/Gradle. [System requirements](https://docs.spring.io/spring-boot/system-requirements.html) | Вывод: версию JDK, Boot и springdoc выбирать совместно; название LTS у JDK не определяет поддержку Boot. |

Для FastAPI, NestJS, Axum и остальных библиотек здесь не заявляется непроверенный срок LTS.
Поддерживаемая ветка требует свежих исправлений; точные версии зависимостей и образов нужно фиксировать при реализации.
Новый SDK сам по себе не делает архитектуру проще или защиту лабораторной убедительнее.

## Широкое сравнение

Все оценки затрат в последнем столбце — **выводы для новичка без подтверждённого опыта языка**.

| Вариант | Проверяемые возможности | Затраты и подходящая цель |
|---|---|---|
| **Python + FastAPI** | Pydantic-валидация, OpenAPI, интерактивная документация, dependency injection, инструменты OAuth2/security. [Features](https://fastapi.tiangolo.com/features/) | Небольшой объём кода API и ранний видимый результат; самостоятельно добавить соглашения о слоях, БД, логах и health. Хороший кандидат при приоритете понятного прототипа. |
| **Python + Django/DRF** | DRF предоставляет сериализацию, authentication/permissions, browsable API, routers и testing API. [DRF](https://www.django-rest-framework.org/) | Выгоднее при необходимости административной CRUD-системы и желании использовать соглашения Django; для нескольких маленьких сервисов нужно изучить больше фреймворка. |
| **C# + ASP.NET Core** | HTTP API через Minimal APIs или controllers; Microsoft рекомендует Minimal APIs для новых API. [API overview](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/apis?view=aspnetcore-10.0) | Явные типы и цельная платформа помогают объяснять границы; дополнительно изучаются C#, DI и инструменты .NET. Сильный кандидат при интересе к C#. |
| **Go + net/http/chi** | Стандартная HTTP-библиотека; chi совместим с `net/http` и предлагает middleware, включая RequestID и Recoverer. [net/http](https://pkg.go.dev/net/http), [chi](https://github.com/go-chi/chi) | Меньше скрытых соглашений; больше ручной работы над DTO validation, ошибками, OpenAPI и auth. Подходит, если изучение HTTP/SQL важнее минимального объёма кода. |
| **TypeScript + NestJS** | Modules, controllers, DI, guards, exception filters; документированы БД, security и gRPC. [Nest docs](https://docs.nestjs.com/) | Чёткая структура, но одновременно изучаются JS/TS, асинхронность, decorators и Nest. Может стать первым выбором при уже знакомом TypeScript. |
| **TypeScript + Fastify** | JSON Schema validation/serialization и отдельные плагины OpenAPI. [Validation](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/), [fastify-swagger](https://github.com/fastify/fastify-swagger) | Меньше архитектурных правил, чем у Nest; структуру business services выбирает разработчик. Хорош для компактного API при знаниях JS/TS. |
| **Java/Kotlin + Spring Boot** | Spring REST поддерживает Java и Kotlin; экосистема включает SQL, Security, Actuator и testing. [REST guide](https://spring.io/guides/gs/rest-service/), [SQL](https://docs.spring.io/spring-boot/reference/data/sql.html) | Много готовых интеграций; обучение языку, build tool и Spring расширяет объём работы. Становится сильным выбором при опыте Java/Kotlin. |
| **Rust + Axum** | Typed extractors, routing, обработка ошибок, middleware Tower; рассчитан на Tokio/Hyper. [Axum](https://docs.rs/axum/latest/axum/) | Дополнительно ownership/borrowing и async-типы. Для этой лабораторной с недельным бюджетом высокая цена обучения; разумен при уже знакомом Rust. |
| **PHP + Laravel** | Официальная документация содержит routing, validation, auth/authz, ORM/migrations, logging и HTTP tests. [Laravel docs](https://laravel.com/framework/docs/13.x/releases) | Полноценный вариант для знакомого PHP; без опыта добавляет ещё один язык и набор соглашений без очевидной выгоды для этой задачи. |
| **Ruby + Rails API** | Есть специальный API-only режим и выбор middleware/components. [Rails API guide](https://guides.rubyonrails.org/api_app.html) | Быстрые CRUD-сценарии при знакомом Rails; без опыта Ruby/Rails не попадает в короткий список на неделю. |

Laravel/Rails не исключены по технической пригодности: их более низкий приоритет здесь — оценка затрат обучения.
Такой набор охватывает основные разумные направления; экзотические и узкоспециализированные стеки не дают очевидной пользы этому учебному сценарию.

## Что действительно придётся собрать

### FastAPI

- База: FastAPI + Pydantic + Uvicorn; API/DTO и dependency injection уже описаны в [официальных features](https://fastapi.tiangolo.com/features/).
- БД: FastAPI не навязывает ORM; tutorial использует SQLModel поверх SQLAlchemy. [SQL databases](https://fastapi.tiangolo.com/tutorial/sql-databases/)
- Альтернатива для более явного разделения DTO/БД: SQLAlchemy + Alembic; Alembic — отдельный инструмент миграций SQLAlchemy. [SQLAlchemy](https://docs.sqlalchemy.org/en/20/intro.html), [Alembic](https://alembic.sqlalchemy.org/en/latest/)
- Конфигурация: типизированные settings/env через `pydantic-settings`. [Settings](https://fastapi.tiangolo.com/advanced/settings/)
- Тесты API: `TestClient`, HTTPX и pytest описаны в [Testing](https://fastapi.tiangolo.com/tutorial/testing/).
- Auth: tutorial показывает JWT и password hashing с дополнительными библиотеками; готовая аннотация Bearer сама не создаёт систему пользователей. [JWT tutorial](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/)
- Вывод: выбрать один подход к БД; маленькие routers, отдельные функции бизнес-правил, middleware requestID и два health endpoint. Не начинать с одновременного освоения нескольких ORM.

### ASP.NET Core

- DTO validation в Minimal APIs .NET 10 включается через `AddValidation`; без регистрации автоматическая проверка не выполняется. [Validation](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/validation?view=aspnetcore-10.0)
- OpenAPI генерируется через `Microsoft.AspNetCore.OpenApi`; визуальная UI — отдельная часть интеграции. [OpenAPI](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/overview?view=aspnetcore-10.0)
- Для PostgreSQL есть EF Core provider Npgsql; версии EF/provider необходимо согласовать. [Npgsql EF provider](https://www.npgsql.org/efcore/)
- JWT bearer handler проверяет токен, а авторизация использует claims; Microsoft отделяет это от выпуска токенов. [JWT bearer](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication?view=aspnetcore-10.0)
- Есть health middleware и интеграционные тесты через test host/server. [Health checks](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-10.0), [Integration tests](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-10.0)
- Вывод: .NET 10 LTS + Minimal APIs + EF Core/Npgsql — связный кандидат; не добавлять CQRS/MediatR и много проектов лишь ради слова «микросервисы».

### Go, TypeScript, Spring и Rust

- Go: стандартные `database/sql` и `httptest` закрывают SQL API и тестирование HTTP; драйвер БД и прикладные соглашения выбираются отдельно. [SQL](https://go.dev/doc/database/), [httptest](https://pkg.go.dev/net/http/httptest)
- Go/chi: RequestID middleware кладёт идентификатор в context; межсервисная передача этого значения остаётся частью проекта. [chi middleware](https://github.com/go-chi/chi#middlewares)
- Nest: runtime validation выполняет `ValidationPipe` с дополнительными библиотеками; TypeScript-типы не заменяют проверку входящего JSON. [Validation](https://docs.nestjs.com/techniques/validation)
- Nest: OpenAPI использует `@nestjs/swagger`; test module помогает собирать тестируемые зависимости. [OpenAPI](https://docs.nestjs.com/openapi/introduction), [Testing](https://docs.nestjs.com/fundamentals/testing)
- Fastify: есть Pino logging, request ID и `inject()` для HTTP-тестов; логирование надо включить. [Logging](https://fastify.dev/docs/latest/Reference/Logging/), [Testing](https://fastify.dev/docs/latest/Guides/Testing/)
- Spring: JDBC/JPA доступны в экосистеме; JWT resource server — Spring Security; health — Actuator. [SQL](https://docs.spring.io/spring-boot/reference/data/sql.html), [JWT](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html), [Actuator](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html)
- Spring: для OpenAPI есть отдельный проект springdoc; Boot testing предлагает starter с тестовыми библиотеками. [springdoc](https://springdoc.org/), [Boot testing](https://docs.spring.io/spring-boot/reference/testing/index.html)
- Axum: SQLx и utoipa — отдельные инструменты SQL/OpenAPI, а не встроенные функции Axum. [SQLx](https://docs.rs/sqlx/latest/sqlx/), [utoipa](https://docs.rs/utoipa/latest/utoipa/)

## Память, запуск и производительность

Факт: Go build создаёт исполняемый файл; Rust ownership обеспечивает memory safety без сборщика мусора. Это свойства платформ, а не доказательство скорости конкретного API. [Go build](https://go.dev/doc/tutorial/compile-install), [Rust ownership](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html)
Факт: несколько Python worker-процессов обычно имеют раздельную память; число workers влияет на общий расход RAM. [FastAPI deployment concepts](https://fastapi.tiangolo.com/deployment/concepts/)
Факт: тяжёлая работа на Node event loop мешает обработке других запросов; асинхронный синтаксис не устраняет блокировки автоматически. [Node event loop](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop)
Факт: .NET Native AOT и Spring GraalVM могут уменьшать startup/RAM, но добавляют build-time обработку и ограничения. [ASP.NET AOT](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/native-aot?view=aspnetcore-10.0), [Spring native images](https://docs.spring.io/spring-boot/reference/packaging/native-image/introducing-graalvm-native-images.html)
**Вывод:** выделенные Docker около 15,25 GiB не дают причины заранее исключать JVM/.NET или оптимизировать всё на Rust; достаточность для шести сервисов с инфраструктурой проверяется запуском полного стека, который здесь не выполнялся.
**Вывод:** начинать с обычного запуска и одного worker на сервис; усложнять deployment после измерений и появления причины.
**План измерений после реализации:** одинаковый сценарий, release-сборки, одинаковые container limits; отдельно startup-to-ready, idle/loaded memory, latency и ошибки под нагрузкой.
Из этого исследования нельзя честно вывести рейтинг RPS, мегабайты на сервис или гарантировать, что Python станет узким местом: такие тесты не проводились.

## REST, gRPC и общий минимальный стандарт

Факт: gRPC задаёт сервисы/методы в контракте и обычно использует Protocol Buffers с генерацией client/server кода; это отдельные инструменты и workflow. [gRPC introduction](https://grpc.io/docs/what-is-grpc/introduction/)
**Вывод:** поскольку по методичке достаточно REST **или** gRPC, для первого законченного прототипа начать с REST/JSON и OpenAPI; выбор gRPC оправдан отдельным интересом к этому протоколу.
**Вывод:** один язык для первых business services снижает число изучаемых инструментов; полиглотность не демонстрирует самостоятельность сервиса сама по себе.
Следующий список — **предлагаемые критерии готовности**, а не утверждения о возможностях конкретного фреймворка:

1. Каждый сервис имеет собственный API, бизнес-правила, конфигурацию, точку запуска и описание запуска; его можно проверить отдельно.
2. Внешний DTO отделён от хранения; БД воспроизводится миграциями, данные переживают перезапуск, сервисы не читают чужие таблицы напрямую.
3. Проверены аутентификация, роль/право и принадлежность ресурса пользователю; отдельно показаны ответы без токена и при недостаточных правах.
4. requestID возвращается клиенту, присутствует в логах и передаётся при внутреннем вызове; логируются сервис, путь, статус и длительность.
5. Есть согласованные ошибки валидации/бизнес-конфликта/отсутствия ресурса; неожиданные ошибки не раскрывают stack trace клиенту.
6. `/health/live` отвечает о работе процесса, `/health/ready` проверяет нужную для работы БД; старт процесса не объявляется готовностью БД.
7. OpenAPI и пример полного сценария соответствуют реализации; тестируются главное бизнес-правило, БД, запрещённый доступ и один межсервисный сценарий.
8. Есть воспроизводимый локальный запуск и объяснение принятых упрощений; Kubernetes, Kafka, service mesh и Native AOT добавляются только при требовании.

## Условный короткий список

- **Если важнее короткий путь к понятному и документированному API:** Python/FastAPI + PostgreSQL + один ORM/migrations-подход + pytest. Условие: Python не вызывает больше трудностей, чем C#/TS.
- **Если интереснее строгие типы и связная платформа:** C#/ASP.NET Core **10 LTS** + PostgreSQL/EF Core + API integration tests. Условие: готовность изучить C#; установленный SDK 8 не задаёт версию нового проекта.
- **Если уже есть TypeScript:** NestJS для готовой структуры либо Fastify для компактной сборки; сначала выбрать один стиль и один способ описания схем.
- **Если приоритет — увидеть HTTP/SQL без большого framework:** Go/net/http/chi; принять дополнительную работу над validation/OpenAPI/security.
- **Если есть опыт Java/Kotlin, Rust, PHP или Ruby:** соответствующий стек может быть выгоднее всех перечисленных; отсутствие этого опыта меняет оценку.

Без ответа на вопросы ниже не обоснован единственный победитель. Под недельный бюджет разумнее выбрать из 2–3 кандидатов и закончить один сквозной сценарий, чем реализовывать один сервис на каждом языке.

## Неизвестные, меняющие выбор

- Какие языки пользователь уже понимает и хочет развивать; насколько знакомы HTTP, SQL, типы, async и тесты?
- Какая предметная область и какое реальное бизнес-правило отличит сервис от простой CRUD-обёртки?
- Каков желаемый объём UI; какие роли, принадлежность ресурсов и правила доступа нужны выбранному домену?
- Что приоритетнее оптимизировать: время изучения, память, старт, задержку или пропускную способность; какая ожидается нагрузка?
- Доступны ли необходимые образы/пакеты и совместимы ли конкретные версии библиотек? Это требует проверки выбранной сборки, которая здесь не выполнялась.
