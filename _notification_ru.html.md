# Уведомления об оплате счетов {#notification}

###### Последнее обновление: 2017-05-31 | [Редактировать на GitHub](https://github.com/QIWI-API/bill-payments-rest-api-docs/blob/master/_notification_ru.html.md)

Уведомление представляет собой POST-запрос. Тело запроса содержит сериализованные данные счета в теле запроса (кодировка UTF-8), и параметр `command=bill`.

<h3 class="request method">Запрос → POST</h3>

~~~shell
Пример

user@server:~$ curl "https://server.ru/qiwi-notify.php"
  -v -w "%{http_code}"
  -X POST
  --header "Accept: application/json"
  --header "Content-Type: application/x-www-form-urlencoded; charset=utf-8"
  --header "X-Api-Signature: J4WNfNZd***V5mv2w="
  -d 'prv_id=2040&bill_id=BILL-1&status=paid&error=0&amount=1.00&phone=tel%3A%2B79031811737&payment_date=2017-03-01T19%3A00%3A00&currency=RUB&comment=test&version=1'

HTTP/1.1 200 OK
Content-Type: application/json

{"error": 0}
~~~

~~~http
Пример
POST /qiwi-notify.php HTTP/1.1
Accept: application/json
Content-type: application/x-www-form-urlencoded
X-Api-Signature: J4WNfNZd***V5mv2w=
Host: server.ru

prv_id=2040&bill_id=BILL-1&status=paid&error=0&amount=1.00&phone=tel%3A%2B79031811737&payment_date=2017-03-01T19%3A00%3A00&currency=RUB&comment=test&version=1

HTTP/1.1 200 OK
Content-Type: application/json

{"error": 0}
~~~

<ul class="nestedList url">
    <li><h3>URL</h3>
    </li>
</ul>

<aside class="notice">
Aдрес вашего сервера для уведомлений вы можете настроить на сайте <a href="https://kassa.qiwi.com/">kassa.qiwi.com</a>

<ul class="nestedList notice_image">
   <li><h3>Подробнее</h3>
        <ul>
           <li><img src="images/pull_rest_notification_url.png" /></li>
        </ul>
   </li>
</ul>
</aside>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>X-Api-Signature: ***</li>
             <li>Accept: application/json</li>
             <li>Content-type: application/x-www-form-urlencoded</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>В POST-запросе уведомления указываются параметры счета.</span>
    </li>
</ul>

