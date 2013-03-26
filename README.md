# API Яндекс.Карт и БЭМ
## Введение

Один из самых частых кейсов использования API Яндекс.Карт — создание меню для показа на карте организаций различных типов (коллекций геообъектов). С помощью такого меню пользователь сайта может отобразить на карте только те объекты, которые его интересуют. Например, [вот так](http://dimik.github.com/ymaps/examples/group-menu/menu03.html). Но давайте реализуем этот пример с помощью методологии БЭМ!

## Первые шаги
Создатели БЭМ позаботились о разработчиках и создали проект-скелет, который поможет начать разработку «с низкого старта». Начнем с него.

````bash
    git clone https://github.com/bem/project-stub.git shopsList
    cd shopsList
    npm install
````

Теперь проект у нас на компьютере. Давайте протестируем, все ли работает. Для этого нужно перейти в папку, запустить make, подождать пока проект соберется и открыть в браузере страницу: [localhost:8080/desktop.bundles/index/index.html](http://localhost:8080/desktop.bundles/index/index.html)
Перед глазами должно быть что-то вроде:

<img src="http://zloylos.me/other/imgs/ymapsbem/project_stub.png" title="" alt="" border="0"/>

Готово, можно переходить к следующему этапу.

## Общее видение проекта

Нам нужно создать блок «map», в котором будет находиться карта, блок «sidebar» — правая или левая колонка, в которой будет находиться блок «menu», реализующий список организаций по категориям. Методология БЭМ подсказывает, что мы должны проектировать так, чтобы блоки не знали о существовании друг друга, поэтому нужно будет создать промежуточный блок, который будет принимать клики на меню и взаимодействовать с картой. Назовем его «i-geo-controller».

Чтобы лучше понять зачем нужен промежуточный блок, как он работает и что такое «миксы», рекомендуем посмотреть [рассказ Кира Белевича «Миксы во вселенной БЭМ»](http://events.yandex.ru/events/yasubbotnik/msk-sep-2012/talks/327/).

## Описание страницы, написание bemjson.

Здесь всё просто. Мы изначально продумали структуру страницы, основные блоки и теперь осталось все это записать в json-подобном виде. Не буду вдаваться в подробности, вы всегда можете посмотреть исходный код. Структура страницы будет такой:

````
    b-page
    == container
    ==== map
    ==== sidebar
    ====== menu
    ======== menu
    ========== items
````

<img src="http://zloylos.me/other/imgs/ymapsbem/index_bemjson.png" alt="">
￼

Более подробно можно посмотреть здесь: [desktop.bundles/index/index.bemjson.js](https://github.com/zloylos/ymaps-and-bem/blob/master/desktop.bundles/index/index.bemjson.js)

## Блок map

Давайте начнем разработку с главного блока — карты. Прежде всего нужно подключить API с необходимыми опциями. Можно было бы создать отдельный блок i-API, но, кажется, куда удобнее реализовать все это в рамках одного блока, используя модификаторы. Для блока map мы создадим модификатор «api», в котором для начала разместим значение — «ymaps».
В примере мы будем использовать [динамический API](http://api.yandex.ru/maps/doc/jsapi/), но нужно помнить, что мы можем использовать и [Static API](http://api.yandex.ru/maps/doc/staticapi/). Это можно реализовать в рамках модификаторов.

Для удобной работы с картой нам стоит продумать интерфейс добавления меток на карту без лишней головной боли. Для этого стоит сделать обработку поля: geoObjects, в котором будут храниться метки или коллекции. Для метки сделаем такой интерфейс:

````js
    {
        coords: [], // координаты метки
        properties: {}, // данные метки 
        options: {}, // опции метки
    }
````

Для коллекции:
````js
    {
        collection: true, // флаг, указывающий, что это коллекция / группа меток
        properties: {}, // свойства группы
        options: {}, // опции группы
        data: [], // массив меток. 
    }
````
Это покрывает 90% всех кейсов.

## Блок menu

Здесь нам нужно сделать двухуровневое меню. Создаем блок menu, который будет распознавать клики по группам и элементам. Соответственно, нам нужны такие элементы:
- item — элемент меню;
- content — контейнер для элементов;
- title — заголовок группы.

Вкладывая один блок меню в другой, можно добиться необходимой иерархичности.

## Блок i-geo-controller

Блок-контроллер, который подписывается на события блоков menu — «menuItemClick» и «menuGroupClick», реагирует на них и совершает определенные действия на карте. В нашем примере у него следующие задачи:
- при клике на метку нужно переместить ее в центр карты и открыть балун;
- при клике на группу нужно либо скрыть ее, либо показать, если до этого она была скрыта.
Кроме того, чтобы правильно взаимодействовать с картой, блок-контроллер должен знать, когда карта готова к тому, чтобы объектами на ней можно было управлять. Для этого блок map у себя будет пробрасывать событие «map-inited», а i-geo-controller — слушать его и запоминать ссылку на экземпляр карты.
￼

<img src="http://zloylos.me/other/imgs/ymapsbem/blocks_scheme.png" alt="">


## Заключение

Готовый пример можно посмотреть по ссылке: [zloylos.github.com/ymaps-and-bem/](http://zloylos.github.com/ymaps-and-bem/).

Возможно, с использованием методологии БЭМ пример и получился более громоздким, чем без неё, но зато у нас есть более структурированный и удобный для поддержки код. А главное, его довольно просто масштабировать и расширять, что без использования методологии вызвало бы значительные проблемы и чаще всего привело бы к полному переписыванию кода.

<img src="http://zloylos.me/other/imgs/ymapsbem/ready.png" alt="">
￼