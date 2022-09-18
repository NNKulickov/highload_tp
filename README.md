# Booking.com
[Аудитория](#) 
| [MVP](#mvp) 
| [Расчет нагрузки](#load) 
| [Логическая схема](#logic) 
| [Физическая схема](#physic)
| [Технологии](#tech)
| [Схема проекта](#schema)
| [Список серверов](#servers)
## 1. Аудитория <a name="auditory"></a>
Booking.com - мировой лидер среди OTA(до недавнего времени единственный), которым пользуются люди разных возрастов и полов по всему миру. 
На [текущий](https://www.booking.com/content/about.ru.html?aid=356980&label=gog235jc-1DCBQoggJCBWFib3V0SDNYA2jCAYgBAZgBIbgBB8gBD9gBA-gBAYgCAagCA7gCrfqYmQbAAgHSAiQyYzVkNDIxYy1hZjE0LTRjNzItOTg4NC0xMDZjMWE5OGMwNGPYAgTgAgE&sid=7a8c633e3408de454615515c7a37247d&keep_landing=1&)
момент имеет 43 языка и варианты размещения в 228 [странах](https://lesboutiquehotels.com/booking-com-reviews#reliable) мира.
Доступен на всех платформах.

Согласно [статистике](https://www.semrush.com/website/booking.com/overview/) Booking обслуживает ежемесячно ~400 млн. пользователей в среднем за год.
Доля распределения пользователей по странам следующая.

| United States | Germany | Italy | Spain | France | Other  |
|---------------|---------|-------|-------|--------|--------|
|  12.24%       | 8.36%   | 7.04% | 6.21% | 4.06%  | 62.09% |


## 2. MVP <a name="mvp"></a>
Минимальный функционал booking.com:
1. Регистрация / авторизация;
2. Поиск размещений во внутреннем хранилище и через партнеров;
3. Бронирование варианта размещения;

## 3. Расчёт нагрузки <a name="load"></a>

### Продуктовые метрики
Booking.com как любой OTA использует двойную систему поиска доступности как в своем собственном хранлище, так и у поставщиков.
Для конечного пользователя важны скорость отдачи информации и ее актуальность.

#### 1. Месячная аудиория

Месячная аудитория сервиса в среднем составляет примерно 400 миллионов людей.

#### 2. Дневная аудиория

Дневная аудитория сервиса в среднем составляет примерно 13.3 млн пользователей.

#### 3. Средний размер хранилища пользователя

Информация пользователя всегда стандартная и выглядит след образом.

#### user:

| Название   | Тип         | Размер  |
|------------|-------------|---------|
| id         | int         | 8 байт  |
| firstName  | varchar(64) | 64 байт |
| lastName   | varchar(64) | 64 байт |
| email      | varchar(64) | 64 байт |
| password   | varchar(64) | 64 байт |
| modifiedAt | Date        | 3 байт  |
| createdAt  | Date        | 3 байт  |

#### card:

| Название   | Тип         | Размер  |
|------------|-------------|---------|
| id         | int         | 8 байт  |
| number     | varchar(64) | 64 байт |
| expiration | Date        | 64 байт |
| holder     | varchar(64) | 64 байт |
| ccv        | int         | 8 байт  |
| createdAt  | Date        | 3 байт  |


#### user_card:

| Название | Тип | Размер |
|----------|-----|--------|
| userID   | int | 8 байт |
| cardID   | int | 8 байт |


#### history:

| Название  | Тип  | Размер |
|-----------|------|--------|
| userID    | int  | 8 байт |
| invID     | int  | 8 байт |
| createdAt | Date | 3 байт |

Из таблиц можно посчитать, что суммарный вес таблицы пользователя составил 518 байт.
Данные пользователя особо не растут, за исключением истории поиска. Для удобства положим, что в среднем один пользователь
путешествует не более 10ти раз в месяц. Таким образом история поездок будет расти не более чем `400x10^6x10x(8+8+3)/(1024x3) = 78`ГБ в месяц.

#### 4. Среднее количество действий пользователя по типам в день

Действие пользователя можно разделить на 3 типа:
 1. Взаимодействие с профилем
 2. Поиск
 3. Бронирование

Основное действие с профилем - это логин/логаут. (2 запроса)

При поиске пользователь проходит несколько этапов и для удобства положим, что он в среднем рассматривает 15 вариантов размещения:
  * Поиск по региону - пользователь вводит страну/город, куда он собирается, и получает список отелей с соответсвующей доступностью по региону. 1 запрос.
  * Детальный поиск доступностей - пользователь раскрывает отель и просматривает детальную информацию по доступностям. (15 запросов)

При бронировании положим, что в среднем, пользователь 2 раза бронирует и 1 раз отменяет бронирование. (3 запроса)

| Действие            | Кол-во запросов |
|---------------------|-----------------|
| Авторизация         | 2               |
| Региональный поиск  | 1               |
| Детальный поиск     | 15              |
| Бронирование        | 2               |
| Отмена бронирования | 1               |

Итого получается 21 запрос.


### Технические метрики

Помимо пользовательских данных в сервисе хранится также информация об отелях, размещениях, бронированиях.

Рассмотрим доп таблицы. 

#### inventory:

| Название    | Тип           | Размер   |
|-------------|---------------|----------|
| id          | int           | 8 байт   |
| name        | varchar(64)   | 64 байт  |
| description | varchar(64)   | 64 байт  |
| hotelName   | varchar(64)   | 64 байт  |
| longitude   | Decimal(9,6)  | 5 байт   |
| latitude    | Decimal(8,6)  | 5 байт   |
| price       | Decimal(10,2) | 9 байт   |
| rateType    | char(4)       | 4 байт   |
| previewURL  | varchar(256)  | 256 байт |
| createdAt   | Date          | 3 байт   |
| modifiedAt  | Date          | 3 байт   |

* rate_type - тип варианта [размещения](https://hoteltechreport.com/news/room-type)
  1. по кроватям (twin,double,single,queen,king) - кодируется первой буквой
  2. по размеру (standard,deluxe,join,apartment) - кодируется второй буквой
  3. по удобствам (cabana,villa,penthouse) - кодируется третьей буквой
  4. по [питанию](https://ru.wikipedia.org/wiki/%D0%A4%D0%BE%D1%80%D0%BC%D1%8B_%D0%BF%D0%B8%D1%82%D0%B0%D0%BD%D0%B8%D1%8F_%D0%B2_%D0%B3%D0%BE%D1%81%D1%82%D0%B8%D0%BD%D0%B8%D1%86%D0%B5) (all inclusive,breakfast,half board,full board) - кодируется четвертой буквой

#### images:

| Название  | Тип          | Размер   |
|-----------|--------------|----------|
| id        | int          | 8 байт   |
| invID     | int          | 8 байт   |
| bucketURL | varchar(256) | 256 байт |

#### book:

| Название | Тип  | Размер |
|----------|------|--------|
| id       | int  | 8 байт |
| invID    | int  | 8 байт |
| userID   | int  | 8 байт |

Таким образом получается, что размещения занимают еще 585 байт. 
Примем, что в среднем на каждый вариант размещения в среднем приходится 10 фотографий.
Тогда для получения полной динамической информации по размещению, пользователь запрашивает
`585 + 272 x 10` = 3305 байт.

С учетом этого получаются следующие нагрузочные данные по типам запросов

| Тип                | Допускаемая константа                                      | Кол-во  | Размер(байт) | Cуммарный объем(МБ) | RPS v*(400*10^6)/(30x24x60x60) | Пиковое потребление в течение суток v*(8x400x10^6)/(1024x30x24x60x60)(Гбит/с) |
|--------------------|------------------------------------------------------------|---------|--------------|---------------------|--------------------------------|-------------------------------------------------------------------------------|
| Авторизация        | -                                                          | 2       | 518          | 0,001               | 309                            | 0,0012                                                                        |
| Региональный поиск | 10 - кол-во размещений на странице                         | 1       | 576 * 10     | 0,0055              | 154                            | 0,0066                                                                        |
| Детальный поиск    | -                                                          | 15      | 3305         | 0,047               | 2314                           | 0,0566                                                                        |
| Картинки           | 40кб - средний вес изображения 1024x768, 10 - кол-во фоток | 15      | 40960*10     | 5,8594              | 2314                           | 7,0643                                                                        |
| Бронирование       | -                                                          | 2       | 24           | 0                   | 309                            | 0                                                                             |
| Отмена             | -                                                          | 1       | 24           | 0                   | 154                            | 0                                                                             |

* Суммарное пиковое потребление в сутки: 7,1287 Гбит/с

### Размер хранения в разбивке по типам данных

Наиболее существенная часть - картинки. В мире [насчитывается](https://bolddata.nl/en/database/worldwide-hotel/) 1073500 отелей, 
в каждом из которых допустим насчитывается в среднем 200 вариантов размещения. С учетом ранее принятого значения в 10 картинок на вариант 
размещения получается размер хранилища в 

`1073500 x 200 x 10 x 40кБ /(1024x1024x1024)` = 79,98 ТБ

## 3. Логическая схема <a name="logic"></a>
## 4. Физическая схема <a name="physic"></a>
## 5. Технологии <a name="tech"></a>
## 6. Схема проекта <a name="schema"></a>
## 7. Список серверов <a name="servers"></a>

