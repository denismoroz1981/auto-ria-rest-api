# REST API сайта AUTO.RIA.com

Все методы данного API отдают данные в формате JSON. Формат данных в большинстве случаев стандартный - коллекция объектов с полями *name* (название объекта) и *value* (его идентификатор).

Идентификатор любой сущности является целым числом.

## Содержание

В этой документации можно найти описание работы со следующими сущностями:
- Типы транспорта
- Типы кузова
- Марки
- Модели
- Области
- Города
- Коробки передач
- Типы привода
- Типы топлива
- Опции
- Цвета

А также, к методу подсчета средней цены.

## 1. Подсчет средней цены

### 1.1. Список поддерживаемых параметров

На данный момент поддерживаются следующие параметры:

|  Название           | Параметр в строке запроса | Тип данных   |
|:--------------------|:--------------------------|:------------:|
|  Тип транспорта     |  category_id              |  Number      |
|  Тип кузова         |  body_id                  |  Number      |
|  Марка              |  marka_id                 |  Number      |
|  Модель             |  model_id                 |  Number      |
|  Год выпуска        |  yers                     |  Number\[\]  |
|  Коробка передач    |  gear_id                  |  Number      |
|  Тип топлива        |  fuel_id                  |  Number      |
|  Тип привода        |  drive_id                 |  Number      |
|  Объем двигателя    |  engineVolume             |  Number      |
|  Опции              |  options                  |  Number      |
|  Пробег             |  raceInt                  |  Number\[\]  |
|  Количество дверей  |  door                     |  Number      |
|  Область            |  state_id                 |  Number      |
|  Город              |  city_id                  |  Number      |
|  Грузоподъемность   |  carrying                 |  Number      |
|  Количество мест    |  seats                    |  Number      |
|  Цвет               |  color_id                 |  Number      |

### 1.2. Формат данных в запросе

Все параметры описанные в таблице поддерживаемых параметров должны передаватся в виде чисел. Исключениями являются только два параметра - *год выпуска* и *пробег*.

Если передать массив в параметре *год выпуска* или *пробег* это будет интерпретироваться как диапазон значений. Например, `http://api.auto.ria.com/average?raceInt=10&raceInt=100` - выберет для подсчета средней цены все объявления с пробегом от 10 до 100 тыс. км.

### 1.3. Формат данных в ответе

В случае успешного подсчета средней цены по указанным параметрам результат будет со статусом **200 OK**.

