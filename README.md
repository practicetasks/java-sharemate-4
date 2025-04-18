# Техническое задание

Ваше приложение для шеринга вещей почти готово! В нём уже реализована вся нужная функциональность — осталось добавить
несколько технических усовершенствований.

## Ставим проблему

Пользователей приложения `ShareMate` становится больше. Вы рады этому, но замечаете, что не всё идёт гладко: приложение
работает медленнее, пользователи чаще жалуются, что их запросы подолгу остаются без ответа.

После небольшого самостоятельного исследования вы начинаете понимать, в чём дело. Пользователи учатся программировать —
совсем так же, как и вы! Некоторые из них теперь используют ваше приложение через другие программы: собственноручно
написанные интерфейсы, боты… Чего они только не придумали!

Не все эти программы работают правильно. В `ShareMate` поступает много некорректных запросов — например, с невалидными
входными данными, в неверном формате или просто дублей. Ваше приложение тратит ресурсы на обработку каждого из
запросов, и в результате его работа замедляется. Пришло время разобраться с этим!

### Ищем решение

В реальной разработке для решения подобных проблем часто применяется микросервисная архитектура. Можно вынести часть
приложения, с которой непосредственно работают пользователи, в отдельное небольшое
приложение и назвать его, допустим, **gateway** (англ. «шлюз»). В нём будет выполняться вся валидация запросов —
некорректные будут исключаться.

Поскольку для этой части работы не требуется базы данных и каких-то особых ресурсов, приложение `gateway` будет
легковесным. При необходимости его получится легко масштабировать. Например, вместо одного экземпляра `gateway` можно
запустить целых три — чтобы справиться с потоком запросов от пользователей.

После валидации в `gateway` запрос будет отправлен основному приложению, которое делает всю реальную работу — в том
числе обращается к базе данных. Также на стороне `gateway` может быть реализовано кэширование: например, если один и тот
же запрос придёт два раза подряд, `gateway` будет самостоятельно возвращать предыдущий ответ без обращения к основному
приложению.

### Формулируем задачу

* Разбить приложение `ShareMate` на два — `shareMate-server` и `shareMate-gateway`. Они будут общаться друг с другом
  через REST. Вынести в `shareMate-gateway` всю логику валидации входных данных — кроме той, которая требует работы с
  БД.
* Настроить запуск `ShareMate` через Docker. Приложения `shareMate-server`, `shareMate-gateway` и база данных PostgreSQL
  должны запускаться в отдельном Docker-контейнере каждый. Их взаимодействие должно быть настроено через Docker Compose.

Приложение `shareMate-server` будет содержать всю основную логику и почти полностью повторять приложение, с которым вы
работали ранее, — за исключением того, что можно будет убрать валидацию данных в контроллерах.

Во второе приложение `shareMate-gateway` нужно вынести контроллеры, с которыми непосредственно работают пользователи, —
вместе с валидацией входных данных.

Каждое из приложений будет запускаться как самостоятельное Java-приложение, а их общение будет происходить через REST.
Чтобы сделать запуск и взаимодействие приложений более предсказуемым и удобным, разместите каждое из них в своём
Docker-контейнере. Также не забудьте вынести в Docker-контейнер базу данных.

### Ещё несколько технических моментов

Вам нужно разбить одно приложение `ShareMate` на два так, чтобы оба остались в том же репозитории и собирались одной
Maven-командой. Реализовать подобный механизм в Maven помогают **многомодульные проекты** (англ. _multi-module
project_). Такие проекты содержат в себе несколько более мелких подпроектов.

В нашем случае каждый из подпроектов будет представлять собой самостоятельное Java-приложение. Вообще же подпроект может
содержать любой набор кода или других сущностей, которые собираются с помощью Maven. Это может быть, например, набор
статических ресурсов — HTML-файлы, изображения и так далее.

Многомодульный проект содержит один родительский `pom`-файл для всего проекта, в котором перечисляются все модули или
подпроекты. Также для каждого из модулей создается собственный `pom`-файл со всей информацией о сборке отдельного
модуля. Когда в корневой директории проекта запустится команда сборки (например, `mvn clean install`), Maven соберёт
каждый из модулей и положит результирующий `jar`-файл в директорию `target` соответствующего модуля.

Подготовьте `Dockerfile` для каждого из сервисов — `shareMate-server` и `shareMate-gateway`. Шаблон для этих файлов
расположен в корневой папке каждого модуля, его содержимое будет таким же, как и в теме про Docker. Затем опишите
настройки развёртывания контейнеров в файле `docker-compose.yaml` в корне проекта. Конфигурация развёртывания должна
включать три контейнера для следующих сервисов: `shareMate-server`, `shareMate-gateway` и `postgresql`.

Убедитесь, что ваше приложение успешно запускается командой `docker-compose up` и пользователи, как и прежде, могут
создавать и бронировать вещи.

### Тестирование

Как и всегда, воспользуйтесь нашей [Postman-коллекцией](postman.json), чтобы протестировать работу приложения.