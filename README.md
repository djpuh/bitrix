# Debug

```php
use Bitrix\Main\Diag\Debug;

// Запись отладочной информации в файл
Bitrix\Main\Diag\Debug::writeToFile(array('ID' => $id, 'fields'=>$fields ),"","/logs/logname.log");
Bitrix\Main\Diag\Debug::dumpToFile(array('ID' => $id, 'fields'=>$fields ),"","/logs/logname.log");

// Время исполнения скрипта
// В начале исследуемого участка кода, добавляем: 
Bitrix\Main\Diag\Debug::startTimeLabel('test 1');
// в конец: 
Bitrix\Main\Diag\Debug::endTimeLabel('test 1'); 
// И для вывода используем: 
Bitrix\Main\Diag\Debug::getTimeLabels(); 

// Получение текущей метки времени
Bitrix\Main\Diag\Helper::getCurrentMicrotime(); 

// Получение стека вызова функций
Bitrix\Main\Diag\Helper::getBackTrace($limit = 0, $options = null););

```

# Работа с iblocks

Для работы с элементами используется класс \Bitrix\Iblock\Elements\ElementXXXXXTable, где XXXXX - Символьный код API. Для инфоблока "Catalog" это будет класс \Bitrix\Iblock\Elements\ElementCatalogTable. Получить название класса для инфоблока можно следующим методом:

```php
var_dump(\Bitrix\Iblock\Iblock::wakeUp($iblockId)->getEntityDataClass());
// string(43) "\Bitrix\Iblock\Elements\ElementCatalogTable"
```

Аналог CIBlockElement::GetByID:

```php
$element = \Bitrix\Iblock\Elements\ElementCatalogTable::getByPrimary($elementId, [
    'select' => ['ID', 'NAME', 'DETAIL_TEXT', 'DETAIL_PICTURE', 'ARTNUMBER'],
])->fetch();
// при помощи алиасов
$element = \Bitrix\Iblock\Elements\ElementCatalogTable::getByPrimary($elementId, [
    'select' => ['ID', 'NAME', 'ARTNUMBER_' => 'ARTNUMBER'],
])->fetch();
// Один элемент
$element = \Bitrix\Iblock\Elements\ElementCatalogTable::getByPrimary($elementId, array(
    'select' => array('ID', 'NAME', 'DETAIL_PICTURE')
))->fetchObject();

var_dump($element->getId());
// int(6)
var_dump($element->getName());
// string(42) "Штаны Цветочная Поляна"
var_dump($element->getDetailPicture());
// int(48)
// Несколько элементов
$elements = \Bitrix\Iblock\Elements\ElementCatalogTable::getList([
    'select' => ['ID', 'NAME', 'DETAIL_PICTURE'],
    'filter' => [
        'ID' => $elementId,
    ],
])->fetchCollection();


```

Аналог CIBlockElement::GetList

```php
$nowDate = new \Bitrix\Main\Type\DateTime();//текущая дата
        $nowYear= ConvertDateTime($nowDate, "Y", "ru");
        $nextYear = intval($nowYear) + 1;
        $dateTimeFrom = new \Bitrix\Main\Type\DateTime("$nowYear-01-01 00:00:00", "Y-m-d H:i:s");
        $dateTimeTo= new \Bitrix\Main\Type\DateTime("$nextYear-01-01 00:00:00", "Y-m-d H:i:s");

        // Новости за этот год
        $order = array('DATE_CREATE' => 'DESC');
        $filter = array(
            "=IBLOCK_ID" => $this->iblockID,
             "=ACTIVE" => "Y",
             ">=DATE_CREATE"   => $dateTimeFrom,
             "<DATE_CREATE" => $dateTimeTo,
            );

        $arSelect = array('ID',
            "PREVIEW_PICTURE",
            "PREVIEW_TEXT",
            "NAME",
            "ACTIVE_FROM",
            "DATE_CREATE",
            "DETAIL_TEXT",
            "DETAIL_PICTURE",
            );
        $cache = array();
        if ($this->cacheType == "A" || $this->cacheType == "Y")     
            $cache = array(
                'ttl' => $this->cacheTime,
                'cache_joins' => true,
            );
        
        $newsList = \Bitrix\Iblock\ElementTable::getList([
                'order' => $order,
                "select" => $arSelect,
                "filter" => $filter,
                "count_total" => true,
                'cache' => $cache,
                "limit" => $this->nPageSize,
        ])->fetchAll();

```

