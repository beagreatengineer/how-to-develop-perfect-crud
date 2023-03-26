# How to develop perfect CRUD
[<img src="https://img.shields.io/badge/channel-telegram-blue">](https://t.me/careerunderhood) [<img src="https://img.shields.io/badge/message-telegram-blue">](https://t.me/ea_kozlov)

# 🤔 What is this?
This article is an attempt to combine best practices that are helpful to keep in mind when developing backend applications.

It can be used as a checklist if:
- You are starting a greenfield project and want to ensure that best practices are used right from the start.
- You received a test assignment and have decided to move forward with it.
 
## 🤔 Does this article only cover backend apps? 
Because the author's expertise lies mostly on the backend side, most suggestions will be useful for developing backend apps. However, a lot of the points covered in this article can also be handy for other types of development.

## 🤔 There is a lot in here, do I really need all of this?

The more features and practices you adopt in your project, the better. Applying all of them at once would be hard so consider your time 🕒 and effort 💪 to prioritize specific practices.
## 🤔 "This article is missing information about technology X"

If you have a proposal about covering a specific technology here, don't be shy: 
- Open an issue ✅
- Open a PR ✅

# 📖 Table of Contents
<!-- TOC start -->
- [Repository](#repository)
- [Code Style](#code-style)
- [✔️Tests](#%EF%B8%8Ftests)
- [⚙️Configuration & Infrastructure around Code](#%EF%B8%8Fconfiguration--infrastructure-around-code)
- [API Design](#api-design)
- [Authorization & Authentification](#authorization--authentification)
- [MVC Explanation](#mvc-explanation)
  - [Controller](#controller)
  - [Model](#model)
  - [Service](#service)
  - [View](#view)
- [📐✏️👷‍♀️Architecture, Design Patterns, Refactoring, etc](#%EF%B8%8F%EF%B8%8Farchitecture-design-patterns-refactoring-etc)
- [🔒CRUD: Validations](#crud-validations)
- [CRUD: Database](#crud-database)
- [CRUD: Operations](#crud-operations)
  - [LIST (HTTP GET)](#list-http-get)
  - [READ (HTTP GET)](#read-http-get)
  - [CREATE (HTTP POST)](#create-http-post)
  - [UPDATE (HTTP PUT/PATCH)](#update-http-putpatch)
  - [DESTROY (HTTP DELETE)](#destroy-http-delete)
- [External API Calls, Long-running tasks (And why we need message queue)](#external-api-calls-long-running-tasks-and-why-we-need-message-queue)
- [📈Logs and Metrics](#logs-and-metrics)
- [🛡️Security](#%EF%B8%8Fsecurity)
- [CORS Headers (Cross-Origin Resource Sharing)](#cors-headers-cross-origin-resource-sharing)
- [WIP: Transactions, Locks, Isolation Levels, ACID](#wip-transactions-locks-isolation-levels-acid)
- [WIP: Cache](#wip-cache)
- [WIP: Full Text Search](#wip-full-text-search)
<!-- TOC end -->
# Repository
* Codebase has to live on a public or private Git repository (Github / Gitlab / Bitbucket)
* Repository settings must disable force pushing into (`push --force`) into main branches (`master`, `main`, release branches)
* 'README' should contain:
  - info about the project
  - short summary of tooling and the tech stack
  - instructions for setting up and launching the application
* Use feature branches, pull requests. Refer to this great [article](https://www.flagship.io/git-branching-strategies/) with comparison of different Git Branching Strategies. 
* Use readable commit history. Here is a good set of rules [Conventional commits](https://www.conventionalcommits.org/en/v1.0.0/)
* Set up Continuous Integration (Gitlab CI / Github Actions)
* `feature/` and master branches should be set up with:
  * test runners and [coverage](https://www.guru99.com/test-coverage-in-software-testing.html) stats
  * linting  
* Setting up Continuous Delivery - app delpoyment into differnet environmetns (e.g. test/staging/prod) would be a huge plus!
* Optional: Set up [dependabot](https://docs.github.com/ru/code-security/dependabot/working-with-dependabot)

# Code Style
Before development:
* Have a working editor or an IDE:
  * VS Code
  * Visual Studio
  * PyCharm
  * IDEA
  * Vim, emacs
* [EditorConfig](https://editorconfig.org/) plugin / add-on is up and running for your editor
* Set up Code formatters: 
  * Rubocop for Ruby
  * Pylint/Black/PEP8 for Python
# ✔️Tests
* Install libraries for writing various test types (unit, integration). For instance: 
   * Pytest for Python
   * RSpec for Ruby
   * Testify, testcontainers for Golang
* Test coverage is calculated automatically after each test run
* Apply [AAA (Arrange Act Assert)](https://medium.com/@pjbgf/title-testing-code-ocd-and-the-aaa-pattern-df453975ab80) pattern when writing unit tests

Try to adhere to the [testing pyramid] (https://martinfowler.com/articles/practical-test-pyramid.html) principle. Note that different test types require different tools. For end-to-end (e2e) testing you can try using [Selenium](https://www.selenium.dev/) or [Cypress](https://www.cypress.io/how-it-works/). Integration tests can be added via `testcontainers`.

# ⚙️Configuration & Infrastructure around Code
* Have `docker` and `docker-compose` installed on your local machine
* There is a Dockerfile in your repository which can be used to build your app into a `Docker container`
  * [Best practices when writing a Dockerfile for a Ruby application](https://lipanski.com/posts/dockerfile-ruby-best-practices) (although, these tips can be useful for other languages)
  * [Google Cloud: Best practices for building containers](https://cloud.google.com/architecture/best-practices-for-building-containers)
* All app dependencies are listed in a `docker-compose.yml` file
* Set up and launch ofr your application should be as easy nad straightforward. It is possible that you may need to write some additional `base/zsh/powershell` scripts
* [Your application should have multiple environments (development, prod, test)](https://12factor.net/dev-prod-parity)
* Для `production` сборки приложения используется рекомендуемый application сервер, например:
   * Puma for Ruby
   * Gunicorn3 for Python
   * Undertow for Java

При описании конфигурации приложения используйте принципы 12factor. Изображение взято из статьи: [12 Factor App Revisited](https://architecturenotes.co/12-factor-app-revisited/)
<img src="https://raw.githubusercontent.com/abstractart/how-to-develop-perfect-crud/main/12-Factor-app-revised.jpg">


# API Design
* Используй конвенции [REST](https://www.freecodecamp.org/news/rest-api-best-practices-rest-endpoint-design-examples/) как фундамент при именовании путей, типов операций и выборе статусов ответов API  
* Формат данных: JSON (если не требуется другого)
* В репозитории есть возможность открыть [Swagger](https://swagger.io/) спецификацию для знакомства с API
  * Её можно написать самостоятельно
  * А можно генерировать c помощью утилит: [rswag (Rails)](https://github.com/rswag/rswag), [safrs (Flask)](https://github.com/thomaxxl/safrs), [echo-swagger (Echo/Golang)](https://github.com/swaggo/echo-swagger)

Если считаешь что связка REST+JSON не подходит под задачу, или по заданию требуется другой формат, то стоит изучить альтернативы:
- [SOAP](https://www.w3.org/TR/soap12-part1/)
- [gRPC](https://grpc.io/)
- [GraphQL](https://graphql.org/)

# Authorization & Authentification
**Аутентификация** – процедура проверки подлинности, например, проверка подлинности пользователя путем сравнения введенного им пароля с паролем, сохраненным в базе данных.

В качестве аутентификации по API можно использовать:
* HTTP Basic Auth (простой путь)
* JSON Web Tokens (посложнее)

**Авторизация** – предоставление определенному лицу прав на выполнение определенных действий.
Например: пользователь которого забанил администратор не может публиковать комментарии к постам (хотя он прошел аутентификацию на сайте).

Примеры библиотек:
- [Pundit for Ruby](https://github.com/varvet/pundit)
- [Casbin (Many languages supported)](https://github.com/casbin/casbin)

# MVC Explanation
Цель: разделить обязанности в коде между компонентами. MVC это один из вариантов достижения цели и не требует от разработчика сильной когнитивной нагрузки (по сравнению с другими подходами)
<img src="https://github.com/abstractart/how-to-develop-perfect-crud/blob/main/mvc-with-service.png?raw=true">
## Controller
- Принимает тело запроса, валидирует его на соответствие API
- Проверяет authorization + authentification 
- Вызывает Service, передает ему данные
- На основе возвращаемого значения от Service вызывает код формирующий нужный ответ API (через View)
## Model
* Хранит только описание схемы данных и связи с другими моделями
* Бизнес логики хранит по минимуму а лучше не хранит вообще
* Используется для того чтобы делать запросы к БД на чтение и запись
## Service
* Принимает данные от контроллера, валидирует тело
* Использует Model для чтения или записи данных в БД.
* Отвечает за бизнес-логику приложения
## View
* Отвечает за то чтобы на основе данных сформировать API ответ.

# 📐✏️👷‍♀️Architecture, Design Patterns, Refactoring, etc
После того как MVC стал открытой книгой стоит углубиться и изучить:
- подходы к построению архитектуры приложения
- принципы которые помогают писать код устойчивый к изменениям.

Бесплатные ресурсы, которые рекомендую для старта:
- [Categorized overview of programming principles & design patterns](https://github.com/webpro/programming-principles)
- [Refactoring Patterns and Design Patters Reference](https://refactoring.guru)
- [Summary of "Clean code" by Robert C. Martin](https://gist.github.com/wojteklu/73c6914cc446146b8b533c0988cf8d29)
- [Summary of "Clean Architecture" by Robert C. Martin](https://gist.github.com/ygrenzinger/14812a56b9221c9feca0b3621518635b)

# 🔒CRUD: Validations
Перед тем как сохранять данные в БД обязательно:
- отвалидируйте данные на тип (там где ожидается строка пришла строка, где int там int итп)
- и соответствие тела запроса API (если пользователь отправил поля которые не имеет права отпралять в БД мы должны их игнорировать)
# CRUD: Database
* Используйте ORM (или что-то подобное), если в задании не указано что нужно писать чистый SQL.
  * Проще для старта
  * Безопасно (Большинство ORM предоставлют защиту от SQL injections из коробки)
* Используйте механизм миграций чтобы создавать таблицы и другие сущности в вашей БД (Rails Migrations, Flask-Migrate, etc)
* При описании таблиц важно сразу указать всем столбцам необходимые constraints (NULLABLE, DEFAULT VALUE, UNIQUE, PRIMARY KEY)
* При описании таблиц важно сразу указать индексы для столбцов по которым ожидается поиск.
* Для защиты API от перебора можно использовать как PRIMARY KEY `uuid` вместо `serial`


P.S. При описании миграций полезно подсматривать [сюда](https://github.com/ankane/strong_migrations), чтобы не написать миграцию которая может заблокировать БД.
# CRUD: Operations
## LIST (HTTP GET)
* Для каждого ресурса в ответе должно присутствовать ID.
* Ресурсы должны быть отсортированными по какому либо признаку, например по времени создания.
* API должен поддерживать пагинацию (чтобы не возвращать все сущности из БД за раз) [Разбор вариантов пагинации](https://dev.to/appwrite/this-is-why-you-should-use-cursor-pagination-4nh5)
* Количество запросов к БД в рамках запроса должно быть фиксированным (Отсутствует [N+1 проблема](https://stackoverflow.com/questions/97197/what-is-the-n1-selects-problem-in-orm-object-relational-mapping))

API не должно возвращать все поля модели.
Пример: если наше API возвращает список постов то оно должно возвращать:
  * ID
  * Название поста
  * Имя автора
  * Первые несколько предложений статьи (превью)

Полный текст поста для этого эндпоинта не нужен.

## READ (HTTP GET)
* Возвращаем полностью ресурс со всеми полями, ничего особенного
## CREATE (HTTP POST)
* Валидируем данные на предмет полей которые пользователь не имеет права изменять в БД а следовательно передавать.
* Делаем в БД INSERT
* Возвращаем в ответ ID и содержимое.

Задачи со звездочкой:
- Убедиться что реализованное API идемпотентно: [Подробнее](https://habr.com/ru/company/yandex/blog/442762/)
- Настроить Rate Limiter чтобы защитить БД от спама и мусора
## UPDATE (HTTP PUT/PATCH)
* Разобраться в чем отличие между PUT и PATCH в HTTP
* Валидировать тело запроса на предмет полей которые пользователь не имеет права изменять в БД а следовательно передавать.
* Проверка права на редактирование у пользователя
  * Например API не должно позволять пользователю редактировать чужие комментарии :)
* Реализовать обновление согласно выбранному методу

## DESTROY (HTTP DELETE)
* Реализовать удаление предварительно проверив наличие сущности в БД и права на удаление у пользователя

Дополнительно может быть полезно: реализовать soft удаление (скрываем от пользователя, оставляем в БД)
# External API Calls, Long-running tasks (And why we need message queue)
Если в рамках API требуется:
- выполнять запросы к внешним системам
- генерировать отчеты/выполнять долгие запросы к БД 
то стоит подумать о том чтобы делать эти операции за пределами HTTP запроса. 

Для этого может понадобиться очередь (Queue) в которую можно будет добавлять задачу (Task). 

Примеры высокоуровневых библиотек которых решают задачу с постановкой, чтением и обработкой задач:
* Celery for Python (Задачи хранятся в `Redis`)
* Sidekiq for Ruby (Задачи хранятся в `Redis`)

Стоит отметить, что `Redis` это не единственный вариант для хранения очереди + не для всех задач он подходит.
Поэтому полезно изучить как минимум 2 более продвинутых варианта для хранения и обработки очередей: `RabbitMQ` и `Kafka`.

Доп.ссылки:
- [RabbitMQ и Apache Kafka: что выбрать](https://slurm.io/tpost/phdmogo9y1-rabbitmq-i-apache-kafka-chto-vibrat-i-mo)
- [Latency, throughput, and availability: system design interview concepts](https://igotanoffer.com/blogs/tech/latency-throughput-availability-system-design-interview) - Подробнее о том, почему так важно чтобы HTTP запросы были быстрыми

# 📈Logs and Metrics
- Настроить Prometheus метрики с информацией о состоянии HTTP API и райнтайме приложения. Рекомендуется использовать готовые пакеты, которые собирают метрики о работе приложения по методикам [RED (Rate Error Duration)](https://www.infoworld.com/article/3638693/the-red-method-a-new-strategy-for-monitoring-microservices.html) и [USE (Utilization Saturation Errors)](https://www.brendangregg.com/usemethod.html):
  - [prometheus, promauto, promhttp для Go](https://prometheus.io/docs/guides/go-application/)
  - [starlette-prometheus для Python](https://github.com/perdy/starlette-prometheus)
- [Логи должны писаться только в stdout](https://12factor.net/logs)
- [Логи должны иметь строгий формат, например это может быть JSON](https://coralogix.com/blog/json-logging-why-how-what-tips/)


# 🛡️Security
- Убедись, что не используешь версии библиотек в которых есть уязвимости, проверять это можно автоматически с помощью утилит, например:
  - [bundler-audit for Ruby](https://github.com/rubysec/bundler-audit)
  - [pip-audit for Python](https://pypi.org/project/pip-audit/)
  - [local-php-security-checker for PHP](https://github.com/fabpot/local-php-security-checker) или команда `symfony check:security`, если используется фреймворк [Symfony](https://symfony.com/)
- Настрой dependabot, который будет автоматически обновлять версии библиотек
- Убедись, что приложение достаточно защищено от актуальных уязвимостей - [OWASP TOP 10](https://owasp.org/www-project-top-ten/). Помочь в этом нелегком деле может [чеклист №1](https://github.com/shieldfy/API-Security-Checklist) и  [№2 (с примерами на Ruby on Rails)](https://github.com/brunofacca/zen-rails-security-checklist)
# CORS Headers (Cross-Origin Resource Sharing)
Если запросы к твоему API будут делать из браузерных скриптов, например Single Page Aplication, построенных на современных Javascript фреймворках (React, Angular, Vue.js) и домен API будет отличаться от домена клиентского приложения, то нужно в API добавить CORS заголовки, чтобы браузер не блокировал ответы от API.

Обычно, модуль для настройки CORS заголовков есть в http фреймворке и можно использовать его как по дефолту, так и с более тонкими настройками по необходимости. Например:
- [Go Echo](https://echo.labstack.com/middleware/cors/)
- [Javascript ExpressJS](https://expressjs.com/en/resources/middleware/cors.html)

Подробнее про CORS заголовки можно прочитать [здесь](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

# WIP: Transactions, Locks, Isolation Levels, ACID
# WIP: Cache
# WIP: Full Text Search
