# How to develop perfect CRUD
[<img src="https://img.shields.io/badge/channel-telegram-blue">](https://t.me/careerunderhood) [<img src="https://img.shields.io/badge/message-telegram-blue">](https://t.me/ea_kozlov)

## Что это?
Данная статья - попытка собрать воедино все хорошие практки которые полезно знать и применять при разработке бэкэнд приложения. 

Её можно использовать как полноценный чеклист который будет вам полезен, если вы:
- Начинаете новый проект с нуля и сразу хотите заручиться набором хороших практик.
- Получили тестовое задание и всерьез настроены реализовать его и доказать чего стоите.
### Здесь только про Backend? 
Так как автор имеет экспертизу только в бэкэнде то и большая часть советов будет относиться к разработке бэкэнд приложений. В прочем многие из советов будут полезны и для других специальностей.
### Слишком много всего, это точно нужно?
Чем больше из всего набора фич и практик вы реализуете тем более качественным будет результат. Реализовать все может быть избыточно и потребовать много времени, рассчитайте время и силы.

## Table of Contents
<!-- TOC start -->
- [Repository](#repository)
- [Code Style](#code-style)
- [Tests](#tests)
- [Infrastructure around Code](#infrastructure-around-code)
- [Configuration](#configuration)
- [API Design](#api-design)
- [Authorization & Authentication](#authorization--authentication)
- [MVC Explanation](#mvc-explanation)
  * [Controller](#controller)
  * [Model](#model)
  * [Service](#service)
  * [View](#view)
- [CRUD: Validations](#crud-validations)
- [CRUD: Database](#crud-database)
- [CRUD: Operations](#crud-operations)
  * [LIST (HTTP GET)](#list-http-get)
  * [READ (HTTP GET)](#read-http-get)
  * [CREATE (HTTP POST)](#create-http-post)
  * [UPDATE (HTTP PUT/PATCH)](#update-http-putpatch)
  * [DESTROY (HTTP DELETE)](#destroy-http-delete)
- [External API Calls, Long-running tasks](#external-api-calls-long-running-tasks)
- [Logs and Metrics](#logs-and-metrics)
- [WIP: Cache](#wip-cache)
- [WIP: Full Text Search](#wip-full-text-search)
<!-- TOC end -->
## Repository
* Код должен храниться в публичном/приватном Git репозитории (Github / Gitlab / Bitbucket)
* README должен содержать информацию о проекте, инструменты и технологии, инструкцию по настройке и запуску приложения
* Читаемая история коммитов. Можно использовать практику [Conventional commits](https://www.conventionalcommits.org/ru/v1.0.0/)
* Используйте feature branches, pull requests
* Должен быть настроен Continuous Integration (Gitlab CI / Github Actions)
* Для каждой feature ветки и master ветки должен быть настроен:
  * запуск тестов
  * запуск линтера
* Будет огромным плюсом если настроен Continuous Delivery - деплой приложения в одно или несколько окружений. (test/stage/prod)

* Необязательно: настроенный [dependabot](https://docs.github.com/ru/code-security/dependabot/working-with-dependabot)
## Code Style
Перед разработкой приложения:
* Настроен редактор или IDE:
  * VS Code
  * Visual Studio
  * PyCharm
  * IDEA
  * Vim, emacs
* Установлен [EditorConfig](https://editorconfig.org/) плагин для твоего редактора
* Установлены наиболее популярные инструменты по верификации качества кода, например
  * Rubocop for Ruby
  * Pylint/Black/PEP8 for Python
## Tests
* Установлены библиотеки для написания тестов различных видов (unit, integration). Например:
   * Pytest for Python
   * RSpec for Ruby
   * Testify, testcontainers for Golang
* После прогона тестов автоматически считается test coverage
* Пишите unit-тесты по паттерну [AAA (Arange Act Assert)](https://medium.com/@pjbgf/title-testing-code-ocd-and-the-aaa-pattern-df453975ab80)

Старайтесь покрывать ваш код по [пирамиде тестирования](https://martinfowler.com/articles/practical-test-pyramid.html), так чтобы было больше всего было unit-тестов, меньше интеграционных тестов и при необходимости были end-to-end тесты. Обратите внимание, что для тестов разного уровня могут использоваться разные инструменты. Для end to end тестирования можно использовать [Selenium](https://www.selenium.dev/) или [Cypress](https://www.cypress.io/how-it-works/). Для интеграционных удобно использовать `testcontainers`

## Infrastructure around Code
- Установлены Docker и docker-compose
- В репозитории есть Dockefile с помощью которого можно собрать приложение в Docker Container ([как это делать правильно](https://cloud.google.com/architecture/best-practices-for-building-containers))
- Все зависимости приложения (`PostgreSQL`, `S3`, `Redis`, `Kafka`, `RabbitMQ`) описаны в `docker-compose.yml`
- Настройка приложения и запуск должны делаться максимально просто и прозрачно (желательно в 1 команду)

## Configuration
* [Приложение должно иметь несколько окружений (development, prod, test)](https://12factor.net/dev-prod-parity)
* Настроен application сервер для production сборки приложения:
   * Puma for Ruby
   * Gunicorn3 for Python
   * Undertow for Java
* При описании конфигурации приложения используйте советы из https://12factor.net

## API Design
* Используй семантику [REST](https://www.freecodecamp.org/news/rest-api-best-practices-rest-endpoint-design-examples/) как фундамент при описании API 
* Формат данных: JSON (если не требуется другого)
* В репозитории есть возможность открыть [Swagger](https://swagger.io/) спецификацию для знакомства с API
  * Её можно написать самостоятельно
  * А можно генерировать c помощью утилит: [Rails (Ruby)](https://github.com/rswag/rswag), [Flask (Python)](https://github.com/thomaxxl/safrs), [Echo (Golang)](https://github.com/swaggo/echo-swagger)

Если считаешь что связка REST+JSON не подходит под задачу, или по заданию требуется другой формат, то стоит изучить альтернативы:
- gRPC
- GraphQL

## Authorization & Authentication
**Аутентификация** – процедура проверки подлинности, например, проверка подлинности пользователя путем сравнения введенного им пароля с паролем, сохраненным в базе данных.

В качестве аутентификации по API можно использовать:
* HTTP Basic Auth (простой путь)
* JSON Web Tokens (посложнее)

**Авторизация** – предоставление определенному лицу прав на выполнение определенных действий.
Например: пользователь которого забанил администратор не может публиковать комменты (хотя он прошел аутентификацию на сайте).

Примеры библиотек:
- [Pundit for Ruby](https://github.com/varvet/pundit)
- [Casbin (Many languages supported)](https://github.com/casbin/casbin)

## MVC Explanation
Цель: разделить обязанности в коде между компонентами: 
<img src="https://harbinger-systems.com/blog/wp-content/uploads/2015/08/MVC-Architecture-With-Service-Layer1-1024x477.png">
### Controller
- Принимает тело запроса, валидирует его на соответствие API
- Проверяет authorization + authentification 
- Вызывает Service, передает ему данные
- На основе возвращаемого значения от Service вызывает код формирующий нужный ответ API (через View)
### Model
* Хранит только описание схемы данных и связи с другими моделями
* Бизнес логики хранит по минимуму а лучше не хранит вообще
* Используется для того чтобы делать запросы к БД на чтение и запись
### Service
* Принимает данные от контроллера, валидирует тело
* Использует Model для чтения или записи данных в БД.
* Отвечает за бизнес-логику приложения
### View
* Отвечает за то чтобы на основе данных сформировать API ответ.

## CRUD: Validations
Перед тем как сохранять данные в БД обязательно:
- отвалидируйте данные на тип (там где ожидается строка пришла строка, где int там int итп)
- и соответствие тела запроса API (если пользователь отправил поля которые не имеет права отпралять в БД мы должны их игнорировать)
## CRUD: Database
* Используйте ORM (или что-то подобное), если в задании не указано что нужно писать чистый SQL.
  * Проще для старта
  * Безопасно (Большинство ORM предоставлют защиту от SQL injections из коробки)
* Используйте механизм миграций чтобы создавать таблицы и другие сущности в вашей БД (Rails Migrations, Flask-Migrate, etc)
* При описании таблиц важно сразу указать всем столбцам необходимые constraints (NULLABLE, DEFAULT VALUE, UNIQUE, PRIMARY KEY)
* При описании таблиц важно сразу указать индексы для столбцов по которым ожидается поиск.
* Для защиты API от перебора можно использовать как PRIMARY KEY `uuid` вместо `serial`


P.S. При описании миграций полезно подсматривать [сюда](https://github.com/ankane/strong_migrations), чтобы не написать миграцию которая может заблокировать БД.
## CRUD: Operations
### LIST (HTTP GET)
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

### READ (HTTP GET)
* Возвращаем полностью ресурс со всеми полями, ничего особенного
### CREATE (HTTP POST)
* Валидируем данные на предмет полей которые пользователь не имеет права изменять в БД а следовательно передавать.
* Делаем в БД INSERT
* Возвращаем в ответ ID и содержимое.

Задачи со звездочкой:
- Убедиться что реализованное API идемпотентно: [Подробнее](https://habr.com/ru/company/yandex/blog/442762/)
- Настроить Rate Limiter чтобы защитить БД от спама и мусора
### UPDATE (HTTP PUT/PATCH)
* Разобраться в чем отличие между PUT и PATCH в HTTP
* Валидировать тело запроса на предмет полей которые пользователь не имеет права изменять в БД а следовательно передавать.
* Проверка права на редактирование у пользователя
  * Например API не должно позволять пользователю редактировать чужие комментарии :)
* Реализовать обновление согласно выбранному методу

### DESTROY (HTTP DELETE)
* Реализовать удаление предварительно проверив наличие сущности в БД и права на удаление у пользователя

Дополнительно может быть полезно: реализовать soft удаление (скрываем от пользователя, оставляем в БД)
## External API Calls, Long-running tasks
Если в рамках API требуется
- выполнять запросы к внешним системам
- генерировать отчеты/выполнять долгие запросы к БД 

стоит подумать о том чтобы делать эти операции за пределами HTTP запроса. Если коротко: чем дольше запросы к вашему API тем меньше клиентов сможет обслужить сервер в единицу времени. Подробнее: [Latency, throughput, and availability: system design interview concepts](https://igotanoffer.com/blogs/tech/latency-throughput-availability-system-design-interview)

Для этого может понадобиться очередь, например:
* Celery for Python
* Sidekiq for Ruby

## Logs and Metrics
- Настроить Prometheus метрики с информацией о состоянии HTTP API и райнтайме приложения. Рекомендуется использовать готовые пакеты, которые собирают метрики о работе приложения по методикам [RED (Rate Error Duration)](https://www.infoworld.com/article/3638693/the-red-method-a-new-strategy-for-monitoring-microservices.html) и [USE (Utilization Saturation Errors)](https://www.brendangregg.com/usemethod.html):
  - [prometheus, promauto, promhttp для Go](https://prometheus.io/docs/guides/go-application/)
  - [starlette-prometheus для Python](https://github.com/perdy/starlette-prometheus)
- [Логи должны писаться только в stdout](https://12factor.net/logs)


## WIP: Transactions, Locks, Isolation Levels, ACID
## WIP: Security
## WIP: Cache
## WIP: Full Text Search

