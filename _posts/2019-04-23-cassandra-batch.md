---
title: Cassandra. Часть 6. Batches
author: Дмитрий Шупляков
date: 2019-04-23 22:32:39
tags:
---
В реляционных базах есть возможность объединения запросов в batch (группу). Обработка таких запросов выполняется быстрее, 
т.к. отсутствует затраты на передачу по сети множества маленьких отдельных запросов. В Кассандре тоже есть batch'и, но основное предназначение их несколько другое.

<!-- more -->

Начнем с того, что батчи бывают трех видов: LOGGED BATCH, UNLOGGED BATCH, COUNTER

## LOGGED BATCH 

Логгируемые батчи предназначены для обеспечения атомарности batch'a. То есть все запросы входящие в batch будут либо выполнены, либо нет.
На самом деле, если batch принят в работу, то откатиться он уже не сможет и будет гарантировано выполнен. Но ведь в момент выполнения 
батча нода может отказать, как же тогда достигается эта гарантия? 

Алгоритма  работы logged batch:
- из приложения сформированный batch отправляется на выполнение в С*
- batch приходит на ноду-координатор, координатор сохраняет его у себя
- координатор копирует batch на 2 любых ноды
- координатор начинает выполнение батча
- после завершения выполенния батча, кординатор шлет сообщение "batchlog delete message" резеврным нодам для удалення копий батчей

Если координатор упал во время выполнения батча, используется механизм восстановления: 
- каждая резервная нода выполняется каждые 10 секунд  сканируют свою таблицу batchlog в которой сохранена копия переданного batch'a. 
- если обнаруживается батч живущий дольше чем write_request_timeout*2 (по умолчанию, больше 4 сек), Кассандра выполняет это батч. 

Таким образом, предполагается, что в штатном режиме каждый батч успеет выполниться за 4 секунды и резервные ноды, получив сообщение 
"batchlog delete message", удалят у себя копии батча. Если же батч сообщение не пришло, это дает основания полагать, что с координатором проблемы, и значит необходимо выполнить батч за него. 

Особенности:

- Кассандра ограничивает размер батча, чтобы успевать уложиться в выделенный таймаут. 
- Нормальный считается батч размером до 5Кб, батчи размеро свыше 50Кб не выполняются. Параметры конфигурируются.

##UNLOGGED BATCH


##COUNTER

##Ссылки
1. https://stackoverflow.com/questions/28348717/how-do-atomic-batches-work-in-cassandra/28348718#28348718
2. https://github.com/apache/cassandra/blob/cassandra-3.11/src/java/org/apache/cassandra/batchlog/BatchlogManager.java