Разделы

```php
// Разделы элемента
$elements = \Bitrix\Iblock\Elements\ElementCatalogTable::getList([
    'select' => ['ID', 'IBLOCK_SECTION'],
    'filter' => [
        'ID' => $elementId,
    ],
])->fetchObject();

var_dump ($element->getIblockSection()->getName());
// string(10) "Штаны"

// Разделы
$element = \Bitrix\Iblock\Elements\ElementCatalogTable::getList([
    'select' => ['ID', 'SECTIONS'],
    'filter' => [
        'ID' => $elementId,
    ],
])->fetchObject();

foreach ($element->getSections()->getAll() as $section) {
    var_dump($section->getId());
    // int(8)
    var_dump($section->getCode());
    // string(5) "pants"
    var_dump($section->getName());
    // string(10) "Штаны"
}
```

Свойства

```php
// Текст или Число
$element = \Bitrix\Iblock\Elements\ElementCatalogTable::getByPrimary($elementId, array(
    'select' => array('ID', 'ARTNUMBER')
))->fetchObject();

var_dump($element->getArtnumber()->getValue());
// string(11) "177-79-00"

// Справочник
$elements = \Bitrix\Iblock\Elements\ElementCatalogTable::getList([
    'select' => ['ID', 'NAME', 'DETAIL_PICTURE', 'BRAND_REF'],
    'filter' => [
        'ID' => $elementId,
    ],
])->fetchCollection();

foreach ($elements as $element) {
    foreach ($element->getBrandRef()->getAll() as $value) {
        var_dump($value->getValue());
    }
}
// string(8) "company2"
// string(8) "company3"

// Файл
$elements = \Bitrix\Iblock\Elements\ElementCatalogTable::getList([
    'select' => ['ID', 'MORE_PHOTO.FILE'],
    'filter' => [
        'ID' => $elementId,
    ],
])->fetchCollection();

foreach ($elements as $element) {
    foreach ($element->getMorePhoto()->getAll() as $value) {
        var_dump('/upload/' . $value->getFile()->getSubdir().'/'.$value->getFile()->getFileName());
    }
}

//string(55) "/upload/iblock/2d6/2d6d01a93d220c012e40415616a3d0e1.jpg"
//string(55) "/upload/iblock/dd3/dd3495f96990223e3076c97e99b062d9.jpg"

// Список
$element = \Bitrix\Iblock\Elements\ElementCatalogTable::getList([
    'select' => ['ID', 'NEWPRODUCT'],
    'filter' => [
        'ID' => $elementId,
    ],
])->fetchObject();
var_dump($element->getNewproduct()->getValue());
// int(1)

//Если требуется получить ID, XML_ID и значение, то используем ITEM после код свойства, тогда к запросу добавится таблица значений списочных свойств:

$element = \Bitrix\Iblock\Elements\ElementCatalogTable::getList([
    'select' => ['ID', 'NEWPRODUCT.ITEM'],
    'filter' => [
        'ID' => $elementId,
    ],
])->fetchObject();
var_dump($element->getNewproduct()->getItem()->getId());
// int(1)
var_dump($element->getNewproduct()->getItem()->getXmlId());
// string(1) "Y"
var_dump($element->getNewproduct()->getItem()->getValue());
// string(4) "да"

//У свойств типов Привязка к элементам и Привязка к разделам есть подобное дополнительное поле (ELEMENT/SECTION) для доступа к элементу/разделу:

$element = \Bitrix\Iblock\Elements\ElementCatalogTable::getList([
    'select' => ['ID', 'RECOMMEND.ELEMENT'],
    'filter' => [
        'ID' => $elementId,
    ],
])->fetchObject();

foreach ($element->getRecommend()->getAll() as $linkElement) {
    var_dump($linkElement->getElement()->getId());
    // int(29)
    var_dump($linkElement->getElement()->getName());
    // string(34) "Платье Красная Фея"
}

```


# DB->Query()

