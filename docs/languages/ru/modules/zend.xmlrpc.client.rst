.. EN-Revision: none
.. _zend.xmlrpc.client:

Zend\XmlRpc\Client
==================

.. _zend.xmlrpc.client.introduction:

Введение
--------

Zend Framework поддерживает клиентское использование удаленных XML-RPC
сервисов через пакет ``Zend\XmlRpc\Client``. Его основные возможности
включают в себя автоматическое преобразование типов между PHP и
XML-RPC, прокси-объект сервера и доступ к средствам интроспекции
на сервере.

.. _zend.xmlrpc.client.method-calls:

Вызов методов
-------------

Конструктор ``Zend\XmlRpc\Client`` принимает URL удаленного XML-RPC сервера в
качестве первого параметра. Новый экземпляр класса может
использоваться для вызова любых удаленных методов этого
сервера.

Для вызова удаленного метода через клиентa XML-RPC инстанцируйте
его и используйте его метод *call()*. В примере ниже используется
демонстрационный XML-RPC сервер на веб-сайте Zend Framework. Вы можете
использовать его для тестирования или изучения компонент
``Zend_XmlRpc``.

.. _zend.xmlrpc.client.method-calls.example-1:

.. rubric:: Вызов метода XML-RPC

.. code-block:: php
   :linenos:

   $client = new Zend\XmlRpc\Client('http://framework.zend.com/xmlrpc');

   echo $client->call('test.sayHello');

   // hello

Значение XML-RPC, возвращаемое при вызове удаленного метода,
будет автоматически приведено к эквивалентному типу в PHP. В
примере выше возвращается строка (тип ``String`` в PHP), и она уже
готова к применению.

Первый параметр метода *call()* принимает имя удаленного метода,
вызов которого требуется. Если удаленный метод требует
каких-либо параметров, то они могут быть переданы методу *call()*
через второй необязательный параметр в виде массива значений
для последующей передачи удаленному методу:

.. _zend.xmlrpc.client.method-calls.example-2:

.. rubric:: Вызов метода XML-RPC с параметрами

.. code-block:: php
   :linenos:

   $client = new Zend\XmlRpc\Client('http://framework.zend.com/xmlrpc');

   $arg1 = 1.1;
   $arg2 = 'foo';

   $result = $client->call('test.sayHello', array($arg1, $arg2));

   // возвращаемый результат имеет "родной" для PHP тип

Если удаленный метод не требует параметров, то этот
необязательный параметр можно опустить или передать пустой
массив. Массив параметров для удаленного метода может
содержать значения "родного" для PHP типа, объекты ``Zend\XmlRpc\Value``,
либо и то и другое вместе.

Метод *call()* будет автоматически преобразовывать ответ XML-RPC и
возвращать его в эквивалентном "родном" для PHP типе. Кроме
этого, можно получить объект ``Zend\XmlRpc\Response`` для возвращенного
значения, вызвав метод *getLastResponse()* после вызова *call()*.

.. _zend.xmlrpc.value.parameters:

Типы и их преобразование
------------------------

Некоторые удаленные методы требуют передачи параметров при
вызове. Они передаются методу *call()* объекта ``Zend\XmlRpc\Client`` в виде
массива во втором параметре. Любой параметр может быть передан
в "родном" для PHP типе, который будет автоматически
преобразован в соответствующий тип XML-RPC, или как объект,
представляющий определенный тип в XML-RPC (один из объектов
``Zend\XmlRpc\Value``).

.. _zend.xmlrpc.value.parameters.php-native:

Параметры в "родном" для PHP типе
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Параметры могут передаваться методу *call()* как переменные
"родного" для PHP типа, это могут быть типы ``String``, *integer*, *float*,
``Boolean``, ``Array`` или *object*. В этом случае каждый из этих типов будет
автоматически определен и преобразован в один из типов XML-RPC
согласно следующей таблице:

.. table:: Преобразование типов PHP и XML-RPC

   +---------------------+-------------+
   |Тип в PHP            |Тип в XML-RPC|
   +=====================+=============+
   |integer              |int          |
   +---------------------+-------------+
   |double               |double       |
   +---------------------+-------------+
   |boolean              |boolean      |
   +---------------------+-------------+
   |string               |string       |
   +---------------------+-------------+
   |array                |array        |
   +---------------------+-------------+
   |array (ассоциативный)|struct       |
   +---------------------+-------------+
   |object               |array        |
   +---------------------+-------------+

