
### <a name="_b7urdng99y53"></a>**Название задачи:** передача ставок в колл-центр
### <a name="_hjk0fkfyohdk"></a>**Автор:**
### <a name="_uanumrh8zrui"></a>**Дата:**
### <a name="_3bfxc9a45514"></a>**Функциональные требования**

Для сотрудников внутреннего колл-центра можно обеспечить интеграцию с API модулем интернет-банка. 
Для внешнего колл-центра необходимо обеспечить выгрузку данных в файл

|**№**|**Действующие лица или системы**|**Use Case**|**Описание**|
| :-: | :- | :- | :- |
|UC1  | Сотрудник внутреннего колл-центра | получение данных о ставке | 1. сотрудник во внутренней системе вводит иформацю о клиенте <br> 2. Посредством API модуля интернет-банка система колл-центра получает  необходимые данные <br>| 
|||||

### <a name="_u8xz25hbrgql"></a>**Нефункциональные требования**
Опишите здесь нефункциональные требования и архитектурно значимые требования.

|**№**|**Требование**|
| :-: | :- |
| **R** | **ограничения** |
| | нет возможности произвести интеграцию внешнего колл-центра с системой. Возможна только передача выгрузки в файле в условленном формате |
### <a name="_qmphm5d6rvi3"></a>**Решение**

```plantuml
@startuml
title рис1. передаца данных в колл-центр - диаграмма контекста 
top to bottom direction

!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Component.puml

Person(opin, "Оператор внутреннего колл-центра", "")
Person_Ext(opex, "Оператор внешнего колл-центра", "")

System(dbank, "Api данных","модуль API интернет-банка или отдельный модуль")
System(callint, "Внутренний колл-центр")
System_Ext(callext, "Внешний колл-центр")

Rel(opin, callint,"")
Rel(opex, callext,"")

Rel(dbank, callint, "получение данных, как общих так и по определенному клиенту")

Rel(dbank, callext, "периодическая выгрузка данных для внешнего колл-центра")

@enduml
```

Функции интеграции можно возложить на контейнер для интернет-банка, т.к. задачи обеспечения доступа к необходимому набору данных и обеспечение надежности и быстродействия уже решен в данном модуле.


```plantuml
@startuml
title рис2. передача данных в колл-центр - диаграмма контейнера
top to bottom direction

!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Component.puml

Person_Ext(user, "Пользователь", "Клиент банка")
Person(callcoper, "Оператор колл-центра", "Внутренний сотрудник")

System(front_http, "nginx", "Фронтальный веб-сервер + маршрутизатор + балансир + HTTPS")
System(site, "Сайт", "PHP + ReactJS")

System(callcenter, "Call Center","Система внутреннего колл-центра")
Rel(callcoper, callcenter,"")

Container_Boundary(ibank_bnd, "Интернет-банк") {
    Container(ibnk_iis, "WebApp", "MS IIS", "Microsoft IIS со старым  ASP MVC приложением интернет-банка")

    ContainerDb(ibnk_db, "БД", "MS-SQL", "База данных Интернет-банка")

    Container(ibnk_web_cl, "WebClient", "Клиент нового веб-сайта, на которое будут переносится функции")
    Container(ibnk_web_srv, "Интернет Банк API", "Backend сервер интернет-банка")

    Rel(ibnk_iis, ibnk_db, "")
    Rel(ibnk_web_srv, ibnk_db, "")
    Rel(ibnk_web_cl, ibnk_web_srv, "")

}

ContainerQueue(qbroker, "Интеграционный брокер", "Kafka", "Интеграция между системами")

Person(oper, "Оператор", "Сотрудник бэк-офиса")

Container_Boundary(abs_bnd, "АБС") {
    Container(absapp, "АБС Клиент", "Delphi Client App", "Клиентской приложение оператора бэк-офиса")
    ContainerDb(abs_db, "БД", "OracleDb", "База данных АБС c логикой")
      
    Container(absapi, "AБС Api","","Сервер интеграции и обработки событий АБС")

    Rel(absapp, abs_db, "")
    Rel(absapi, abs_db, "")
}

Rel(callcenter,ibnk_web_srv,"запрос данных для кц")
Rel(ibnk_web_srv,to_ext_callcenter,"периодическая выгрузка данных для внешнего КЦ")

Rel(oper, absapp, "")

System(sms, "Шлюз телеком-оператора")

Rel(ibnk_web_srv, qbroker, "" )
Rel(absapi, qbroker, "" )

Rel(user, front_http, "соединение до внутреннего контура банка защищено https")
Rel(front_http, ibnk_iis, "")

Rel(front_http, site, "")
Rel(front_http, ibnk_web_cl, "")

Rel(qbroker, sms, "сообщения для отправки")

@enduml
```