Параметр|Описание|Тип
---------|--------|---
prv_id|[Идентификатор сайта](#auth_param) провайдера в API|String
bill_id|Уникальный идентификатор счета в системе провайдера|String(30)
amount | Сумма счета. Способ округления зависит от валюты | Number(6.3)
currency | Идентификатор валюты счета (Alpha-3 ISO 4217 код) | String(3)
status | [Статусы](#status)|String
error | [Ошибки](#errors)| String(20)
user | Номер Visa Qiwi Wallet, на который был выставлен счет | Number
email | E-mail пользователя (если был указан при выставлении счета). | String
user_id | Идентификатор пользователя в системе провайдера (если был указан при выставлении счета). | String
payment_date | Дата оплаты счета | URL-закодированная строка ГГГГ-ММ-ДДTЧЧ:ММ:СС
comment | Комментарий к счету | String(255)
extra_\*|Дополнительные данные счета (если были указаны при выставлении счета).|String(255)
version | Версия уведомлений | Number

<h3 class="request method">Ответ → POST</h3>
Ответ на запрос должен быть в формате JSON.

### Заголовки

*  `Content-type: application/json`

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{"error": 0}
~~~

<aside class="warning">
В ответе должен вернуться код результата обработки уведомления. Код результата должен находиться в параметре <i>error</i>.
Если в ответе код результата обработки уведомления отличается от 0 и/или код состояния HTTP отличается от 200 (OK), это интерпретируется как временная ошибка провайдера. Сервер повторяет запрос с нарастающим интервалом в течение суток (**не более 50 попыток**) до получения в ответе кода результата 0 и кода состояния HTTP 200.
</aside>


<aside class="notice">
<ul>
<li>Рекомендуется возвращать коды результата в соответствии с таблицей <a href="#notify_codes">кодов завершения</a>.</li>
<li>Если ответ с кодом результата 0 и кодом состояния HTTP 200 так и не был получен в указанное время, повторные уведомления от сервера Visa QIWI Wallet прекращаются, и на адрес электронной почты провайдера высылается письмо с новым статусом счета и указанием на возможную техническую неисправность в работе сервиса провайдера.</li>
<li>Для получения уведомлений провайдер должен принимать HTTP-запросы из следующих подсетей исключительно по портам 80, 443:</li>
<ul>
<li> 91.232.230.0/23</li>
<li> 79.142.16.0/20</li>
</ul>
</ul>
</aside>


## Авторизация уведомлений

Используется авторизация по цифровой подписи. В https-запросах необходимо также проверять серверный сертификат Visa QIWI Wallet.

Подпись уведомления отправляется в заголовке `X-Api-Signature`. Для формирования подписи используется механизм проверки целостности HMAC с хэш-функцией SHA256.

HMAC sha256 от части полей, с разделителем |.+secret_key
X-Api-Signature: J4WNfNZd***V5mv2w=

* В качестве разделителей параметров используется символ `|`.
* В подписи участвуют следующие параметры:
   * prv_id 
   * bill_id
   * amount
   * currency
   * status
   * error
   * email/phone/user_id (если были переданы)
* Параметры для подписи переводятся в байт-представление с UTF-8 и располагаются в алфавитном порядке.
* Ключ подписи равен secret_key из [параметров авторизации](#auth_param).

Алгоритм проверки подписи:

1. Получить строку, содержащую значения перечисленных параметров POST-запроса в алфавитном порядке перечисления параметров, разделенных символами `|`:

   `{parameter1}|{parameter2}|…`

   где `{parameter1}` – значение параметра уведомления. Все значения при проверке подписи должны трактоваться как строки.

2. Cтроку и ключ подписи преобразовать в байты с UTF-8.
3. Вычислить HMAC-хэш c шифрованием SHA256:

   `hash = HMAС(SHA256, secret_key_bytes, invoice_parameters_bytes)`
   Где:

   * `secret_key_bytes` – ключ функции (байт-представление секретного ключа);
   * `invoice_parameters_bytes` – байт-представление параметров POST-запроса для проверки подписи;
   * `hash` – результат хэш-функции.

4. HMAC-хэш преобразовать из строк в байты с использованием кодировки UTF-8 и base64-преобразовать.
5. Сравнить значение заголовка X-Api-Signature с результатом 4.

~~~php
<?php

function hexToStr($hex){
    $string='';
    for ($i=0; $i < strlen($hex)-1; $i+=2){
        $string .= chr(hexdec($hex[$i].$hex[$i+1]));
    }
    return $string;
}

//функция генерации подписи по ключу и строке параметров
function checkSign($key, $req){
    $sign_hash = hash_hmac("sha1", $req, $key);
    $sign_tr = hexToStr($sign_hash);
    $sign = base64_encode($sign_tr);
    return $sign;
}

//Функция возвращает упорядоченную строку значений параметров POST-запроса
function getReqParams(){
    $reqparams = "";
    ksort($_POST);
    foreach ($_POST as $param => $valuep) {
        $reqparams = "$reqparams|$valuep";
    }
    return substr($reqparams,1);
}

//Извлечение цифровой подписи из заголовков запроса
function getSign(){
    $HEADERS = getallheaders();
    foreach ($HEADERS as $header => $value) {
       if ($header == 'X-Api-Signature') {
            $SIGN_REQ = $value;
       }
    }
    return $SIGN_REQ;
}

// Сортировка параметров
$Request = getReqParams();
// Пароль ishop для уведомлений магазина
$NOTIFY_PWD = "***";
// Вычисляем подпись
$reqres = checkSign($NOTIFY_PWD, $Request);

// Подпись из запроса
$SIGN_REQ = getSign();

if ($reqres == $SIGN_REQ) {
    $error = 0;
}
else $error = 151;

//Ответ
header('Content-Type: text/xml');
$xmlres = <<<XML
<?xml version="1.0"?>
<result>
<result_code>$error</result_code>
</result>
XML;
echo $xmlres;
?>
~~~

### Пример реализации


## Коды уведомлений  {#notify_codes}

Код завершения|Описание
--------------|--------
0|Успех
5|Ошибка формата параметров запроса
13|Ошибка соединения с базой данных
150|Ошибка проверки пароля
151|Ошибка проверки подписи
300|Ошибка связи с сервером