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
На данный момент на нетфликсе порядка 36000 часов контента. Каждая единица контента кодируется в более чем 50 различных версий в зависимости от разрешения видео и качества звука. В 2013 году размер хранилища составлял 3 ПБ. Учитывая, что с тех пор контента стало гораздо больше, возьмем 6 ПБ для MVP.


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
| SD     | 12.5 GBit |
| HD      | 20.8 GBit     | 
| 4K | 104 GBit    |

В 2016 году в день на нетфликс заходило порядка 33 миллионов человек. Тогда трафик в день:

| Качество       | Размер трафика |
| ------------- |:-------------:| 
| SD | 402832.06  TBit |
| HD | 670312.54 TBit| 
| 4K | 3351562.71 TBit|

Тогда трафик в секунду (учитывая, что нетфликс смотрят по всему миру в разное время):

| Качество       | Размер трафика |
| ------------- | :-------------: | 
| SD | 4,66 TBit/s |
| HD | 7,76 TBit/s| 
| 4K | 38,79 TBit/s|

Пусть 60 % пользователей смотрят контент в HD, 20% в 4к и 20% в SD.
Тогда трафик на стриминг ***13,34 Tb/s***

Рассчитаем пиковый трафик на стриминг.
За 2 часа каждый пользователь посмотрит:
| Качество       | Размер трафика |
| ------------- |:-------------:| 
| SD     | 21.125 GBit |
| HD      | 35,152 GBit     | 
| 4K | 178,76 GBit    |

Пусть в пиковые часы на сайт зашло 20% пользователей - это 6,6млн человек.
| Качество       | Размер трафика |
| ------------- |:-------------:| 
| SD     | 139425 TBit |
| HD      | 232003,2 TBit     | 
| 4K | 1179816 GBit    |

В секунду:
| Качество       | Размер трафика |
| ------------- |:-------------:| 
| SD     | 19,36 TBit |
| HD      | 32,22 TBit     | 
| 4K | 163,86 TBit    |

Тогда пиковый трафик:
19,36 * 0,2 + 32,22 * 0,6 + 163,86 * 0,2 = ***55,976 Tb/s***


Для загрузки одной страницы требуется примерно 18 МБ трафика. 
Тогда трафик для загрузки страницы в день:
18 * 3.85 * 33 * 10^6 / 1000 = 2286900 Гб/день = 26,46875 ГБ/с = ***211,68 GBit/s***

#### RPS
На основе своей пользовательской истории разобьем запросы на категории:

| Тип запроса       | Процент от общего числа |
| ------------- | :-------------: | 
| Страница с контентом (главная, фильмы, сериалы, избранное) | 50 % |
| Страница одного фильма/сериала | 20% |
| Выборка по жанру/тегу/актеру... | 10% |
| Поиск | 10% |
| Лайк | 4% |
| Избранное | 4% |
| Профиль | 2% |


RPS на бэкенд = 33 * 10^6 * 4 / (24 * 60 * 60) = 1666 RPS
RPS на статику = 160 * 3.85 * 33 * 10^6 = 266560 RPS


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

Схема базы данных с учетом разбиения таблиц по разным базам представлена на рисунке:
![Иллюстрация к проекту](https://github.com/Moxxx1e/technopark_highload/raw/main/img/db_diagram.png)

#### CDN

CDN-сервера расположим в ISP(провайдеры интернета) и IXP (точки обмена трафиком). Расположим сервера преимущественно в США и Западной Европе. Примерное расположение серверов:

![Иллюстрация к проекту](https://github.com/Moxxx1e/technopark_highload/raw/main/img/netflix_cdn_map.jpeg)


Для балансировки на уровне стран будем использовать GeoDNS, а внутри страны - Anycast. Делая анонсы сети из ДЦ по всей стране, с помощью BGP будет определяться кратчайший маршрут к автономной системе.

Для оптимизации чтения популярный контент (10 - 20%) можно хранить в оперативной памяти. Сервера будут пополняться с использованием проактивного кэширования, на основе данных из сервиса рекомендаций. 

Конфигурация CDN-сервера:
```
Процессор: Intel Xeon E5 2697v3

SSD: 360 TB

RAM: 512 GB

Пропускная способность: 36 Gbps
```

| Название       | CPU | RAM | SSD | Количество | 
| ------------- |:-------------:|:-------------:|:-------------:|:-------------:|
| CDN     |  |  |  | |
| Фронтенд    |  |  |  | | 
| Бэкенд |  |  |  | |
| Бэкенд |  |  |  | |


Источники:

[Netflix system design overview](https://medium.com/@narengowda/netflix-system-design-dbec30fede8d)

[Как мы в ivi переписывали etl: Flink+Kafka+ClickHouse](https://habr.com/ru/company/ivi/blog/347408/)

[Serving 100 Gbps from an Open Connect Appliance](https://netflixtechblog.com/serving-100-gbps-from-an-open-connect-appliance-cdb51dda3b99)

[Content Popularity for Open Connect](https://netflixtechblog.com/content-popularity-for-open-connect-b86d56f613b)

[How Netflix Works With ISPs Around the Globe to Deliver a Great Viewing Experience](https://about.netflix.com/en/news/how-netflix-works-with-isps-around-the-globe-to-deliver-a-great-viewing-experience)

[Open Connect Appliances](https://openconnect.netflix.com/en/appliances/#the-hardware)


### 5. Выбор технологий
В качестве основного языка программирования будем использовать Go, из-за высокой скорости работы, эффективных средств конкурентного программирования и удобной работы с зависимостями. Для обучения моделей и работы с рекомендациями подойдет Python, как зарекомендовавшее себя решение в этой области. 
Будем использовать микросервисную архитектуру, протокол взаимодействия - gRPC. Фронтенд стэк - TypeScript и React.

В качестве очереди сообщений будем использовать Kafka, а для хранения логов и отслеживания ошибок - Elastic Stack. 

Для хранения видео подойдет хранилище от Amazon - Amazon S3.

### 6. Схема проекта
![Иллюстрация к проекту](https://github.com/Moxxx1e/technopark_highload/raw/main/img/netflix_arch_fixed_2.png)