### <a name="_bjrr7veeh80c"></a>**Альтернативы**

В качестве альтернативы можно рассмотреть создание отдельной системы, аналогичной подисистеме интернет-банка, обеспечивающии функционла доступа и выгрузки данных для коллцентров.

В такой системе колл-центр не будет зависить от работы интернет-банка, но с другой стороны создание и поддрежка такой службы станет дороже. При необходимости такое решение может быть рассмотрено для развития системы в будущем, если на КЦ понадобится возложить дополнительные задачи (оформление заявки по телефону, аналогично оформлению на сайте - интернет-банке).

Для частичного устранения зависимости колл-центра от работы интернет-магазина, а также для сокращения кол-ва внутренних обращений к API интернет-банка, внутренний колл-центр может такде ериодически импортировать в свою систему периодическую выгрузку данных для внешней системы.

Также можно предложить задачу выгрузки данных переложить на ситему АБС, как самый дешевый вариант решения.

```plantuml
@startuml
title рис3. альтернатива 1. передача данных в колл-центр - диаграмма контейнера
top to bottom direction

!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Component.puml

Person_Ext(user, "Пользователь", "Клиент банка")
Person(callcoper, "Оператор колл-центра", "Внутренний сотрудник")

System(front_http, "nginx", "Фронтальный веб-сервер + маршрутизатор + балансир + HTTPS")
System(site, "Сайт", "PHP + ReactJS. Самостоятельный модуль, содержит только статичные данные ")

Container_Boundary(ibank_bnd, "Интернет-банк") {
    Container(ibnk_iis, "WebApp", "MS IIS", "Microsoft IIS со старым  ASP MVC приложением интернет-банка")

    ContainerDb(ibnk_db, "БД", "MS-SQL", "База данных Интернет-банка")

    Container(ibnk_web_cl, "WebClient", "Клиент нового веб-сайта, на которое будут переносится функции")
    Container(ibnk_web_srv, "Интернет Банк API", "Backend сервер интернет-банка")

    Rel(ibnk_iis, ibnk_db, "")
    Rel(ibnk_web_srv, ibnk_db, "")
    Rel(ibnk_web_cl, ibnk_web_srv, "")

}

Container_Boundary(call_bnd, "Колл-центр") {

    System(callcenter, "Call Center","Система внутреннего колл-центра")
    ContainerDb(callc_db, "БД", "MS-SQL", "База данных Колл-центра")
    Container(callc_api, "CallCenter API", "Backend сервер call center")
    Container(callc_report, "CallCenter Report Service", "подсистема выгрузка данных для внешнего КЦ")

    Rel(callcenter, callc_api, "")
    Rel(callc_api, callc_db, "")
    Rel(callc_db, callc_report, "данные для выгрузки во внешний КЦ")

}

Rel(callcoper, callcenter,"")

ContainerQueue(qbroker, "Интеграционный брокер", "Kafka", "Интеграция между системами")

Person(oper, "Оператор", "Сотрудник бэк-офиса")

Container_Boundary(abs_bnd, "АБС") {
    Container(absapp, "АБС Клиент", "Delphi Client App", "Клиентской приложение оператора бэк-офиса")
    ContainerDb(abs_db, "БД", "OracleDb", "База данных АБС c логикой")
      
    Container(absapi, "AБС Api","","Сервер интеграции и обработки событий АБС")

    Rel(absapp, abs_db, "")
    Rel(absapi, abs_db, "")
}


Rel(oper, absapp, "")

System(sms, "Шлюз телеком-оператора")

Rel(ibnk_web_srv, qbroker, "" )
Rel(absapi, qbroker, "" )
Rel(callc_api, qbroker, "" )

Rel(user, front_http, "соединение до внутреннего контура банка защищено https")
Rel(front_http, ibnk_iis, "")

Rel(front_http, site, "")
Rel(front_http, ibnk_web_cl, "")

Rel(qbroker, sms, "сообщения для отправки")

@enduml
```

**Недостатки, ограничения, риски**

- работа коллцентра зависит от работы модуля интренет-банка. При полной недоступности интернет-банка колл-центр также не сможет функционировать
- экспорт данных для внешней системы производится периодически вручную