```php
Bitrix\Main\Loader::includeModule('iblock');

// создаем объект Query, в качестве параметра передаем объект сущности (элемент инфоблока)
$query = new Bitrix\Main\Entity\Query(
    Bitrix\Iblock\ElementTable::getEntity()
);
// выбираем идентификатор элемента, символьный код и наименование
$query->setSelect(array('ID', 'CODE', 'NAME'))
      // идентификатор инфоблока равен 5
      ->setFilter(array('IBLOCK_ID' => 5))
      // сортируем элементы по идентификатору, по возрастанию
      ->setOrder(array('ID' => 'ASC'))
      // выбираем только три элемента
      ->setLimit(3);
// посмотрим, какой запрос был сформирован
echo $query->getQuery();
// выполняем запрос
$result = $query->exec();
// выводим результат
while ($row = $result->fetch()) {
    debug($row);
}
```

Список методов Bitrix\Main\Entity\Query:

- setSelect(), setGroup() — устанавливает массив с именами полей
- addSelect(), addGroup() — добавляет имя поля
- getSelect(), getGroup() — возвращает массив с именами полей
- setFilter() — устанавливает одно- или многомерный массив с описанием фильтра
- addFilter() — добавляет один параметр фильтра со значением
- getFilter() — возвращает текущее описание фильтра
- setOrder() — устанавливает массив с именами полей и порядком сортировки
- addOrder() — добавляет одно поле с порядком сортировки
- getOrder() — возвращает текущее описание сортировки
- setLimit(), setOffset() — устанавливает значение
- getLimit(), getOffset() — возвращает текущее значение
- registerRuntimeField() — регистрирует новое временное поле для исходной сущности

# Работа с датой

```php

use Bitrix\Main\Type\DateTime;
// Аналог встроенного в PHP класса \DateTime
$objDateTime = DateTime::createFromPhp(new \DateTime('2000-01-01'));
$objDateTime = DateTime::createFromTimestamp(1346506620);
// Конструкторы
// Текущее время:
$objDateTime = new DateTime();
// Из строки в формате текущего сайта
$objDateTime = new DateTime("25.12.2012 12:30:00");
// Из строки с указанием формата:
$objDateTime = new DateTime("2007-05-14 12:10:00", "Y-m-d H:i:s");
// Методы
echo $objDateTime->getTimestamp();
//в виде строки в формате текущего сайта:
echo $objDateTime->toString();
//в произвольном формате (фактически обёртка над DateTime::format):
echo $objDateTime->format("Y-m-d H:i:s");
// Метод add реализует сложение и вычитание дат, можно указывать смещение словами years, months, days, weeks, hours, minutes, seconds и знаками +/-:
$objDateTime = new DateTime("01.01.2012 00:00:00"); // "2012-01-01 00:00:00"
$objDateTime->add("1 day"); // "2012-01-01 00:00:00" => "2012-01-02 00:00:00"
$objDateTime->add("-1 day"); // "2012-01-01 00:00:00" =>"2011-12-31 00:00:00"
$objDateTime->add("3 months - 5 days + 10 minutes"); // "2012-01-01 00:00:00" =>"2012-03-27 00:10:00"
$objDateTime->add("7M5DT2M"); // "2012-01-01 00:00:00" =>"2012-08-06 00:02:00"
$objDateTime->add("-2YT10M"); // "2012-01-01 00:00:00" =>"2009-12-31 23:50:00"

```

# Валидный JSON

```php
return Bitrix\Main\Web\Json::encode($arData);

\Bitrix\Main\Web\Json::decode(string $data);

```
# REQUEST


```php
use Bitrix\Main\Context;
// implements Dictionary, Request, Uri

$request = Context::getCurrent()->getRequest();
// работа с параметрами
getValues();         @return array
get($name);          @return string | array | null
toArray();           @return array
isEmpty();           @return bool
set($name, $value = null);
setValues($values);  @param $values
getValue($name);     @return string | array | null
getQuery($name);     @return string | array | null
getQueryList();      @return Type\ParameterDictionary
getPost($name);      @return string | array | null
getPostList();       @return Type\ParameterDictionary
getFile($name);      @return string | array | null
getFileList();       @return Type\ParameterDictionary
getCookie($name);    @return null|string
getCookieList();     @return Type\ParameterDictionary
getCookieRaw($name); @return null|string

$request = Context::getCurrent();
getJsonList();
getCulture();       @return Context\Culture | null
isAjaxRequest();    @return bool
// Uri
getServer();
getRequestUri();    @return null|string
getRequestMethod(); @return null|string
getRequestedPage(); @return string
getDecodedUri();    @return string
```
# BACKGROUNDJOBS

```php
Application::getInstance()->addBackgroundJob(function () use ($filePath) {
    unlink($filePath);
});
```
