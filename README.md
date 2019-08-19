# common catalogs
Сервис предназначен для создания и репликации элементов общих справочников в распределенной среде, в том числе, поверх mdm между разными абонентами и разными типами баз (couchdb, postgres, 1C и т.д.)
В сценарии "одна база 1С и несколько бесправных абонентов", необходимости в сервисе не возникает, но если справочники редактируют десятки пользователей из слабосвязанных организаций и делают это в физически разных 1С и через браузеры разных абонентов метадаты, сервис незаменим для разруливания конфликтов версий и дублирования элементов

### демо настройки
Стартовые настройки включают в себя справочник цветов для [Заказа дилера](https://github.com/oknosoft/windowbuilder), но сюда же, откровенно просятся контрагенты, договоры, классификатор банков и банковские счета

### суть
- Элементы общих справочников создаются в одном месте и разбегаются по базам-подписчикам
- Возможно кеширование общих данных в idb браузеров как в отдельной, так и общей базе ram
- Гибкая настройка прав на чтение-создание-изменение элементов
- Гибкие алгоритмы при создании элементов (например, запрос к внешним сервисам при создании контрагентов по ИНН или внешнему классификатору банков при создании счетов)
- Автономный режим не поддержан. В оффлайне доступна копия данных из idb ram или sdbl 1C, но для создания-изменения элементов справочника, требуется пожключение к сервису
- Администраторы и полноправные пользователи 1C и couchdb, могут нарушить целостность данных, если будут создавать элементы общих справочников в offline-репликах распределенной системы - проблему предлагается решать организационно - редактировать общие справочники через сервис

### как это работает
Сервис имеет визуальный и http интерфейсы
- Для авторизации через браузер, используется [auth-proxy](https://github.com/oknosoft/metadata-auth-proxy). Пользователь должен иметь учетную запись couchdb или ldap с ролями `doc_full` или `common_editor` (`полные права` либо `редактор общих данных`)
- Само хранилище общих данных - это база pouchdb в озу сервера со снапшотом в приватную couchd. Для внешних пользователей, сервис притворяется базой couchdb и предоставляет поток `_changes`, на который могут подписаться другие базы couchdb, базы 1C и базы pouchdb в браузерах.  
Поддержана как статическая выборка изменений, так и longpool для непрерывной репликации. Важным отличием от стандартной couchdb, является встроенный механизм фильтрации `_changes`. На потоки событий, отдаваемых пользователям разных абонентов и разных отделов абонентов могут накладываться фильтры с учетом настроек, заданных в справочнике `branches` или иным гибким способом
- Сервис общих данных ничего не знает о структуре своих подписчиков и не пытается из центра синхронизировать данные в десятке баз. Он выступает единственным источником правды про общие данные и предосталяет возможность подписаться на поток этих данных всем жедающим по стандартному протоколу couchdb
- Кроме потока `_changes`, сервис может иметь произвольное число специализированных endpoints для выполнения операций с данными. Например, создания контрагента по ИНН или создания составного цвета по комбинации цветов. Состав и права на эти endpoints, настраиваются индивидуально в зависимости от прикладного решения  

