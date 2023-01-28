# How to develop perfect CRUD
## Что это?
Данная статья рассказывает о наборе правил которые полезно знать и применять при разработке бэкэнд приложения.
Он будет вам полезен, если вы:
- Хотите стартануть новый проект с нуля и хотите сразу заручиться набором хороших практик.
- Получили перед или после собеседования тестовое задание и всерьез настроены реализовать его. 

Я написал этот гайд с целью собрать все хорошие практики и опыт в одну статью чтобы можно было использовать ее как чеклист при старте новых проектов. 
### Здесь только про Backend? 
Так как автор имеет экспертизу только в бэкэнде то и большая часть советов будет относиться к разработке бэкэнд приложений. В прочем многие из советов будут полезны и для других специальностей.
### Пригодится ли мне это на работе?
Данные советы могут быть полезны не только при разработке тестового задания, но и при разработке новых фич или старте нового приложения с нуля.
### Слишком много всего, это точно нужно?
Чем больше из всего набора фич и практик вы реализуете тем более качественным будет результат. Реализовать все может быть избыточно и потребовать много времени, рассчитайте время и силы.
## Советы
### Репозиторий
* Код должен храниться в публичном/приватном Git репозитории (Github / Gitlab / Bitbucket)
* README должен содержать информацию о проекте, инструменты и технологии, инструкцию по настройке и запуску приложения
* Должен быть настроен Continuous Integration (Gitlab CI / Github Actions)
  * Интегрируй прогон тестов  в CI для каждой фича ветки и мастер ветки
  * Интегрируй прогон линтера в CI для каждой фича ветки и мастер ветки.
