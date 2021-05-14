## 1. Выбор темы и целевой аудитории
**Тема**: Netflix.

**Целевая аудитория**: 200 млн. пользователей по всему миру.

**Функционал MVP**: просмотр фильмов, сериалов и другого контента (шоу, стендап) в высоком качестве, подборки по жанрам, странам и т.п., система рекомендаций, поиск, добавление в избранное, система подписки.

### 2. Расчет нагрузки

#### Статистика
| Значение       | Параметр |
| ------------- |:-------------:| 
| Пользователи    | 200 млн.|
| Пользователей в день     | 33 млн.      | 
| Страниц за визит  | 3.85 | 
| Время просмотра контента | 71 минута    |
| Суммарная длительность контента | 36000 часов  |
| Событий в день | 700 млн.|

#### Размер хранилища
На данный момент на нетфликсе порядка 36000 часов контента. Каждая единица контента кодируется в более чем 50 различных версий в зависимости от разрешения видео и качества звука. Размер хранилища Netlfix - 40 петабайт.

#### Сетевой трафик
Для стриминга видео Netlfix рекомендует следующую скорость интернет-соединения.
| Качество       | Скорость соединения |
| ------------- |:-------------:| 
| SD     | 3 MBit/s |
| HD      | 5 MBit/s      | 
| 4K | 25 MBit/s    |

В среднем пользователь тратит на просмотр в день 71 минуту, тогда размер трафика от одного пользователя в день:

| Качество       | Размер трафика |
| ------------- |:-------------:| 
| SD     | 1,278 GBit |
| HD      | 2,130 GBit     | 
| 4K | 10,65 GBit    |

В 2016 году в день на нетфликс заходило порядка 33 миллионов человек. Тогда трафик в день:

| Качество       | Размер трафика |
| ------------- |:-------------:| 
| SD | 42174000 GBit |
| HD | 70290000 GBit| 
| 4K | 351450000 GBit|

Тогда трафик в секунду (учитывая, что нетфликс смотрят по всему миру в разное время):

| Качество       | Размер трафика |
| ------------- | :-------------: | 
| SD | 488 GBit/s |
| HD | 815 GBit/s| 
| 4K | 4068 GBit/s|

Пусть 60 % пользователей смотрят контент в HD, 20% в 4к и 20% в SD.
Тогда трафик на стриминг: ***1400 GBit/s***

Для загрузки одной страницы требуется примерно 18 МБ трафика. 
Тогда трафик для загрузки страницы в день:
18 * 3.85 * 33 * 10^6 / 1000 = 2286900 Гб/день = 26,46875 ГБ/с = ***211,68 GBit/s***

#### RPS
В день на Netlfix заходят 33 миллиона пользователей. В среднем пользователь посещает 3.85 страниц.
На отображение главной страницы целиком требуется около 480 запросов. Из них 160 - xhr-запросы и порядка 300 - запросы за картинками.
Тогда RPS на бэкенд = 160 * 3.85 * 33 * 10^6 = 235277 RPS

RPS на статику = 160 * 3.85 * 33 * 10^6 = 441145 RPS


Источники:

[Netflix Productionizing Spark On Yarn For ETL At Petabyte Scale](https://youtu.be/TY3_G4SFhGo?t=113)

[Simillar web overview](https://account.similarweb.com/login?returnUrl=https%3A%2F%2Fpro.similarweb.com%2F#/marketing/competitiveanalysis/overview/website-performance/netflix.com/*/643/3m?webSource=Total)

[Netflix internet connection speed recommendations](https://help.netflix.com/en/node/306)

[How long would it take to watch all of netflix](https://www.whats-on-netflix.com/news/how-long-would-it-take-to-watch-all-of-netflix/)




### 3. Логическая схема базы данных
![Иллюстрация к проекту](https://github.com/Moxxx1e/technopark_highload/raw/main/img/netflix_schema_3.png)

### 4. Физическая схема базы данных
Для хранения информации о пользователях (users), подписках и контенте будем использовать PostgreSQL. В зависимости от метрик, можно попробовать поэкспериментировать с денормализацией. Например, если число запросов за полной информацией п о конкретному фильму преобладает над запросом выборки по странам (то же касается и актеров, режиссеров, жанров), то можно хранить страны в одной таблице с контентом. Таким образом, можно снизить накладные расходы на JOIN-ы. 

Для быстрой работы с сессиями будем использовать Redis. Историю просмотров и другие пользовательские метрики будем хранить в ClickHouse, как в базе изначально предусмотренной для хранения истории пользовательский действий. Шардировать будем по первичному ключу. 

Источники:

[Netflix system design overview](https://medium.com/@narengowda/netflix-system-design-dbec30fede8d)

[Как мы в ivi переписывали etl: Flink+Kafka+ClickHouse](https://habr.com/ru/company/ivi/blog/347408/)

### 5. Выбор технологий
В качестве основного языка программирования будем использовать Go, из-за высокой скорости работы, эффективных средств конкурентного программирования и удобной работы с зависимостями. Для обучения моделей и работы с рекомендациями подойдет Python, как зарекомендовавшее себя решение в этой области. 
Будем использовать микросервисную архитектуру, протокол взаимодействия - gRPC. Фронтенд стэк - TypeScript и React.

В качестве очереди сообщений будем использовать Kafka, а для хранения логов и отслеживания ошибок - Elastic Stack. 

### 6. Схема проекта
![Иллюстрация к проекту](https://github.com/Moxxx1e/technopark_highload/raw/main/img/netflix_arch.png)