.. note::

   **Какому типу будет соответствовать пустой массив?**

   Передача пустого массива методу XML-RPC несет в себе
   потенциальную проблему, т.к. он может быть представлен и
   массивом, и структурой. ``Zend\XmlRpc\Client`` в этом случае делает
   запрос к методу сервера *system.methodSignature* для определения
   требуемого типа аргумента и производит соответствующее
   преобразование.

   Но такое решение само по себе тоже может быть источником
   проблем. Во-первых, сервера, которые не поддерживают метод
   *system.methodSignature*, будут журналировать это как ошибочные вызовы,
   в этом случае ``Zend\XmlRpc\Client`` будет производить преобразование
   значения к типу array в XML-RPC. Кроме того, это приводит к
   дополнительным вызовам к удаленному серверу в случае
   передачи аргументов в виде массивов.

   Для того, чтобы полностью отключить эти вызовы, вы можете
   вызвать метод *setSkipSystemLookup()* до собственно запроса к методу
   XML-RPC:

   .. code-block:: php
      :linenos:

      $client->setSkipSystemLookup(true);
      $result = $client->call('foo.bar', array(array()));

.. _zend.xmlrpc.value.parameters.xmlrpc-value:

Параметры в виде объектов Zend\XmlRpc\Value
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Параметры могут также создаваться как экземпляры ``Zend\XmlRpc\Value``
для точного указания типа XML-RPC. Основные причины для этого:



   - Вы хотите быть уверенными в том, что процедуре передается
     корректный тип параметра (т.е. процедура требует
     целочисленное значение, а вы можете получать его из БД в
     виде строки)

   - Удаленная процедура требует тип *base64* или *dateTime.iso8601* (которых
     нет среди "родных" для PHP типов).

   - Автоматическое преобразование может работать неправильно
     (например, вы хотите передать пустую структуру XML-RPC в
     качестве параметра. Пустая структура представляется в PHP
     пустым массивом, но если вы передаете пустой массив в
     качестве параметра, то он преобразовывается в массив XML-RPC,
     т.к. не является ассоциативным массивом)



Есть два способа создания объектов ``Zend\XmlRpc\Value`` ―
непосредственное инстанцирование одного из подклассов
``Zend\XmlRpc\Value`` и использование статического фабричного метода
``Zend\XmlRpc\Value::getXmlRpcValue()``.

.. _zend.xmlrpc.value.parameters.xmlrpc-value.table-1:

.. table:: Объекты Zend\XmlRpc\Value для типов XML-RPC

   +----------------+---------------------------------------+--------------------------+
   |Тип XML-RPC     |Константа Zend\XmlRpc\Value            |Объект Zend\XmlRpc\Value  |
   +================+=======================================+==========================+
   |int             |Zend\XmlRpc\Value::XMLRPC_TYPE_INTEGER |Zend\XmlRpc\Value\Integer |
   +----------------+---------------------------------------+--------------------------+
   |double          |Zend\XmlRpc\Value::XMLRPC_TYPE_DOUBLE  |Zend\XmlRpc\Value\Double  |
   +----------------+---------------------------------------+--------------------------+
   |boolean         |Zend\XmlRpc\Value::XMLRPC_TYPE_BOOLEAN |Zend\XmlRpc\Value\Boolean |
   +----------------+---------------------------------------+--------------------------+
   |string          |Zend\XmlRpc\Value::XMLRPC_TYPE_STRING  |Zend\XmlRpc\Value\String  |
   +----------------+---------------------------------------+--------------------------+
   |base64          |Zend\XmlRpc\Value::XMLRPC_TYPE_BASE64  |Zend\XmlRpc\Value\Base64  |
   +----------------+---------------------------------------+--------------------------+
   |dateTime.iso8601|Zend\XmlRpc\Value::XMLRPC_TYPE_DATETIME|Zend\XmlRpc\Value\DateTime|
   +----------------+---------------------------------------+--------------------------+
   |array           |Zend\XmlRpc\Value::XMLRPC_TYPE_ARRAY   |Zend\XmlRpc\Value\Array   |
   +----------------+---------------------------------------+--------------------------+
   |struct          |Zend\XmlRpc\Value::XMLRPC_TYPE_STRUCT  |Zend\XmlRpc\Value\Struct  |
   +----------------+---------------------------------------+--------------------------+