* Будет огромным плюсом если будет настроен Continuous Delivery - деплой приложения на сервер
* Осмысленная история коммитов + использование feature branches, pull requests
* Необязательно: настроенный [dependabot](https://docs.github.com/ru/code-security/dependabot/working-with-dependabot)
### Инструменты и Code Style
Перед разработкой приложения:
* Настроен редактор или IDE:
  * VS Code
  * Visual Studio
  * PyCharm
  * IDEA
  * Vim, emacs
* Установлен EditorConfig плагин для твоего редактора
* Установлены наиболее популярные инструменты по верификации качества кода, например
  * Rubocop for Ruby
  * Pylint/Black/PEP8 for Python
### Тесты
* Установлены библиотеки для написания тестов различных видов (unit, integration). Например:
   * Pytest for Python
   * RSpec for Ruby
   * Testify, testcontainers for Golang
* После прогона тестов автоматически считается test coverage

### Инфраструктура вокруг кода
Прежде всего: установлены Docker и вocker-compose


Далее:
- Заверни свое приложение в Docker Container ([как это делать правильно](https://cloud.google.com/architecture/best-practices-for-building-containers)
- Опиши все зависимости своего приложения (PostgreSQL, S3, Redis etc) как Docker контейнеры.
- Настройка приложения и запуск должны делаться максимально просто и прозрачно (желательно в 1 команду)
#### Конфигурация
* Настройте application сервер для production сборки приложения:
   * Puma for Ruby
   * Gunicorn3 for Python
   * Undertow for Java
* При описании конфигурации приложения используйте советы из https://12factor.net

### API
* API должен быть описан по [REST](https://www.freecodecamp.org/news/rest-api-best-practices-rest-endpoint-design-examples/) (если не требуется иначе)
* Должна быть настроена [Swagger](https://swagger.io/) спецификация которую можно открыть в браузере
* Формат данных: JSON (если не требуется другого)
* API должен иметь настроенные таймауты на запросы.

### Распределение логики в коде (MVC)
Цель: разделить обязанности в коде между компонентами: 
#### Controller
- Принимает тело запроса, валидирует его на соответствие API
- Проверяет authorization + authentification 
- Вызывает Service, передает ему данные
- На основе возвращаемого значения от Service вызывает код формирующий нужный ответ API (через View)
#### Model
* Хранит только описание схемы данных и связи с другими моделями
* Бизнес логики хранит по минимуму а лучше не хранит вообще
* Используется для того чтобы делать запросы к БД на чтение и запись
#### Service
* Принимает данные от контроллера, валидирует тело, проверяет права доступа (авторизацию, аутентификацию)
* Использует Model для чтения или записи данных в БД.
* Отвечает за бизнес-логику приложения
#### View
* Отвечает за то чтобы на основе данных сформировать API ответ.
### Авторизация и аутентификация
**Аутентификация** – процедура проверки подлинности, например, проверка подлинности пользователя путем сравнения введенного им пароля с паролем, сохраненным в базе данных.
В качестве аутентификации по API можно использовать:
* HTTP Basic Auth (простой путь)
* JSON Web Tokens (посложнее)

**Авторизация** – предоставление определенному лицу прав на выполнение определенных действий.
Также важно реализовать авторизацию (означает доступ к ресурсу для пользователя). 
Например: пользователь которого забанил администратор не может публиковать комменты (хотя он прошел аутентификацию на сайте).


### CRUD: Validations
Перед тем как сохранять данные в БД обязательно:
- отвалидируйте данные на тип (там где ожидается строка пришла строка, где int там int итп)
- и соответствие тела запроса API (если пользователь отправил поля которые не имеет права отпралять в БД мы должны их игнорировать)
### CRUD: Database
* Всегда используйте ORM (это проще и безопаснее) если в задании не указано что нужно писать чистый SQL
* Используйте механизм миграций чтобы создавать таблицы и другие сущности в вашей БД (Rails Migrations, Flask-Migrate, etc)
* При описании таблиц важно сразу указать всем столбцам необходимые constraints (NULLABLE, DEFAULT VALUE, UNIQUE, PRIMARY KEY)
* При описании таблиц важно сразу указать индексы для столбцов по которым ожидается поиск.
* Для защиты API от перебора можно использовать как PRIMARY KEY `uuid` вместо `serial`
* 
P.S. При описании миграций полезно подсматривать [сюда](https://github.com/ankane/strong_migrations), чтобы не написать миграцию которая может заблокировать БД.
### CRUD: Operations
#### LIST (HTTP GET)
* Для каждого ресурса в ответе должно присутствовать ID.
* Ресурсы должны быть отсортированными по какому либо признаку, например по времени создания.
* API должен поддерживать пагинацию (чтобы не возвращать все сущности из БД за раз) [Разбор вариантов пагинации](https://dev.to/appwrite/this-is-why-you-should-use-cursor-pagination-4nh5)
* Количество запросов к БД должно быть фиксированным и не зависеть от количества запрашиваемых данных (Отсутствует [N+1 проблема])(https://stackoverflow.com/questions/97197/what-is-the-n1-selects-problem-in-orm-object-relational-mapping))

API не должно возвращать все поля модели.
Пример: если наше API возвращает список постов то оно должно возвращать:
  * ID
  * Название поста
  * Имя автора
  * Первые несколько предложений статьи (превью)

Полный текст поста для этого эндпоинта не нужен.

#### READ (HTTP GET)
* Возвращаем полностью ресурс со всеми полями, ничего особенного
#### CREATE (HTTP POST)
* Валидируем данные на предмет полей которые пользователь не имеет права изменять в БД а следовательно передавать.
* Делаем в БД INSERT
* Возвращаем в ответ ID и содержимое.
Дополнительно:
- убедиться что реализованное API идемпотентно: [https://habr.com/ru/company/yandex/blog/442762/](https://habr.com/ru/company/yandex/blog/442762/)
#### UPDATE (HTTP PUT/PATCH)
* Разобраться в чем отличие между PUT и PATCH в HTTP
* Реализовать обновление согласно выбранному методу
* Валидировать ввод на предмет полей которые пользователь не имеет права изменять в БД а следовательно передавать.
#### DESTROY (HTTP DELETE)
* Реализовать удаление предварительно проверив наличие сущности в БД и права на удаление у пользователя
Дополнительно: реализовать soft удаление (скрываем от пользователя, оставляем в БД)
### Взаимодействие с внешними сервисами, трудоемкие операции
Если требуется в рамках API
- выполнять запросы к внешним системам
- генерировать отчеты

Стоит задаться вопросами:
* Можно ли это делать в отложенном режиме. Для этого может понадобиться очередь, например:
  * Celery for Python
  * Sidekiq for Ruby

### Метрики, Логи
- Настроить Prometheus Collector и HTTP handler через который можно просмотреть состояние метрик.

### WIP: Cache
### WIP: Full Text Search

