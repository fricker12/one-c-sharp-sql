Использование таблиц SQL Server в качестве очередей сообщений

Возможности.
1. Очереди сообщений типа 'FIFO', 'LIFO', 'HEAP', 'TIME', 'FILE'.
2. Режим конкурентного доступа к очередям 'S' (single) и 'M' (multiple).

API базы данных:
1. Хранимые процедуры:
- sp_create_queue @name, @type, @mode
- sp_delete_queue @name
- sp_produce_message @queue_name, @message_body, @message_type, @consume_time
- sp_consume_message @queue_name, @number_of_messages
2. Скалярные функции:
- fn_queue_exists @name returns int (0 = false, 1 = true)
- fn_is_name_valid @name returns bit (служебная функция)

Установка.
1. Запустить скрипт install-database.sql на SQL Server.
Будет создана база данных one-c-sharp-table-queues

Использование.
- Можно посмотреть как использовать в файле test-produce-consume-message.sql
- Можно воспользоваться обработкой 1С OneCSharpTableQueues.epf

Дополнительная информация.

https://infostart.ru/public/1214312/

Тестирование производительности.

- CPU: Intel Core i5-4460 3.20 GHz
- RAM: 8 Gb
- HDD: 7200 rpm (при копировании больших файлов показывает в среднем 60 Mb/s

Тестовый файл сообщения: 120 Kb (json = СериализаторXDTO.ЗаписатьJSON).

Для получения тестового файла использовался документ "УстановкаЦенНоменклатуры", имеющий 100 позиций, + подчинённые ему записи регистра сведений "ЦеныНоменклатуры". Типовая конфигурация "Управление торговлей" 11.х

Количество сообщений: 10 000 * 120 Kb = 1,14 Gb

- Отправка сообщений из 1С: 50 сек. = 200 msg/s = 23 Mb/s = 20 000 позиций цен / сек.
- Получение всех сообщений из 1С по 1 шт. за раз: 50 сек. = 200 msg/s = 23 Mb/s = 20 000 позиций цен / сек.
- Получение всех сообщений из 1С по 10 шт. за раз: 25 сек. = 400 msg/s = 46 Mb/s = 40 000 позиций цен / сек.
- Получение всех сообщений из 1С по 100 шт. за раз: 20 сек. = 500 msg/s = 58 Mb/s = 50 000 позиций цен / сек.

Сообщения только получались и преобразовывались в массив текстовых сообщений в памяти.

Сообщения пересылались / получались как nvarchar, то есть 2 байта на символ. Можно сделать в varchar - объём будет в 2 раза меньше. Использование типа данных varbinary затруднено из-за особенностей работы с бинарными данными 1С - очень медленно конвертируется тип данных 1С COMSafeArray в ДвоичныеДанные или Строку.

SQL Server, сервер 1С и клиент 1С (обработка) находились на одной машине (сетевые издержки не учитывались).

Disclaimer.
- Технически данная версия поддерживает работу с очередями типа 'FIFO', 'LIFO', 'HEAP', 'TIME'.
  Тип очереди 'FILE' находится в разработке (для передачи больших файлов).
- Практически в обработке 1С реализована только работа с типом очереди 'FIFO' с режимом конкурентного доступа 'S'.