.. note::

   **Автоматическое преобразование**

   Когда создается новый объект ``Zend\XmlRpc\Value``, его значение
   устанавливается в "родном" для PHP типе. Тип в PHP будет
   преобразован к определенному типу средствами PHP. Например,
   если в качестве значения для объекта ``Zend\XmlRpc\Value\Integer`` была
   передана строка, то она будет преобразована через *(int) $value*.

.. _zend.xmlrpc.client.requests-and-responses:

Прокси-объект сервера
---------------------

Другим способом вызова удаленных методов через клиента XML-RPC
является использование "заместителя" сервера. Это PHP-объект,
который предоставляет интерфейс к удаленному пространству
имен XML-RPC, делая работу с ним максимально близкой к работе с
обычным объектом в PHP.

Для того, чтобы инстанцировать "заместителя" сервера, вызовите
метод *getProxy()* объекта ``Zend\XmlRpc\Client``. Он вернет объект класса
``Zend\XmlRpc\Client\ServerProxy``. Любые вызовы методов прокси-объекта
сервера будет перенаправлены к удаленному серверу, параметры
могут передаваться так же, как и для любых других методов в PHP.

.. _zend.xmlrpc.client.requests-and-responses.example-1:

.. rubric:: Прокси-объект к пространству имен по умолчанию

.. code-block:: php
   :linenos:

   $client = new Zend\XmlRpc\Client('http://framework.zend.com/xmlrpc');

   // Создание прокси-объекта к пространству имен по умолчанию
   $server = $client->getProxy();

   $hello = $server->test->sayHello(1, 2);
   // test.Hello(1, 2) возвращает "hello"

Метод *getProxy()* принимает необязательный аргумент, указывающий,
к какому пространству имен следует создать прокси-объект. Если
этот аргумент не был указан, то то будет использоваться
пространство имен по умолчанию. В следующем примере
используется пространство имен *test*:

.. _zend.xmlrpc.client.requests-and-responses.example-2:

.. rubric:: Прокси-объект к произвольному пространству имен

.. code-block:: php
   :linenos:

   $client = new Zend\XmlRpc\Client('http://framework.zend.com/xmlrpc');

   // Создание прокси-объекта к пространству имен "test"
   $test  = $client->getProxy('test');

   $hello = $test->sayHello(1, 2);
   // test.Hello(1,2) возвращает "hello"

Если удаленный сервер поддерживает сколько угодно вложенные
пространства имен, то они также могут использоваться через
прокси-объект сервера. Например, если сервер в примере выше
имеет метод *test.foo.bar()*, то он может вызываться следующим
образом: ``$test->foo->bar()``.

.. _zend.xmlrpc.client.error-handling:

Обработка ошибок
----------------

При вызове методов XML-RPC могут могут быть ошибки двух типов: HTTP и
XML-RPC. ``Zend\XmlRpc\Client`` распознает оба типа, позволяя обнаруживать и
отлавливать их независимо друг от друга.

.. _zend.xmlrpc.client.error-handling.http:

Ошибки HTTP
^^^^^^^^^^^

Если произошла ошибка HTTP - например, удаленный HTTP-сервер вернул
код *404 Not Found*, - то будет сгенерировано исключение
``Zend\XmlRpc\Client\HttpException``.

.. _zend.xmlrpc.client.error-handling.http.example-1:

.. rubric:: Обработка ошибок HTTP

.. code-block:: php
   :linenos:

   $client = new Zend\XmlRpc\Client('http://foo/404');

   try {

       $client->call('bar', array($arg1, $arg2));

   } catch (Zend\XmlRpc\HttpException $e) {

       // $e->getCode() возвращает 404
       // $e->getMessage() возвращает "Not Found"

   }

Независимо от того, какой клиент XML-RPC используется, всякий раз,
когда происходит ошибка HTTP, генерируется исключение
``Zend\XmlRpc\Client\HttpException``.

.. _zend.xmlrpc.client.error-handling.faults:

Ошибки XML-RPC
^^^^^^^^^^^^^^

Ошибка XML-RPC аналогична исключению в PHP. Это специальный тип,
возвращаемый при вызове метода XML-RPC и включающий в себя код и
сообщение ошибки. Ошибки XML-RPC обрабатываются по-разному, в
зависимости от контекста использования ``Zend\XmlRpc\Client``.

