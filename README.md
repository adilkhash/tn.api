![TraderNet](https://raw.githubusercontent.com/tradernet/tn.api/master/rsz_logo_tradernet.png "TraderNet")

* [REST API](#RESTAPI)
    * [Формат запроса](#restReqFormat)
    * [Формирование подписи](#restReqSig)
    * [Ответ сервера](#restResponse)
    * [Возможные коды ответа](#restResponseCodes)
    * [Команды](#restCommands)
        * [AddBlogPost](#AddBlogPost) - добавление поста в свой блог
        * [GetBlogsFeed](#GetBlogsFeed) - получение ленты подписок пользователя (в блогах)
        * [getBlogsFeedByClientId](#getBlogsFeedByClientId) - получение ленты подписок клиента (в блогах)
        * [setOrderstat](#setOrderstat) - сохраняет статистику обработки неисполняемых заявок
        * [getProfitList](#getProfitList) - список достижений
        * [getInvestFeedByTicker](#getInvestFeedByTicker) - лента инвестидей по тикеру всех авторов на которых подписан пользователь
        * [getBlogsPostById](#getBlogsPostById) - пост по ID
        * [getBestTraders](#getBestTraders) - рейтинг трэйдеров
        * [getUserInfoById](#getUserInfoById) - данные пользователя по ID
        * Анкета
            * [reg](#reg) - шаг 1. Регистрация пользователя. Создание учётной записи. Получение его userId
            * [regPhone](#regPhone) - шаг 2. Отправка sms кода для подтверждения номера телефона
            * [regPhoneSms](#regPhoneSms) - шаг 3. Подтверждение номера телефона при помощи кода
            * [regTariff](#regTariff) - шаг 4. Выбор тарифа пользователя
            * [regPassport](#regPassport) - шаг 5. Паспортные данные нового пользователя
            * [regDocs](#regDocs) - шаг 6. Регистрационные документы нового пользователя
        * [getQuotesHistory](#getQuotesHistory) - получение истории котировок по бумаге
        * [getSecurityInfo](#getSecurityInfo) - получение информации по бумаге
            
* [Socket.IO API](#SIOAPI)
    * [Введение](#intro)
    * [Рыночные данные](#marketData): 
        * [notifyQuotes](#notifyQuotes) - подписка на котировки
        * [notifyOrderBook](#notifyOrderBook) - подписка на стакан 
        * [notifyMarkets](#notifyMarkets) - подписка на сообщения о рынках
        * [setDelay](#setDelay) - установка минимальной задержки между обновлениями
        * [searchSecurities](#searchSecurities) - поиск ценных бумаг по частичному совпадению тикера
    * [Клиентские данные](#clientData): 
        * [notifySessions](#notifySessions) - подписки на клиентские сессии безопасности
        * [notifyPortfolio](#notifyPortfolio) - подписка на портфель клиента
        * [notifyOrders](#notifyOrders) - подписка на приказы клиента
    * [Торговые приказы](#orders): 
        * [putOrder](#putOrder) - выставление приказа
        * [deleteOrder](#deleteOrder) - отмена приказа
    * [Сессии безопасности](#sessions): 
        * [getSafetyTypes](#getSafetyTypes) - получение списка типов сессий безопасности
        * [getSecuritySessions](#getSecuritySessions) - получение открытых сессий безопасности
        * [initValidation](#initValidation) - инициализация двухэтапного открытия сессии безопасности
        * [openSecuritySession](#openSecuritySession) - открытие сессии безопасности
    * [Токены безопасности](#tokens): 
        * [activateToken](#activateToken) - активация токена
        * [syncToken](#syncToken) - синхронизация токена
        * [tokenInfo](#tokenInfo) - информация о токене

<a name="RESTAPI"></a>
# REST API

API_URL: https://tradernet.ru/api/

API_SECRET: при доступе по userId задаётся в профиле пользователя в разделе "лучшие трейдеры", при доступе по ip задаётся администратором

Формат данных: JSON

Тип HTTP запроса: POST (возможен GET для тестирования)

<a name="restReqFormat"></a>
## Формат запроса 

Запрос представляет из себя JSON со следующими параметрами:

|параметр | описание | обязательный |
|-------|----------------------------| ---------------------------------------------|
|uid    | id пользователя (int)      | да, для не аутентифицированных пользователей |
|cmd    | команда (string)           | да |
|params | параметры команды (object) | да |
|sig    | подпись (string)           | да |

<a name="restReqSig"></a>
### Формирование подписи

Параметр sig равен md5 от конкатенации пар "parameter_name=parameter_value", 
отсортированных в порядке возрастания имени параметра (по алфавиту) 
и добавленного в конец строки секрета API_SECRET. 
Вложенные параметры обрабатываются рекурсивно.

<a name="restResponse"></a>
## Ответ сервера

| параметр | описание                      | обязательный |
| -------- | --------                      | ------------ |
| code     | Код ответа (int)              | да |
| data     | Данные ответа (mixed)         | нет |
| errMsg   | Сообщение об ошибке (string), | необязательный параметр, может взвращаться в случае возникновения ошибки | нет |

<a name="restResponseCodes"></a>
### Возможные коды ответа

| Код | Имя | Описание |
| --- | --- | -------- |
| 0  | ERROR_OK | Всё Ок, ошибок нет |
| 1  | ERROR_INCORRECT_QUERY | Ошибка в запросе (неправильный формат, несуществующая команда и т.д.) |
| 2  | ERROR_BAD_JSON | Неправильный/повреждённый json |
| 3  | ERROR_UNKNOWN_CMD | Несуществующая команда |
| 4  | ERROR_BAD_SIGN | Неправильная подпись |
| 5  | ERROR_CMD_PARAMS | Ошибка в параметрах команды |
| 6  | ERROR_IMG_DOWNLOAD | Ошибка загрузки картинки |
| 7  | ERROR_UNKNOWN_UID | Не определён userId |
| 8  | ERROR_UNKNOWN_IP | Не известный IP |
| 9  | ERROR_ORDERSTAT_INSERT | Вставка в модель orderstat не удалась |
| 10 | ERROR_UNKNOWN_CLIENT_ID | Неизвестный clientId |
| 11 | ERROR_EMPTY_PROFITS | Пустой рейтинг |
| 12 | ERROR_ACCESS_DENIED | Доступ закрыт / ошибка авторизации |
| 13 | ERROR_UPLOAD | Ошибка загрузки файла |
| 14 | ERROR_BAD_CODE | Неверный проверочный код |


<a name="restCommands"></a>
## Команды 

<a name="AddBlogPost"></a>
### AddBlogPost

Добавление поста в свой блог

*Параметры команды:*

| параметр | описание | обязательный |
| -------- | -------- | ------------ |
| title    | Заголовок (string)                         | нет |
| subtitle | Подзаголовок (string)                      | нет |
| tags     | Тэги поста (string), разделитель - запятая | нет |
| text     | Текст поста (string)                       | да |
| images   | Изображения прикрепляемые к посту (array)  | нет |

Images может содержать в себе несколько объектов image:

| параметр    | описание                               | обязательный |
| ----------- | -------------------------------------- | ------------ |
| num         | уникальный номер картинки внутри поста | да |
| url         | url картинки (string)                  | да |
| position    | позиция изображения (string), возможные варианты - left, right, between | нет |
| description | описание картинки | нет |

*Пример:*

```php
<?php
$params = array (
    'title'    => 'title',
    'subtitle' => 'subtitle',
    'tags'     => 'tag1 ,tag2, tag3',
    'text'     => 'Posttext[img_1][img_2]',
    'images' =>
    array (
        array (
            'num'         => 1,
            'url'         => 'http://example.com/sample.jpg',
            'position'    => 'left',
            'description' => 'Imagedescription',
        ),
        array (
            'num'         => 2,
            'url'         => 'http://example.com/sample2.jpg',
            'position'    => 'between',
            'description' => 'Secondimagedescription',
        ),
    ),
);
?>
```

<a name="GetBlogsFeed"></a>
### GetBlogsFeed

Получение ленты подписок пользователя (в блогах)

*Параметры команды*

| параметр | описание | обязательный | 
| -------- | -------- | ------------ | 
| page | запрашиваемая страница | да | 
| ts | время, когда была просмотрена первая страница ленты | для первой страницы - нет, для остальных желателен

*Формат ответа:*

| параметр | описание | обязательный | 
| -------- | -------- | ------------ | 
| userId | пользователь, чью подписку смотрим | да | 
| tsStart | время просмотра первой страницы ленты | да | 
| page | текущая страница | да | 
| posts | массив объектов постов | да | 

*posts item:*

| параметр | описание | обязательный | 
| -------- | -------- | ------------ | 
| id | id поста | да | 
| date_crt | дата создания | да | 
| title | заголовок | да | 
| subtitle | подзаголовок | да | 
| text | текст | да | 
| agg_comments | кол-во комментариев к посту | да | 
| agg_likes | кол-во лайков | да | 
| login | отображаемое имя юзера | да | 
| ilike | лайк текущего юзера | да | 
| type | Тип поста (Пост = 1 или Инвестидея = 2 или Обучение = 3) | нет | 
| param_1 | если тип = 2 - рекомендация (1 = Купить, 2 = Продать) | нет | 
| param_2 | если тип = 2 - ожидаемый процент дохода | нет | 
| avatar | аватрка юзера | да | 
| tags | тэги (object) | нет | 


*Пример ответа сервера:*

```json
{
  "code": 0,
  "blogs_feed": {
    "userId": 10,
    "tsStart": "2012-06-02 23:52:50",
    "page": "1",
    "posts": [
      {
        "id": 240,
        "date_crt": "2012-06-02 23:52:21.830204",
        "title": "title",
        "subtitle": "subtitle",
        "text": "TEXT",
        "agg_comments": 0,
        "agg_likes": null,
        "login": "gti",
        "ilike": null,
        "mainImg": "/data/avatar/10.s.png",
        "type": "1 - блог || 2 - инвестидея",
        "param_1": "Рекомендация (если type = 2 )",
        "param_2": "Доход (если type = 2 )",
        "tags": {
          "1": "Тэг №1",
          "2": "тэг №2"
        }
      },
      {
        "id": 239,
        "date_crt": "2012-06-02 23:51:32.776083",
        "title": "",
        "subtitle": "",
        "text": "TEXTTEXT",
        "agg_comments": 3,
        "agg_likes": 2,
        "login": "gti",
        "ilike": true,
        "mainImg": "/data/avatar/10.s.png"
      }
    ]
  }
}
```

<a name="getBlogsFeedByClientId"></a>
### getBlogsFeedByClientId

Получение ленты подписок клиента (в блогах)


*Параметры команды*

|параметр | описание | обязательный |
| -------- | -------- | ------------ | 
|clientId | ID клиента | да |
|page | запрашиваемая страница | да |
|ts | время, когда была просмотрена первая страница ленты | для первой страницы - нет, для остальных желателен |

*Формат ответа*

|параметр | описание | обязательный |
| -------- | -------- | ------------ | 
|userId | пользователь, чью подписку смотрим | да |
|tsStart |время просмотра первой страницы ленты | да |
|page |текущая страница | да |
|posts |массив объектов постов |да |

*posts item*

|параметр |описание |обязательный |
| -------- | -------- | ------------ | 
|id | id поста | да |
|date_crt |дата создания | да |
|title | заголовок | да |
|subtitle | подзаголовок | да |
|text | текст | да |
|agg_comments | кол-во комментариев к посту | да |
|agg_likes | кол-во лайков | да |
|login |отображаемое имя юзера |да|
|ilike |лайк текущего юзера |да|
|type |Тип поста (Пост = 1 или Инвестидея = 2) | нет|
|param_1 |если тип = 2 - рекомендация (1 = Купить, 2 = Продать) | нет |
|param_2 |если тип = 2 - ожидаемый процент дохода | нет |
|avatar |аватрка юзера |да|
|tags |тэги (object) |нет|

*Пример ответа сервера*

```json
{
  "code": 0,
  "blogs_feed": {
    "userId": 10,
    "tsStart": "2012-06-02 23:52:50",
    "page": "1",
    "posts": [
      {
        "id": 240,
        "date_crt": "2012-06-02 23:52:21.830204",
        "title": "title",
        "subtitle": "subtitle",
        "text": "TEXT",
        "agg_comments": 0,
        "agg_likes": null,
        "login": "gti",
        "ilike": null,
        "mainImg": "/data/avatar/10.s.png",
        "type": "1 - блог || 2 - инвестидея || 3 - обучение",
        "param_1": "Рекомендация (если type = 2 )",
        "param_2": "Доход (если type = 2 )",
        "tags": {
          "1": "Тэг №1",
          "2": "тэг №2"
        }
      },
      {
        "id": 239,
        "date_crt": "2012-06-02 23:51:32.776083",
        "title": "",
        "subtitle": "",
        "text": "TEXTTEXT",
        "agg_comments": 3,
        "agg_likes": 2,
        "login": "gti",
        "ilike": true,
        "mainImg": "/data/avatar/10.s.png"
      }
    ]
  }
}
```

<a name="setOrderstat"></a>
### setOrderstat

Сохраняет статистику обработки неисполняемых заявок, 
а в случае если: нет соединения, заявки не выставляются и т.п.; 
статистика высылается на настраиваемые почтовые ящики.

*Описание команды*

Полученные на входе параметры записываются в бд (orderstat), если нет соединения, то высылаются на настраиваемые почтовые ящики (mail.conf.php::developer_emails). Доступ осуществляется по IP (project.conf.php::accessIPtoAPI).
Для передачи параметров используются методы GET или POST (POST предпочтительнее), формат параметров - массив JSON


*Параметры команды*

| параметр | описание | обязательный |тип|
| -------- | -------- | ------------ | ---- |
|broker_id |id брокера |да |INT4|
|order_type |тип заявки (выставление неисполняемой заявки, снятие неисполняемой заявки) |да |INT2|
|instanse_id |идентификатор робота |да |INT4|
|connection_type |тип соединения |да |INT2|
|time_begin |время отправки заявки (локальное клиентское время) |да |TIMESTAMP|
|time_end |время получения заявки (локальное клиентское время) |да |TIMESTAMP|
|time_roundtrip |(time_end – time_begin) |да |INT4|

*Пример запроса*

```php
{
  "cmd":"setOrderstat",
  "params":{
      "broker_id":"767",
      "instanse_id":"1234",
      "order_type":"1",
      "connection_type":"1",
      "time_begin":"2012-07-02 15:29:00.666",
      "time_end":"2012-07-02 15:29:00.777",
      "time_roundtrip":"7676" 
  }
}
```

*Примеры ответа сервера*

Успешное завершение:

```json
{ "code": 0}
```

Ошибка в параметрах:

```json
{
    "code": 5,
    "errMsg": "Invalid params for Orderstat"
}
```

Ошибка авторизации:

```php
{
    "code": 8,
    "errMsg": "Unkonown IP"
}
```

Ошибка вставки:

```php
{
    "code": 9,
    "errMsg": "Error Insertion"
}
```

<a name="getProfitList"></a>
### getProfitList

Список достижений

*Формат ответа*

|параметр |описание |обязательный|
| -------- | -------- | ------------ | 
|portfolioId|  |да|
|profit | |да|
|income | |да|
|profileUrl |Ссылка на портфель |да|
|login |Логин владельца портфеля |да|
|score | |да|

*Пример ответа сервера*

```json
{
  "profit_list": {
    "2337": {
      "portfolioId": 2337,
      "profit": "1.016255",
      "income": "1727.520000",
      "profileUrl": "https://sb-ek.testdev/portfolios/view/id/2337",
      "login": "KGQ",
      "score": 0
    },
    "2713": {
      "portfolioId": 2713,
      "profit": "1.005844",
      "income": "540.460000",
      "profileUrl": "https://sb-ek.testdev/portfolios/view/id/2713",
      "login": "9LH",
      "score": 0
    },
    "4125": {
      "portfolioId": 4125,
      "profit": "1.002844",
      "income": "1592.340000",
      "profileUrl": "https://sb-ek.testdev/portfolios/view/id/4125",
      "login": "CMR",
      "score": 0
    }
  },
  "code": 0
}
```

<a name="getInvestFeedByTicker"></a>
### getInvestFeedByTicker

Лента инвестидей по тикеру всех авторов на которых подписан пользователь

*Параметры команды*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| clientId | ID клиента, | да |
| ticker | Тикер | да |
| page | запрашиваемая страница, | да |
| ts | время, когда была просмотрена первая страница ленты | для первой страницы - нет, для остальных желателен |

*Формат ответа*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| userId | пользователь, чью подписку смотрим, | да |
| tsStart | время просмотра первой страницы ленты. | да |
| page | текущая страница | да |
| posts | массив объектов постов | да |

*posts item*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| id | id поста | да |
| date_crt | дата создания | да |
| title | заголовок | да |
| subtitle | подзаголовок | да |
| text | текст | да |
| agg_comments | кол-во комментариев к посту | да |
| agg_likes | кол-во лайков | да |
| login | отображаемое имя юзера | да |
| ilike | лайк текущего юзера | да |
| type | Тип поста (Пост = 1 или Инвестидея = 2) | нет |
| param_1 | если тип = 2 - рекомендация (1 = Купить, 2 = Продать) | нет |
| param_2 | если тип = 2 - ожидаемый процент дохода | нет |
| avatar | аватрка юзера | да |
| tags | тэги (object) | нет |

*Пример ответа сервера*

```json
{
  "code": 0,
  "blogs_feed": {
    "userId": 10,
    "tsStart": "2012-06-02 23:52:50",
    "page": "1",
    "posts": [
      {
        "id": 240,
        "date_crt": "2012-06-02 23:52:21.830204",
        "title": "title",
        "subtitle": "subtitle",
        "text": "TEXT",
        "agg_comments": 0,
        "agg_likes": null,
        "login": "gti",
        "ilike": null,
        "mainImg": "/data/avatar/10.s.png",
        "type": "1 - блог || 2 - инвестидея || 3 - обучение",
        "param_1": "Рекомендация (если type = 2 )",
        "param_2": "Доход (если type = 2 )",
        "tags": {
          "1": "Тэг №1",
          "2": "тэг №2"
        }
      },
      {
        "id": 239,
        "date_crt": "2012-06-02 23:51:32.776083",
        "title": "",
        "subtitle": "",
        "text": "TEXTTEXT",
        "agg_comments": 3,
        "agg_likes": 2,
        "login": "gti",
        "ilike": true,
        "mainImg": "/data/avatar/10.s.png"
      }
    ]
  }
}
```

<a name="getBlogsPostById"></a>
### getBlogsPostById

Пост по ID

*Параметры команды*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| postId | ID поста | да |

*Формат ответа*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| userId | пользователь, чью подписку смотрим, | да |
| viewerId | пользователь, который смотрит, | да |
| tsStart | время просмотра первой страницы ленты. | да |
| page | текущая страница | да |
| posts | объектов пост | да |

*blog_post item*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| id | id поста | да |
| date_crt | дата создания | да |
| title | заголовок | да |
| subtitle | подзаголовок | да |
| text | текст | да |
| agg_comments | кол-во комментариев к посту | да |
| agg_likes | кол-во лайков | да |
| login | отображаемое имя юзера | да |
| ilike | лайк текущего юзера | да |
| type | Тип поста (Пост = 1 или Инвестидея = 2 или Обучение = 3) | нет |
| param_1 | если тип = 2 - рекомендация (1 = Купить, 2 = Продать) | нет |
| param_2 | если тип = 2 - ожидаемый процент дохода | нет |
| avatar | аватрка юзера | да |
| tags | тэги (object) | нет |

*Пример ответа сервера*

```json
{
  "blog_post": {
    "userId": null,
    "viewerId": 0,
    "tsStart": "2012-10-25 12:28:46",
    "posts": [
      {
        "id": 749,
        "user_id": 339110,
        "date_crt": "2012-10-12 14:11:12.841533",
        "date_mod": "2012-10-12 14:11:12.841533",
        "title": "тест",
        "subtitle": "Подзаголовок",
        "text": "<p>[img_1]обучаемся</p>",
        "agg_comments": 0,
        "agg_likes": null,
        "agg_likers": null,
        "type": 3,
        "security_id": null,
        "param_1": null,
        "param_2": null,
        "approve": 1,
        "liker_1": null,
        "liker_2": null,
        "login": "ztalker",
        "ilike": null,
        "avatar": "/i/_tradersClubUserPicture.png",
        "postDate": "12 октября в 14:11",
        "images": [
          {
            "id": 279,
            "num": 1,
            "post_id": 749,
            "position": 1,
            "description": ""
          }
        ]
      }
    ],
    "page": 1,
    "moder": true
  },
  "code": 0
}
```

<a name="getBestTraders"></a>
### getBestTraders

Рейтинг трэйдеров

*Параметры команды*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| type | тип акаунта boolean or null | нет |
| viewMode | сколько записей в рейтинге[1-10] or null | нет |
| period | Период на котором строится рейтинг[D,W,M,Y] or null | нет |

*Формат ответа*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| code | Код статуса исполнения | да |
| profits | ассоциативный массив лучших трейдеров. В качестве ключей используются id портфелей | нет |

*profits item*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| rating | Рейтинг | да |
| profit |  | да |
| income |  | да |
| user_id |  | да |
| countMonth | кол-во сделок за мес. | да |
| isDemo | тип портфеля | да |
| name | имя текущего юзера | да |
| login | логин текущего юзера | да |
| score |  | нет |

*Пример ответа сервера*

```json
{
  "profits": {
    "21198": {
      "rating": 5,
      "profit": 1254.0858,
      "income": null,
      "user_id": 56423,
      "countMonth": 1,
      "isDemo": 1,
      "name": "TDGJ",
      "login": "TDGJ",
      "score": 25
    },
    "35807": {
      "rating": 2,
      "profit": 10233.3143,
      "income": "102280090.780000",
      "user_id": 101262,
      "countMonth": 1,
      "isDemo": 1,
      "name": "3G2K",
      "login": "Target2006",
      "score": 145
    }
  },
  "code": 0
}
```

<a name="getUserInfoById"></a>
### getUserInfoById

Данные пользователя по ID

*Параметры команды*

параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| user_id | ID Пользователя | да |

*Формат ответа*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| user_data | объект юзер | нет |
| code | Код статуса исполнения | да |

*user_data*

параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| id | ID юзера | да |
| group_id |  | да |
| login |  | да |
| pwd |  | да |
| lastname |  | нет |
| firstname |  | нет |
| middlename |  | нет |
| email |  | нет |
| mod_tmstmp |  | да |
| rec_tmstmp |  | да |
| last_visit_tmstmp |  | да |
| f_active |  | да |
| md5 |  | да |
| trader_systems_id |  | нет |
| client_id |  | да |
| f_demo |  | да |
| birthday |  | нет |
| sex |  | нет |
| citizenship |  | да |
| status |  | да |
| type |  | да |
| umod_tmstmp |  | да |
| status_id |  | да |
| date_tsmod |  | да |
| utm_campaign |  | да |
| role_name |  | да |
| role |  | да |

*Пример ответа сервера*

```json
{
  "user_data": {
    "id": 10,
    "group_id": 2,
    "login": "gti",
    "pwd": "wrew43ew4w3e3435wser",
    "lastname": "Вованов",
    "firstname": "Вован",
    "middlename": "Вованович",
    "email": null,
    "mod_tmstmp": "2012-10-24 07:39:31",
    "rec_tmstmp": "2010-04-01 13:02:22",
    "last_visit_tmstmp": null,
    "f_active": 1,
    "md5": null,
    "trader_systems_id": null,
    "client_id": 343434,
    "f_demo": 0,
    "birthday": null,
    "sex": null,
    "citizenship": null,
    "status": null,
    "type": null,
    "umod_tmstmp": null,
    "status_id": null,
    "date_tsmod": null,
    "utm_campaign": null,
    "role_name": "user",
    "role": 2
  },
  "code": 0
}
```

<a name="reg"></a>
### reg

Анкета: Шаг 1. Регистрация пользователя. Создание учётной записи. Получение его userId.

*Параметры команды*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| login | логин пользователя | да |
| password | пароль пользователя | да |
| utm_campaign | APIv1 | да |

В ответ возвращается идентификатор созданного пользователя - userId.

*Пример запроса*

```php
<?php
$params = array (
    'login'    => 'testlogin@testdomain.test',
    'password' => 'pasSwD73245',
    'utm_campaign'     => 'APIv1',
);
?>
```

<a name="regPhone"></a>
### regPhone

Анкета: Шаг 2. Отправка sms кода для подтверждения номера телефона

*Параметры команды*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| userId | id пользователя (возвращается при вызове команды reg) | да |
| phone | номер телефона в формате +7xxxzzzzzzz | да |

При вызове данной команды пользователю отправляется sms сообщение с проверочным кодом. Код считается действительным в течение 30 минут.

В ответ возвращается phoneId.

<a name="regPhoneSms"></a>
### regPhoneSms

Анкета: Шаг 3. Подтверждение номера телефона при помощи кода

*Параметры команды*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| userId | id пользователя (возвращается при вызове команды reg) | да |
| phone | номер телефона в формате +7xxxzzzzzzz | да |
| phoneId | id телефона (возвращается при вызове regPhone) | да |
| code | Код полученный пользователем в sms сообщении | да |

В случае удачной проверки возвращается isCodeCorrect=true

<a name="regTariff"></a>
### regTariff

Анкета: Шаг 4. Выбор тарифа пользователя

*Параметры команды*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| userId | id пользователя (возвращается при вызове команды reg) | да |
| tariff | выбранный пользователем тариф, например tariff_investor, tariff_allInclusive или tariff_personalBroker | да |
| phoneId | id телефона (из regPhone) | да |
| code | Проверочный код (из regPhone) | да |

На данный момент можно указать только три тарифа:

* tariff_investor
* tariff_allInclusive
* tariff_personalBroker

<a name="regPassport"></a>
### regPassport

Анкета: Шаг 5. Паспортные данные нового пользователя

*Параметры команды*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| userId | id пользователя (возвращается при вызове команды reg) | да |
| anketa | массив с данными анкеты | да |
| phoneId | id телефона (из regPhone) | да |
| code | Проверочный код (sms из regPhone) | да |

*Массив с данными анкеты*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| country | Гражданство пользователя (Россия, Украина и т.д.) | да |
| fname | Фамилия | да |
| iname | Имя | да |
| oname | Отчество | да |
| bday | Дата рождения - число | да |
| bmonth | Дата рождения - месяц | да |
| byear | Дата рождения - год | да |
| passport_ser | Серия паспорта | да |
| passport_number | Номер паспорта | да |

<a name="regDocs"></a>
### regDocs

Анкета: Шаг 6. Регистрационные документы нового пользователя

*Параметры команды*

| параметр | описание | обязательный |
| -------- | -------- | ------------ | 
| userId | id пользователя (возвращается при вызове команды reg) | да |
| filesData | массив "имя файла" => "тип файла" | да |
| phoneId | id телефона (из regPhone) | да |
| code | Проверочный код (sms из regPhone) | да |

Сами файлы отправляются стандартным способом.

В ответ возвращается code=0, в случае удачной загрузки, либо код ошибки.

<a name="getQuotesHistory"></a>
## getQuotesHistory

Получение истории котировок по бумаге

```javascript
var json = {         
   'cmd': 'getQuotesHistory',
   'params': 
   {
       'ticker': 'SBER',
       'interval': 60, // '1', '5', '10', '15', '30', '60', 
       'from': '2015-01-01’,
       'to': '2015-02-01' //если параметр опущен, будет взята текущая дата
   }
}

$.ajax({
    url: 'https://tradernet.ru/api/',
    type: 'POST',
    data: {q: json},
    success: function (response) {
        console.log(response);
    }
});

```


<a name="getSecurityInfo"></a>
## getSecurityInfo

Получение информации по бумаге

```javascript

var json = {         
   'cmd': 'getSecurityInfo',
   'params': 
   {
       'ticker': 'SBER',
       'sup': true // если опустить этот ключ, придет более подробная инфрмация из торговой системы 
   }
}

$.ajax({
    url: 'https://tradernet.ru/api/',
    type: 'POST',
    data: {q: json},
    success: function (response) {
        console.log(response);
    }
});

```

Получение информации по бумаге


<a name="SIOAPI"></a>
# Socket.IO API

<a name="intro"></a>
## Введение

### Подключение к Socket.IO:

Для работы с сервисом нужно подключить код библиотеки *socket.io* на странице клиента:
```html
<script src="https://wsbeta.tradernet.ru/socket.io/socket.io.js"></script>
```
И создать объект socket.io:
```javascript
var ws = io('https://wsbeta.tradernet.ru');
```
В настоящий момент возможна работа с одним из двух серверов - боевым или демо:
 
 * **Боевой**: https://ws.tradernet.ru
 * **Демо**: https://wsbeta.tradernet.ru
 
Для работы с боевым сервером необходима авторизация на сайте https://tradernet.ru, 
для работы с демо сервером - на сайте https://beta.tradernet.ru 

### Подключение из Node.js (без браузера)

Для подключения к API из Node.js можно использовать библиотеку socket.io-client:

```javascript
var io = require('socket.io-client');
var ws = io('https://wsbeta.tradernet.ru', {
    transports: ['websocket']
});

ws.on('connect', function () {
    console.log('socket.io-client connected');
});
```

В отличие от браузерной авторизации, в Node.js можно авторизоваться по ключу.

Это позволяет избежать необходимости открытия сессии безопасности обычным способом (по токену или по SMS).

Чтобы авторизоваться в системе по ключу нужно:

1. Сгенерировать публичный и секретный ключи на странице профиля.
2. Создать объект авторизации.
3. С помощью секретного ключа сгенерировать подпись.
4. Открыть WebSocket-соединение.
5. Отправить запрос авторизации через открытое в предыдущем пункте WebSocket-соединение.

*Пример авторизации:*

```javascript
var io = require('socket.io-client');
var tncrypto = require('./tn-crypto');

var pubKey = '*** PUBLIC KEY ***';
var secKey = '*** SECRET KEY ***';

var ws = io('https://wsbeta.tradernet.ru', {
    transports: ['websocket']
});

ws.on('connect', function () {
    console.log('WS connect');
    auth(ws, pubKey, secKey, function (err, auth) {
        if (err) return console.error('Ошибка авторизации', err);
        console.log('login:', auth.login);
        console.log('mode:', auth.mode);
        if (auth.trade)
            console.log('Приказы подавать можно');
        else
            console.log('Приказы подавать нельзя');
    });
});

function auth(ws, pubKey, secKey, cb) {
    var data = {
        apiKey: pubKey,
        cmd: 'getAuthInfo',
        nonce: Date.now()
    };
    var sig = tncrypto.sign(data, secKey);
    ws.emit('auth', data, sig, cb);
}
```

Здесь использован модуль шифрования tn-crypto.js, который находится в папке examples/


### Запуск примеров на JSFIDDLE

Примеры в этой статье можно запускать в *JSFIDDLE*. 
Для этого нужно предварительно авторизоваться на сайте https://beta.tradernet.ru.

**Внимание! Не используйте для запуска примеров боевой логин.** 
Если Вы авторизованы как клиент, перед запуском примеров завершите свой сеанс и войдите с логином демо-пользователя.

Чтобы открыть пример в JSFIDDLE, достаточно перейти по ссылке под кодом примера:
```javascript
var ws = io('https://wsbeta.tradernet.ru');

ws.on('connect', function () {
    console.log('socket.io connected');
});
```
[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/bfsjpgvs/)

<a name="marketData"></a>
## Рыночные данные

<a name="notifyQuotes"></a>
### notifyQuotes

Подписка на котировки

Осуществляется отправкой на сервер сообщения **notifyQuotes** со списком тикеров. Котировки приходят в сообщениях с именем **q**. 
В первый раз котировка приходит полностью. Потом приходят изменения.

*Синонимы: sup_updateSecurities2.* 

```javascript
var ws = io('https://wsbeta.tradernet.ru');
    
ws.on('q', function (data) {
   console.log(data);
});
    
ws.emit('notifyQuotes', ['SBER', 'LKOH', 'GMKN', 'ROSN', 'GAZPM']);
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/asbx8bno/)
 
*Пример сообщения:*
 
```json 
 {
   "q": [
     {
       "n": 0,
       "c": "SBER",
       "mrg": "M",
       "ltr": "MCX",
       "kind": 1,
       "type": 1,
       "name": "Сбербанк",
       "name2": "Sberbank",
       "bbp": 73.56,
       "bbc": " ",
       "bbs": 12000,
       "bbf": 0,
       "bap": 73.57,
       "bac": " ",
       "bas": 100,
       "baf": 0,
       "pp": 74,
       "op": 74,
       "ltp": 73.565,
       "lts": 140,
       "ltt": "2014-10-29T02:23:59",
       "chg": -0.435,
       "pcp": -0.59,
       "ltc": " ",
       "mintp": 73.39,
       "maxtp": 74.17,
       "vol": 110940760,
       "vlt": 1644150.795,
       "yld": 0,
       "acd": 0,
       "fv": 0,
       "mtd": "",
       "cpn": 0,
       "cpp": 0,
       "ncd": "",
       "ncp": 0,
       "dpb": 0,
       "dps": 0,
       "trades": 0,
       "min_step": 0.01,
       "step_price": 0,
       "p5": 79.935,
       "chg5": -7.97,
       "p22": 77.475,
       "chg22": -5.05,
       "p110": 82,
       "chg110": -10.29,
       "p220": 97.08,
       "chg220": -24.22,
       "x_dsc1": "18.0",
       "x_dsc2": "23.0",
       "x_dsc3": "30.0",
       "x_descr": "Сберегательный банк России...",
       "x_curr": "RUR",
       "x_short": "1",
       "x_lot": "10",
       "x_currVal": "1.00000000",
       "x_min": "20",
       "x_max": "100",
       "x_istrade": "1"
     }
   ]
 }
```

*Поля:*

| Поле | Описание |
| ------- |-------- | 
| n | Порядковый номер котировки. Каждый тикер имеет свою нумерацию | 
| c | Тикер | 
| mrg | Признак маржинальности. Если бумага маржинальна, содержит строку 'M'                 | 
| ltr | Биржа последней сделки |
| kind | Вид бумаги (1 - Common Обыкновенные, 2 - Pref Привилегированные, 3 - Percent Процентные, 4 - Discount Дисконтные, 5 - Delivery Поставочный, 6 - Rated Расчетный, 7 - Interval Интервальный) |
| type | Тип бумаги (1 - акции, 2 - облигации, 3 - фьючерсы) |
| name | Название бумаги |
| name2 | Латинское название бумаги |
| bbp | Лучший бид |
| bbc | Обозначение изменения лучшего бида ('' - не изменился, 'D' - вниз, 'U' - вверх) |
| bbs | Количество (сайз) лучшего бида |
| bbf | Объем(?) лучшего бида |
| bap | Лучший аск |
| bac | Обозначение изменения лучшего аска ('' - не изменился, 'D' - вниз, 'U' - вверх) |
| bas | Количество (сайз) лучшего аска |
| baf | Объем(?) лучшего аска |
| pp | Цена предыдущего закрытия |
| op | Цена открытия в текущей торговой сессии |
| ltp | Цена последней сделки |
| lts | Количество (сайз) последней сделки |
| ltt | Время последней сделки |
| chg | Изменение цены последней сделки в пунктах относительно цены закрытия предыдущей торговой сессии |
| pcp | Изменение в процентах относительно цены закрытия предыдущей торговой сессии |
| ltc | Обозначение изменения цены последней сделки ('' - не изменилась, 'D' - вниз, 'U' - вверх) |
| mintp | Минимальная цена сделки за день |
| maxtp | Максимальная цена сделки за день |
| vol | Объём торгов за день в штуках |
| vlt | Объём торгов за день в валюте |
| yld | Доходность к погашению (для облигаций) |
| acd | Накопленный купонный доход (НКД) |
| fv | Номинал |
| mtd | Дата погашения |
| cpn | Купон в валюте |
| cpp | Купонный период (в днях) |
| ncd | Дата следующего купона |
| ncp | Дата последнего купона |
| dpd | ГО покупки |
| dps | ГО продажи |
| trades | Количество сделок |
| min_step | Минимальный шаг цены |
| step_price | Шаг цены |
| p5 | Цена 5 дней назад |
| chg5 | Изменение цены за 5 дней |
| p22 | Цена 22 дня назад |
| chg22 | Изменение цены за 22 дня |
| p110 | Цена 110 дней назад |
| chg110 | Изменение цены за 110 дней |
| p220 | Цена 220 дней назад |
| chg220 | Изменение цены за 220 дней |
| x_dsc1 | Дисконт |
| x_dsc2 | *не используется* |
| x_dsc3 | *не используется* |
| x_descr | Описание |
| x_curr | Валюта |
| x_short | Можно ли шортить бумагу |
| x_lot | Минимальный лот |
| x_currVal | Курс валюты по отношению к рублю |
| x_min | Минимум за (период) |
| x_max | Максимум за (период) |
| x_istrade | Были ли по бумаге сделки |
 
<a name="notifyOrderBook"></a>
### notifyOrderBook

Подписка на стакан

Осуществляется отправкой на сервер сообщения **notifyOrderBook** со списком тикеров.
Стакан приходит в сообщениях с именем **b**.
В первый раз стакан приходит полностью. Потом приходят изменения.

```javascript
var ws = io('https://wsbeta.tradernet.ru');
    
ws.on('b', function (orderBook) {
    console.log(orderBook);
});
    
ws.emit('notifyOrderBook', ['SBER']);
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/zenzwvbk/)

*Пример сообщения:*

```json
{
  "dom": [{
    "n": 0,
    "i": "SBER",
    "min_step": 0.01,
    "step_price": 0,
    "del": [],
    "ins": [{
      "p": 73.42,
      "s": "S",
      "q": 182000,
      "k": 0
    },
     
     ...
     
    {
      "p": 73.22,
      "s": "B",
      "q": 87700,
      "k": 19
    }],
    "upd": [],
    "cnt": 20,
    "x": 10
  }]
}
```

Каждое сообщение стакана содержит такие поля:

| Поле | Описание |
| ------- |-------- |
| n | Порядковый номер. Каждый тикер имеет свою нумерацию |
| i | Тикер |
| min_step | Минимальный шаг цены |
| step_price |  |
| del | Массив удаляемых позиций |
| ins | Массив вставляемых позиций |
| upd | Массив обновляемых позиций |
| cnt | Глубина стакана |
| x | ... |

Каждый массив удаляемых, вставляемых или обновляемых позиций содержит следующие поля:

| Поле | Описание |
| ------- |-------- |
| p | Цена |
| s | Признак *bid* или *sell*, который может принимать значения **"B"** или **"S"** соответственно |
| q | Количество |
| k | Номер позиции в стакане |

При получении сообщений стакана обновление позиций следует производить так:
 1. Удалить позиции, указанные в массиве *del*
 2. Вставить позиции, указанные в массиве *ins*
 3. Обновить позиции, указанные в массиве *upd*

<a name="notifyMarkets"></a>
### notifyMarkets

Подписка на сообщения о рынках

Осуществляется отправкой сообщения **notifyMarkets**.

Сообщения о рынках имеют метку **markets**.

```javascript
var ws = io('https://wsbeta.tradernet.ru');

ws.on('markets', function (markets) {
    console.log(markets);
});

ws.emit('notifyMarkets');
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/v4392o3c/)

*Пример сообщения:*

```json
[
  {
    "markets": {
      "t": "11/5/2014 3:28:30 PM",
      "m": [
        {
          "n": "ММВБ_АКЦ_Ф",
          "n2": "MCX",
          "s": "OPEN",
          "o": "19:00:00",
          "c": "18:40:00",
          "dt": "0",
          "ev": [
            {
              "id": "StartGate",
              "t": "19:00:00",
              "next": "2014-11-05T19:00:00"
            },
            {
              "id": "MarketOpen",
              "t": "19:00:00",
              "next": "2014-11-05T19:00:00"
            },
            {
              "id": "ResetMIS",
              "t": "00:00:00",
              "next": "2014-11-06T00:00:00"
            },
            {
              "id": "MarketClose",
              "t": "18:40:00",
              "next": "2014-11-05T18:40:00"
            },
            {
              "id": "StopGate",
              "t": "18:40:00",
              "next": "2014-11-05T18:40:00"
            }
          ]
        },
        
        ...
        
      ]
    }
  }
] 
```

Каждое сообщение о рынке содержит такие поля:

| Поле | Описание |
| ------- |-------- |
| n | Название рынка |
| n2 | Альтернативное название рынка |
| s | Состояние рынка ( *OPEN* или *CLOSE* ) |
| o | Время открытия рынка |
| c | Время закрытия рынка |
| dt | Сдвиг в минутах относительно Москвы |

<a name="setDelay"></a>
### setDelay

Установка минимальной задержки между обновлениями котировок и стакана.
Принимает интервал обновления в миллисекундах.

```javascript
var ws = io('https://wsbeta.tradernet.ru');
ws.emit('setDelay', 1000);
```

<a name="searchSecurities"></a>
### searchSecurities

Поиск ценных бумаг по частичному совпадению тикера

```javascript
var ws = io('https://wsbeta.tradernet.ru');

ws.emit('searchSecurities', 'SBER', function (err, res) {
    console.log(err, res);
});
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/b1rcseL0/)

*Пример сообщения:*

```json
[
  {
    "sid": 16394,
    "n": "Сбербанк России",
    "t": "SBER",
    "isin": "RU0009029540",
    "type": "ао",
    "tp": "ao",
    "tn": "Сбербанк России SBER",
    "exkey": "MICEX",
    "exname": "ММВБ",
    "nt_ticker": "SBER"
  },
  {
    "sid": 16395,
    "n": "Сбербанк-п",
    "t": "SBERP",
    "isin": "RU0009029557",
    "type": "ап",
    "tp": "ap",
    "tn": "Сбербанк-п SBERP",
    "exkey": "MICEX",
    "exname": "ММВБ",
    "nt_ticker": "SBERP"
  }
]
```



<a name="clientData"></a>
## Клиентские данные

<a name="notifySessions"></a>
### notifySessions

Подписка на сессии безопасности клиента

Осуществляется отправкой сообщения **notifySessions**.

Сообщения о рынках имеют метку **sessions**.

```javascript
var ws = io('https://wsbeta.tradernet.ru');

ws.on('sessions', function (sessions) {
    console.log(sessions);
});

ws.emit('notifySessions');
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/k361p8g6/)

*Поля сообщения*

| id | Идентификатор сессии |
| --- | -------------------- |
| owner_login | Логин пользователя, от имени которого открыта сессия |
| user_login | Логин авторизованного пользователя |
| safety_type_id | Тип сессии безопасности |
| start_datetime | Дата открытия сессии безопасности |
| expire_datetime | Дата истечения сессии безопасности |
| expire | Время, оставшееся до истечения сессии безопасности, мс |

*Пример сообщения:*

```json
[
  {
    "response": {
      "res": [
        {
          "id": 6765662,
          "owner_login": "OWNER_LOGIN",
          "user_login": "USER_LOGIN",
          "safety_type_id": 4,
          "key_current": "Логин, пароль",
          "start_datetime": "2014-11-11T13:22:06.247",
          "expire_datetime": "2014-11-12T13:22:06.247",
          "expire": 37790
        }
      ]
    }
  }
]
```

<a name="notifyPortfolio"></a>
### notifyPortfolio

Подписка на портфель клиента

Осуществляется отправкой сообщения **notifyPortfolio**.

Сообщения о рынках имеют метку **portfolio**.


```javascript
var ws = io('https://wsbeta.tradernet.ru');

ws.on('portfolio', function (portfolio) {
    console.log(portfolio);
});

ws.emit('notifyPortfolio');
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/1j2rbr6x/)

*Пример сообщения:*

```json
[
  {
    "ps": {
      "key": "%YOUR_LOGIN",
      "acc": [
        {
          "s": 422453.751611,
          "forecast_in": 85000,
          "forecast_out": 12400,
          "curr": "RUR",
          "currval": 1
        }
      ],
      "pos": [
        {
          "i": "LKOH",
          "t": 1,
          "k": 1,
          "s": 22226.9,
          "q": 11,
          "fv": "0",
          "curr": "RUR",
          "currval": 1,
          "name": "ЛУКОЙЛ",
          "name2": "LUKOIL",
          "open_bal": 22226.9,
          "mkt_price": 2130.1,
          "vm": "0",
          "go": 22226.9,
          "profit_close": 1006.2,
          "acc_pos_id": 36054354,
          "trade": [
            {
              "trade_count": 11
            }
          ]
        }
      ]
    }
  }
]
```

*Поля сообщения:*

| Поле | Описание |
| ------- |-------- |
| key | Ключ сообщений портфеля (логин, предварённый знаком процента) |
| acc | Массив счётов клиента |
| pos | Массив позиций клиента |

*Поля счёта:*

| Поле | Описание |
| ------- |-------- |
| s | Свободные средства |
| forecast_in |  |
| forecast_out |  |
| curr | Валюта счёта |
| currval | Курс валюты счёта |

*Поля позиции:*

| Поле | Описание |
| ------- |-------- |
| i | Тикер бумаги |
| t | Тип бумаги |
| k | Вид бумаги |
| s | Стоимость |
| q | Количество |
| curr | Валюта |
| currval | Курс валюты |
| name | Наименование бумаги |
| name2 | Альтернативное наименование бумаги |
| open_bal | Цена открытия |
| mkt_price | Рыночная цена |
| vm |  |
| go |  |
| profit_close |  |
| acc_pos_id |  |
| trade |  |
| trade[].trade_count |  |

<a name="notifyOrders"></a>
### notifyOrders

Подписка на приказы клиента

```javascript
var ws = io('https://wsbeta.tradernet.ru');

ws.on('orders', function (orders) {
    console.log(orders);
});

ws.emit('notifyOrders');
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/huptqc48/)

*Пример сообщения:*

```json
[
  {
    "orders": {
      "key": "%BOYTSOV.MAXIM@GMAIL.COM",
      "order": [
        {
          "id": 6878243,
          "date": "2014-10-14T11:30:17.010",
          "stat": 10,
          "stat_orig": 10,
          "stat_d": "2014-10-14T11:30:17.023",
          "instr": "GAZPM",
          "oper": 3,
          "type": 5,
          "cur": "RUR",
          "p": 135.35,
          "stop": 125,
          "q": 680,
          "aon": "0",
          "exp": 3,
          "rep": "0",
          "fv": "0",
          "name": "ГАЗПРОМ ао",
          "name2": "Gazprom",
          "stat_prev": 1,
          "userOrderId": "0"
        }
      ]
    }
  }
]
```

<a name="orders"></a>
## Торговые приказы

<a name="putOrder"></a>
### putOrder

Выставление приказа

*Параметры*

| Название | Описание |
| ---- | ---- |
| instr_name | Код ценной бумаги (тикер), в отношении которой подаётся приказ |
| action_id | Тип операции. Принимает следующие значения:<br />1 – Покупка (Buy)<br />2 – Покупка при совершении сделок с маржой (Buy on Margin)<br />3 – Продажа (Sell)<br />4 – Продажа при совершении сделок с маржой (Sell Short) |
| order_type_id | Тип приказа. Принимает следующие значения:<br />1 – Рыночный Приказ (Market)<br />2 – Приказ по заданной цене (Limit)<br />3 – Рыночный Стоп-приказ (Stop)<br />4 – Стоп-приказ по заданной цене (Stop Limit) |
| curr | Код валюты приказа по ISO |
| limit_price | Цена приказа |
| stop_price | Стоп-цена приказа |
| qty | Количество ценных бумаг (в штуках) |
| aon | Признак «всё или ничего». Принимает следующие значения:<br />0 – с возможностью частичного исполнения<br />1 – с невозможностью частичного исполнения |
| expiration_id | Срок действия приказа. Принимает следующие значения:<br />1 – Приказ «до конца текущей торговой сессии» (Day)<br />2 – Приказ «день/ночь или ночь/день» (Day + Ext)<br />3 – Приказ «до отмены» (GTC, до отмены с участием в ночных сессиях) |
| submit_ch_c | Канал приёма сообщения:<br />1 – В электронной форме через сайт брокера в сети<br />2 – В устной форме<br />3 – В письменной форме |
| message_id | Всегда передаётся 0 |
| replace_order_id | При создании приказа значение поля не указывается.<br />При формировании поручения на изменение приказа данное поле содержит ID приказа, в отношении которого делается поручение |
| groupPortfolioName | id, который идентифицирует группу бумаг пользователя - в дальнейшем бумаги в позициях будут при наличии этого признака != 0 сгруппированы по данному признаку. Например, отправив две сделки с указанием groupPortfolioName == 12, в портфеле пользователя появится отдельная группа бумаг с этими двумя бумагами |
| userOrderId | Признак для отслеживания выставляемых приказов по внутреннему (со стороны клиента) признаку |

```javascript
var ws = io('https://wsbeta.tradernet.ru');

var putData = {
    instr_name: 'SBER',
    action_id: 1,
    order_type_id: 1,
    curr: 'RUR',
    limit_price: 50,
    stop_price: 40,
    qty: 100,
    aon: 0,
    expiration_id: 1,
    submit_ch_c: 1,
    message_id: 0,
    replace_order_id: 0,
    groupPortfolioName: 12,
    userOrderId: 123
};

ws.emit('putOrder', putData, function (err, res) {
    console.log(err, res);
});
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/fosxg6vx/)

*Результат*

Возвращает идентификатор приказа

```json
{ 
    "orderId": 6949116
}
```

<a name="deleteOrder"></a>
### deleteOrder

Отмена приказа

```javascript
var ws = io('https://wsbeta.tradernet.ru');

var putData = {
    instr_name: 'SBER',
    action_id: 1,
    order_type_id: 1,
    curr: 'RUR',
    limit_price: 50,
    stop_price: 40,
    qty: 100,
    aon: 0,
    expiration_id: 1,
    submit_ch_c: 1,
    message_id: 0,
    replace_order_id: 0,
    userOrderId: 123
};

ws.emit('putOrder', putData, function (err, res) {
    if (res && res.orderId) deleteOrder(res.orderId);
});

function deleteOrder(orderId) {
    var delData = {
        order_id: orderId,
    };

    ws.emit('deleteOrder', delData, function (err, res) {
        console.log(err, res);
    });
}
```
[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/pqmpx9nt/)

<a name="sessions"></a>
## Сессии безопасности

<a name="getSafetyTypes"></a>
### getSafetyTypes 

Получение списка типов сессий безопасности.

```javascript
var ws = io('https://wsbeta.tradernet.ru');

ws.emit('getSafetyTypes', function (err, safetyTypes) {
    console.log(err, safetyTypes);
});
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/j5gvbckq/)

*Результат:*

| Название | Описание |
| -------- | -------- |
| safety_type_id | Идентификатор типа сессии безопасности |
| safety_type | Идентификатор тип сессии безопасности |
| description | Тип сессии безопасности, отображаемый в интерфейсе пользователя |
| enabled | Доступен или нет клиенту |
| status | Дата истечения сессии безопасности |
| status_description | Время, оставшееся до истечения сессии безопасности (мс) |
| status_id | Время, оставшееся до истечения сессии безопасности (мс) |

*Пример:*

```json
[
  {
    "safety_type_id": 2,
    "safety_type": "Token Aladdin",
    "description": "Использование генератора временных паролей компании Aladdin",
    "enabled": false,
    "status": "Использование недоступно",
    "status_description": "Использование этого уровня безопасности Вам не разрешено. За разъяснениями Вы можете обратиться в службу поддержки.",
    "status_id": 0
  },
  {
    "safety_type_id": 3,
    "safety_type": "SMS",
    "description": "Подтверждение с помощью SMS",
    "enabled": true,
    "status": "Доступно [PHONE_NUMBER]",
    "status_id": 1
  },
  {
    "safety_type_id": 4,
    "safety_type": "Логин, пароль",
    "description": "Без дополнительного подтверждения",
    "enabled": true,
    "status": "Доступно",
    "status_id": 1
  }
]
```

<a name="getSecuritySessions"></a>
### getSecuritySessions

Получение списка открытых клиентом сессий безопасности

```javascript
var ws = io('https://wsbeta.tradernet.ru');

ws.emit('getSecuritySessions', function (err, sessions) {
    console.log(err, sessions);
});
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/n412k8rh/)

*Результат:*

Результат выполнения передаётся в аргументы *err* и *sessions* колбэка.

Аргумент *sessions* содержит список открытых клиентом сессий безопасности.

```json
[
  {
    "id": 6760536,
    "safety_type_id": 4,
    "safety_type": "Логин, пароль",
    "start_datetime": "2014-11-06T17:57:46.967",
    "expire_datetime": "2014-11-07T17:57:46.967",
    "expire": 86004
  }
]
```

*Параметры сессии*

| Название | Описание |
| -------- | -------- |
| id | Идентификатор сессии |
| safety_type_id | Идентификатор типа сессии безопасности |
| safety_type | Тип сессии безопасности, отображаемый в интерфейсе пользователя |
| start_datetime | Дата открытия сессии |
| expire_datetime | Дата истечения сессии безопасности |
| expire | Время, оставшееся до истечения сессии безопасности (мс) |

<a name="initValidation"></a>
### initValidation

Инициализация двухэтапного открытия сессии безопасности.
Отправляет код подтверждения указанным способом (SMS). 

*Параметры*

| Название | Описание |
| -------- | -------- |
| safetyTypeId | идентификатор типа сессиия безопасности |

```javascript
var ws = io('https://wsbeta.tradernet.ru');

// SMS
var safetyTypeId = 3;

ws.emit('initValidation', safetyTypeId, function (err, res) {
    console.log(err, res);
});
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/pabzmkky/)

*Результат:*

Результат выполнения передаётся в аргументы *err* и *res* колбэка.

Аргумент *err* принимает объект ошибки или *null*. 

В случае успешной отправки аргумент *res* получает значение *0*.


<a name="openSecuritySession"></a>
### openSecuritySession

Открытие сессии безопасности. 
Если был выбран тип безопасности с подтверждением, проверяет код подтверждения, укананный клиентом.
Если тип безопасности не требует подтверждения, просто открывает сессию.

*Параметры*

| Название | Описание |
| -------- | -------- |
| safetyTypeId | идентификатор типа сессиия безопасности |
| validationKey | код подтверждения. Если тип безопасности не требует подтверждения, должен быть null |

```javascript
var ws = io('https://wsbeta.tradernet.ru');

var data = {
    safetyTypeId: 3,     // SMS
    validationKey: 'CODE FROM SMS'
};

ws.emit('openSecuritySession', data, function (err, res) {
    console.log(err, res);
});
```

[Запустить на JSFIDDLE](http://jsfiddle.net/papageno/pabzmkky/)

*Результат:*

Результат выполнения передаётся в аргументы *err* и *res* колбэка.

Если возникла ошибка, аргумент *err* принимает объект ошибки, иначе *null*. 

Если сессия открылась, аргумент *res* получает идентификатор сессии безопасности.


<a name="tokens"></a>
## Токены безопасности

<a name="activateToken"></a>
### activateToken

Активация токена. Передаётся серийный номер токена.

```javascript
var ws = io('https://wsbeta.tradernet.ru');

var tokenSN = 'TOKEN_SN';

ws.emit('activateToken', tokenSN, function (err, res) {
    console.log(err, res);
});
```

<a name="syncToken"></a>
### synchronizeToken

Синхронизация токена. Передаётся два последовательных ключа, выданных токеном.

```javascript
var ws = io('https://wsbeta.tradernet.ru');

var key1 = 'KEY1';
var key2 = 'KEY2';

ws.emit('synchronizeToken', key1, key2, function (err, res) {
    console.log(err, res);
});
```

<a name="tokenInfo"></a>
### getTokenInfo

Возвращает информацию о токене

```javascript
var ws = io('https://wsbeta.tradernet.ru');

ws.emit('getTokenInfo', function (err, res) {
    console.log(err, res);
});
```