Пример успешного ответа:
```javascript
{
    total: 17,
    arithmeticMean: 16305.882352941177,
    interQuartileMean: 8483.333333333334,
    percentiles: {
        1.0: 1944,
        5.0: 2520,
        25.0: 3500,
        50.0: 8000,
        75.0: 23500,
        95.0: 53539.999999999985,
        99.0: 64868
    },
    prices: [
        67700,
        27000,
        3000,
        23500,
        3500,
        8100,
        10000,
        3500,
        2700,
        8000,
        11000,
        45800,
        50000,
        1800,
        4350,
        4400,
        2850
    ],
    classifieds: [
        14663610,
        14226353,
        14138132,
        13969588,
        14697569,
        13386778,
        13279188,
        14555863,
        14754932,
        14816842,
        14664706,
        13873344,
        14681607,
        14772056,
        14059841,
        14290096,
        14890250
    ]
}
```
Расшифровка параметров:
- *total* - общее количество объявлений, учавствующих в подсчете.
- *arithmeticMean* - [среднее арифметическое](https://ru.wikipedia.org/wiki/%D0%A1%D1%80%D0%B5%D0%B4%D0%BD%D0%B5%D0%B5_%D0%B0%D1%80%D0%B8%D1%84%D0%BC%D0%B5%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B5).
- *interQuartileMean* - среднее арифметическое из значений, находящихся между первым и четвертым [квантилем](https://ru.wikipedia.org/wiki/%D0%9A%D0%B2%D0%B0%D0%BD%D1%82%D0%B8%D0%BB%D1%8C). Грубо говоря, это среднее арифметическое без учета 25% самых маленьких и самых больших значений.
- *percentiles* - значения [процентилей](https://ru.wikipedia.org/wiki/%D0%9A%D0%B2%D0%B0%D0%BD%D1%82%D0%B8%D0%BB%D1%8C#.D0.9F.D0.B5.D1.80.D1.86.D0.B5.D0.BD.D1.82.D0.B8.D0.BB.D1.8C). Т.е. для данного примера 25% всех объявлений имеют цену ниже $3500.
- *prices* - список цен, которые учавствовали в подсчете средней цены. Размер ограничен 1000 элементов.
- *classifieds* - идентификаторы объявлений, к которым принадлежат цены соответственно. Размер ограничен 1000 элементов.

Если по каким-либо причинам не удалось подсчитать среднюю цену, ответ будет иметь статус **400 Bad Request**, а тело ответа будет содержать следующее:
```javascript
{ "message": "Not Enough Data" }
```

### 1.4. Примеры

## 2. Методы для работы с типами транспорта и кузова

### 2.1. Типы транспорта

Получить список типов транспорта можно отправив GET запрос на адрес [http://api.auto.ria.com/categories](http://api.auto.ria.com/categories). Результат будет примерно следующим:
```javascript
[
    { name: "Легковые", value: 1 },
    { name: "Мото", value: 2 },
    { name: "Водный транспорт", value: 3 },
    { name: "Спецтехника", value: 4 },
    { name: "Прицеп", value: 5 },
    { name: "Грузовик", value: 6 },
    { name: "Автобус", value: 7 },
    { name: "Автодом", value: 8 },
    { name: "Воздушный транспорт", value: 9 }
]
```

### 2.2. Типы кузова

Типы кузова зависят от типов транспорта. Поэтому для того, чтобы получить список типов кузова необходимо отправить GET запрос на адрес `http://api.auto.ria.com/categories/:categoryId/bodystyles`, где *categoryId* - идентификатор типа транспорта.

Например, для легковых автомобилей ([http://api.auto.ria.com/categories/1/bodystyles](http://api.auto.ria.com/categories/1/bodystyles)), результат будет следующим:
```javascript
[
    { name: "Седан", value: 3 },
    { name: "Внедорожник / Кроссовер", value: 5 },
    { name: "Минивэн", value: 8 },
    { name: "Хэтчбек", value: 4 },
    { name: "Универсал", value: 2 },
    { name: "Купе", value: 6 },
    { name: "Легковой фургон (до 1,5 т)", value: 254 },
    { name: "Кабриолет", value: 7 },
    { name: "Пикап", value: 9 },
    { name: "Лимузин", value: 252 },
    { name: "Другой", value: 28 }
]
```

Также типы кузова могут быть разделены на группы. Это актуально для спецтехники. Поэтому существует способ получить сгруппированные типы кузова отправив GET запрос по адресу `http://api.auto.ria.com/categories/:categoryId/bodystyles/_group`, где *categoryId* - идентификатор типа транспорта.

Например, для мотоциклов ([http://api.auto.ria.com/categories/2/bodystyles/_group](http://api.auto.ria.com/categories/2/bodystyles/_group)), результат будет следующим:
```javascript
[
    [
        { name: "Мопеды", value: 58 },
        { name: "Скутер / Мотороллер", value: 11 },
        { name: "Макси-скутер", value: 12 }
    ],
    [
        { name: "Мотоциклы", value: 13 },
        { name: "Мотоцикл Без обтекателей (Naked bike)", value: 15 },
        { name: "Мотоцикл Внедорожный (Enduro)", value: 21 },
        { name: "Мотоцикл Кастом", value: 30 },
        { name: "Мотоцикл Классик", value: 14 },
        { name: "Мотоцикл Кросс", value: 19 },
        { name: "Мотоцикл Круизер", value: 24 },
        { name: "Мотоцикл Многоцелевой (All-round)", value: 25 },
        { name: "Мотоцикл с коляской", value: 29 },
        { name: "Спортбайк", value: 18 },
        { name: "Мотоцикл Спорт-туризм", value: 17 },
        { name: "Мотоцикл Супермото (Motard)", value: 22 },
        { name: "Мотоцикл Триал", value: 20 },
        { name: "Мотоцикл Туризм", value: 16 },
        { name: "Мотоцикл Чоппер", value: 23 }
    ],
    [
        { name: "Мини мотоциклы", value: 31 },
        { name: "Мини спорт", value: 32 },
        { name: "Мини крос (Питбайк)", value: 33 }
    ],
        { name: "Трицикл", value: 34 },
        { name: "Трайк", value: 57 },
    [
        { name: "Квадроциклы", value: 35 },
        { name: "Квадроцикл детский", value: 36 },
        { name: "Квадроцикл спортивный", value: 39 },
        { name: "Квадроцикл утилитарный", value: 41 },
        { name: "Мотовездеход", value: 42 },
        { name: "Вездеход-амфибия", value: 43 },
        { name: "Гольф-кар", value: 44 },
        { name: "Картинг", value: 45 }
    ],
    { name: "Снегоход", value: 46 },
    { name: "Другое", value: 56 }
]
```

Формат данных при этом отличается от обычного - это коллекция объектов, в которой есть другие коллекции. Группа типов кузовов всегда начинается с её названия. Например, в группу *Квадроциклы* входят типы кузовов *Квадроцикл детский*, *Квадроцикл спортивный*, *Квадроцикл утилитарный*, *Мотовездеход* и т.д.

Также, при необходимости, можно получиться просто весь список типов кузовов, послав GET запрос по адресу [http://api.auto.ria.com/bodystyles](http://api.auto.ria.com/bodystyles).

## 3. Методы для работы с марками и моделями

### 3.1. Марки

Марки зависят от типов транспорта. Поэтому для того, чтобы получить список марок необходимо отправить GET запрос по адресу `http://api.auto.ria.com/categories/:categoryId/marks`, где *categoryId* - идентификатор типа транспорта.

Например, для легковых автомобилей ([http://api.auto.ria.com/categories/1/marks](http://api.auto.ria.com/categories/1/marks)), результат будет следующим:
```javascript
[
    { name: "Acura", value: 98 },
    { name: "Adler", value: 2396 },
    { name: "Aixam", value: 2 },
    { name: "Alfa Romeo", value: 3 },
    { name: "Alpine", value: 100 },
    { name: "Altamarea", value: 3988 },
    { name: "Aro", value: 101 },
    { name: "Artega", value: 3105 },
    { name: "Asia", value: 4 },
    { name: "Aston Martin", value: 5 },
    { name: "Audi", value: 6 },
    { name: "Austin", value: 7 },
    { name: "Autobianchi", value: 102 }
    ...
]
```

### 3.2. Модели

Модели зависят от типов транспорта и марок. Следовательно список марок можно получить по адресу `http://api.auto.ria.com/categories/:categoryId/marks/:markId/models`, где *categoryId* - идентификатор типа транспорта а *markId* - идентификатор марки.

Например, для мотоциклов BMW ([http://api.auto.ria.com/categories/2/marks/9/models](http://api.auto.ria.com/categories/2/marks/9/models)), список моделей будет следующим:
```javascript
[
    { name: "Adventure", value: 25290 },
    { name: "C", value: 25291 },
    { name: "CS", value: 25292 },
    { name: "DKW", value: 28318 },
    { name: "F", value: 25293 },
    { name: "G", value: 29468 },
    { name: "GS", value: 25295 },
    { name: "HP", value: 38148 },
    { name: "Independent", value: 25297 },
    { name: "K", value: 25298 },
    { name: "LT", value: 25299 },
    { name: "R", value: 25300 },
    { name: "RS", value: 32736 },
    { name: "RT", value: 25301 },
    { name: "S", value: 25302 },
    { name: "X", value: 42030 }
]
```

Модели, также как и типы кузовов, могут быть сгруппированы. Чтобы получить такой список, необходимо отправить запрос по адресу `http://api.auto.ria.com/categories/:categoryId/marks/:markId/models/_group`, где *categoryId* - идентификатор типа транспорта а *markId* - идентификатор марки.

Например, для легковых автомобилей BMW ([http://api.auto.ria.com/categories/1/marks/9/models/_group](http://api.auto.ria.com/categories/1/marks/9/models/_group)), список моделей будет следующим:
```javascript
[
    [
        { name: "1 Series (все)", value: 2161 },
        { name: "116", value: 34670 },
        { name: "118", value: 34671 },
        { name: "120", value: 34672 },
        { name: "123", value: 34673 },
        { name: "125", value: 34674 },
        { name: "130", value: 34675 },
        { name: "135", value: 34676 }
    ],
    [
        { name: "3 Series (все)", value: 3219 },
        { name: "3 Series GT", value: 43029 },
        { name: "315", value: 37454 },
        { name: "316", value: 30851 },
        { name: "318", value: 31612 },
        { name: "320", value: 31611 },
        { name: "321", value: 37389 },
        { name: "323", value: 34677 },
        { name: "324", value: 30687 },
        { name: "325", value: 29713 },
        { name: "326", value: 44061 },
        { name: "328", value: 31661 },
        { name: "330", value: 34678 },
        { name: "335", value: 34679 },
        { name: "340", value: 35568 }
    ],
    ...
    { name: "Alpina", value: 906 },
    { name: "Dixi", value: 33383 },
    { name: "I3", value: 44838 },
    { name: "I8", value: 44537 },
    { name: "Isetta", value: 32380 },
    { name: "Z1", value: 97 },
    { name: "Z3", value: 98 },
    { name: "Z4", value: 99 },
    { name: "Z8", value: 100 }
]
```
Формат данных такой же как и в случае с типа кузова - коллекция объектов, в которой могут быть другие объекты. Группа моделей всегда начинается с её названия.

Также, при необходимости, можно просто получить список всех моделей, отправив GET запрос по адресу [http://api.auto.ria.com/models](http://api.auto.ria.com/models).

## 4. Методы для работы с областями и городами

### 4.1. Области

Получить список областей можно отправив GET запрос по адресу [http://api.auto.ria.com/states](http://api.auto.ria.com/states).

Результат будет следующим:
```javascript
[
    { name: "Винницкая", value: 1 },
    { name: "Волынская", value: 18 },
    { name: "Днепропетровская", value: 11 },
    { name: "Донецкая", value: 13 },
    { name: "Житомирская", value: 2 },
    { name: "Закарпатская", value: 22 },
    { name: "Запорожская", value: 14 },
    { name: "Ивано-Франковская", value: 15 },
    { name: "Киевская", value: 10 },
    { name: "Кировоградская", value: 16 },
    { name: "Луганская", value: 17 },
    { name: "Львовская", value: 5 },
    { name: "Николаевская", value: 19 },
    { name: "Одесская", value: 12 },
    { name: "Полтавская", value: 20 },
    { name: "Республика Крым", value: 21 },
    { name: "Ровенская", value: 9 },
    { name: "Сумская", value: 8 },
    { name: "Тернопольская", value: 3 },
    { name: "Харьковская", value: 7 },
    { name: "Херсонская", value: 23 },
    { name: "Хмельницкая", value: 4 },
    { name: "Черкасская", value: 24 },
    { name: "Черниговская", value: 6 },
    { name: "Черновицкая", value: 25 }
]
```

### 4.2. Города

Города зависят от областей, поэтому, чтобы получить их список, необходимо послать GET запрос по адресу `http://api.auto.ria.com/states/:stateId/cities`, где *stateId* - идентификатор области.

Например, для Винницкой области ([http://api.auto.ria.com/states/1/cities](http://api.auto.ria.com/states/1/cities)) список городов будет следующим:
```javascript
[
    { name: "Винница", value: 1 },
    { name: "Жмеринка", value: 27 },
    { name: "Казатин", value: 30 },
    { name: "Крыжополь", value: 31 },
    { name: "Липовец", value: 32 },
    { name: "Литин", value: 33 },
    { name: "Могилев-Подольский", value: 34 },
    { name: "Мурованые Куриловцы", value: 35 },
    { name: "Немиров", value: 36 },
    { name: "Оратов", value: 37 },
    { name: "Песчанка", value: 38 },
    { name: "Погребище", value: 39 },
    { name: "Теплик", value: 40 },
    { name: "Тывров", value: 41 },
    { name: "Томашполь", value: 42 },
    { name: "Тростянец", value: 43 },
    { name: "Тульчин", value: 44 },
    { name: "Хмельник", value: 45 },
    { name: "Черновцы", value: 46 },
    { name: "Чечельник", value: 47 },
    { name: "Шаргород", value: 48 },
    { name: "Ямполь", value: 49 },
    { name: "Бар", value: 597 },
    { name: "Бершадь", value: 599 },
    { name: "Гайсин", value: 602 },
    { name: "Ильинцы", value: 603 },
    { name: "Калиновка", value: 604 },
    { name: "Гнивань", value: 609 },
    { name: "Ладыжин", value: 644 }
]
```

## 5. Методы для работы с техническими характеристиками

### 5.1. Коробки передач

Коробки передач зависят от типа транспорта, поэтому, чтобы получить их список, необходимо послать GET запрос по адресу `http://api.auto.ria.com/categories/:categoryId/gearboxes`, где *categoryId* - идентификатор типа транспорта.

Например, список коробок передач для мотоциклов ([http://api.auto.ria.com/categories/2/gearboxes](http://api.auto.ria.com/categories/2/gearboxes)) будет выглядеть следующим образом:
```javascript
[
    { name: "Ручная / Механика", value: 1 },
    { name: "Автомат", value: 2 },
    { name: "Типтроник", value: 3 },
    { name: "Адаптивная", value: 4 },
    { name: "Вариатор", value: 5 }
]
```

### 5.2. Типы привода

Типы привода также зависят от типа транспорта, поэтому, чтобы получить их список, необходимо плсать GET запрос по адресу `http://api.auto.ria.com/categories/:categoryId/driverTypes`, где *categoryId* - идентификатор типа транспорта.

Например, список типов привода для мотоциклов ([http://api.auto.ria.com/categories/2/driverTypes](http://api.auto.ria.com/categories/2/driverTypes)) выглядит следующим образом:
```javascript
[
    { name: "Кардан", value: 4 },
    { name: "Ремень", value: 5 },
    { name: "Цепь", value: 6 }
]
```

### 5.3. Типы топлива

Типы топлива можно получить отправив GET запрос по адресу [http://api.auto.ria.com/fuels](http://api.auto.ria.com/fuels). Ответ будет выглядеть так:
```javascript
[
    { name: "Бензин", value: 1 },
    { name: "Дизель", value: 2 },
    { name: "Газ", value: 3 },
    { name: "Газ/бензин", value: 4 },
    { name: "Гибрид", value: 5 },
    { name: "Электро", value: 6 },
    { name: "Другое", value: 7 },
    { name: "Газ метан", value: 8 },
    { name: "Газ пропан-бутан", value: 9 }
]
```

### 5.4. Опции

Опции зависят от типа транспорта. Получить их список можно отправив GET запрос по адресу `http://api.auto.ria.com/categories/:categoryId/options`, где *categoryId* - идентификатор типа транспорта.

Например, список опций для легковых автомобилей ([http://api.auto.ria.com/categories/1/options](http://api.auto.ria.com/categories/1/options)) будет выглядеть примерно так:
```javascript
[
    { name: "ABD", value: 354 },
    { name: "ABS", value: 217 },
    { name: "ESP", value: 459 },
    { name: "Галогенные фары", value: 463 },
    { name: "Замок на КПП", value: 481 },
    { name: "Иммобилайзер", value: 225 },
    { name: "Пневмоподвеска", value: 442 },
    { name: "Подушка безопасности (Airbag)", value: 211 },
    { name: "Серворуль", value: 485 },
    { name: "Сигнализация", value: 303 },
    ...
]
```

## 6. Цвета

Получить список всех цветов можно, если отправить GET запрос по адресу [http://api.auto.ria.com/colors](http://api.auto.ria.com/colors). Результат будет седующим:
```javascript
[
    { name: "Бежевый", value: 1 },
    { name: "Черный", value: 2 },
    { name: "Синий", value: 3 },
    { name: "Бронзовый", value: 4 },
    { name: "Коричневый", value: 5 },
    { name: "Золотой", value: 6 },
    { name: "Зеленый", value: 7 },
    { name: "Серый", value: 8 },
    { name: "Апельсин", value: 9 },
    { name: "Магнолии", value: 10 },
    { name: "Розовый", value: 11 },
    { name: "Фиолетовый", value: 12 },
    { name: "Красный", value: 13 },
    { name: "Серебряный", value: 14 },
    { name: "Белый", value: 15 },
    { name: "Желтый", value: 16 },
    { name: "Голубой", value: 17 },
    { name: "Вишнёвый", value: 18 },
    { name: "Сафари", value: 19 },
    { name: "Гранатовый", value: 20 },
    { name: "Асфальт", value: 21 }
]
```