Если используется метод *call()* или прокси-объект сервера, то
ошибка XML-RPC приведет к тому, что будет сгенерировано
исключение ``Zend\XmlRpc\Client\FaultException``. Код и сообщение исключения
будут в точности соответствовать значениям в возвращенном
ответе с сообщением об ошибке.

.. _zend.xmlrpc.client.error-handling.faults.example-1:

.. rubric:: Обработка ошибок XML-RPC

.. code-block:: php
   :linenos:

   $client = new Zend\XmlRpc\Client('http://framework.zend.com/xmlrpc');

   try {

       $client->call('badMethod');

   } catch (Zend\XmlRpc\FaultException $e) {

       // $e->getCode() возвращает 1
       // $e->getMessage() возвращает "Unknown method"

   }

Если для выполнения запроса используется метод *call()*, то в
случае ошибки будет сгенерировано исключение
``Zend\XmlRpc\FaultException``. Объект ``Zend\XmlRpc\Response``, содержащий
возвращенную ошибку, можно также получить через метод
*getLastResponse()*.

Если для выполнения запроса используется метод *doRequest()*, то
исключение не генерируется. Вместо этого будет возвращен
объект ``Zend\XmlRpc\Response``, содержащий возвращенную XML-RPC ошибку.
Проверить, содержит ли объект ошибку, можно через метод *isFault()*
объекта ``Zend\XmlRpc\Response``.

.. _zend.xmlrpc.client.introspection:

Интроспекция сервера
--------------------

Некоторые XML-RPC сервера поддерживают интроспекцию методов под
пространством имен *system.*. ``Zend\XmlRpc\Client`` предоставляет
специальную поддержку для серверов с этой возможностью.

Экземпляр ``Zend\XmlRpc\Client\ServerIntrospection`` может быть получен через
вызов метода *getIntrospector()* класса ``Zend_XmlRpcClient``. Далее он может
использоваться для выполнения операций интроспекции на
сервере.

.. _zend.xmlrpc.client.request-to-response:

От запроса к ответу
-------------------

Метод *call()* экземпляра ``Zend\XmlRpc\Client`` в процессе выполнения
строит объект запроса (``Zend\XmlRpc\Request``) и передает его другому
методу *doRequest()*, который возвращает объект ответа
(``Zend\XmlRpc\Response``).

Метод *doRequest()* также доступен для непосредственного
использования:

.. _zend.xmlrpc.client.request-to-response.example-1:

.. rubric:: Выполнение запроса

.. code-block:: php
   :linenos:

   $client = new Zend\XmlRpc\Client('http://framework.zend.com/xmlrpc');

   $request = new Zend\XmlRpc\Request();
   $request->setMethod('test.sayHello');
   $request->setParams(array('foo', 'bar'));

   $client->doRequest($request);

   // $server->getLastRequest() возвращает экземпляр Zend\XmlRpc\Request
   // $server->getLastResponse() возвращает экземпляр Zend\XmlRpc\Response

После того, как через клиента был вызван метод XML-RPC (через
методы *call()*, *doRequest()* или через прокси-объект сервера), всегда
можно получить объекты последнего запроса и ответа на него
через методы *getLastRequest()* и *getLastResponse()* соответственно.

.. _zend.xmlrpc.client.http-client:

HTTP-клиент и тестирование
--------------------------

Ни в одном из предыдущих примеров не указывался HTTP-клиент. В
этом случае создается новый экземпляр ``Zend\Http\Client`` с
настройками по умолчанию и автоматически используется
клиентом ``Zend\XmlRpc\Client``.

HTTP-клиент может быть получен в любое время через метод
*getHttpClient()*. В большинстве случаев достаточно использование
HTTP-клиента по умолчанию. Тем не менее, метод *setHttpClient()*
позволяет установить HTTP-клиент, отличный от принятого по
умолчанию.

*setHttpClient()* может быть полезен при unit-тестировании. При
совместном использовании с ``Zend\Http\Client\Adapter\Test`` можно
имитировать удаленные сервисы для тестирования. В качестве
примера реализации рассмотрите unit-тесты для ``Zend\XmlRpc\Client``,
входящие в поставку Zend Framework.


