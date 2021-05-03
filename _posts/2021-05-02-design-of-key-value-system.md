---
title: Проектируем key-value БД
author: Дмитрий Шупляков
date: 2022-05-02 23:30:00
tags:
---

## Key-value хранилище

Key-value хранилище - это нереляционная база данных, которая хранит данные в виде пар ключ-значение. В качестве ключа используется некоторый уникальный 
идентификатор, по которому осуществуется доступ к ассоциированным данным - документу. В качестве документа могут использоваться строки, json-документы, списки и т.д.

К этому классу систем относятся: Redis, Memcache, Amazon DymanoDB, Cassandra, Couchbase.

### Требования к key-value БД.

Перед проектированием key-value БД опишем требования, которым она должна удовлетворять:

- Небольшой размер key-value пары, 10 KB.
- Отсутствие лимита на количество хранимых документов.
- Высокая доступность (high availability). Система отвечает быстро, даже при отказе нод.
- Горизонтальная масштабируемость.
- Атоматическая масштабируемость. Добавление/удаление нод прозрачно и не требует ручных действий.
- Настраиваемая консистетность (configurable cosistancy level).
- Низкий latency.


<!-- more -->
### Распределенная БД
Наиболее простым и очевидным способом будет спроектировать БД на основе хеш-мапы, которая будет целиком располагаться на одном сервере. Однако такая система 
не сможет горизононтально масштабироваться, переживать отказы и обеспечивать high avaliability. Хотя и сможет обеспечить низкий latency и будет простав в реализации. 
Поэтому мы будем рассматривать распределенную архитектуру. 

Все распределенные система сталкиваются с проблемой CAP-теоремы. 

#### CAP-теорема
CAP-теорема утверждает, что всякая распределенная система удовлетворяет только 2-ум из 3-х свойств:
- Consistency - консистетность данных.
- Availability - доступность системы. Способность обрабатывать запросы.
- Partition Tolerance - устойчивость к сетевому расщеплению. Потеря связи между частями системы (нодами) называется сетевым разрывом.

Группы систем:
- *CA-системы* обеспечивают конситентность и доступность, но не переживают потерю связи между нодами. К этой группе относят реляционные базы данных (RDBS). Чаще всего несколько
серверов БД работают в режиме репликации, и при недоступности одного из узла, прекращают обработку запросов.
- *CP-системы* - поддерживают консистентность и устойчивость к сетевому расщеплению. В случае сетевого разрыва кластер перестает принимать входящие запросы, тем 
самым сохраняя консистентность. 
Так же есть некоторый компромисс, обеспечивающий частичную доступность. В нем, при обнаружении разрыва, разделяющего кластер на две группы изолированных нод, 
система инактивирует меньшую группу и позволяет работать большей группе. Образуется меньший кластер сохраняющий констистетность. В случае восстановления
 связи с нерабочей группой нод, она получает изменения данных, произошедшие за время ее бездействия, восстанавливая консистентность всех узлов. Такой алгоритм 
 используется, например, в Zookeeper.
- *AP-системы* - при сетевом разрыве система продолжает работать, но теряет консистентность. Когда кластер разделяется на две изолированные группы узлов, каждая
из них продолжает работать самостоятельно. При восстановлении связи, данные в кластере будут находиться в неконсистеность состоянии. 

Наша система должны обеспечивать настраиваемую конситетность, обеспечивая, в зависимости от требований либо CP, либо AP.

### Партиционирование данных
Учитывая, что проектируемая БД распределенная, то есть состоит из нескольких сервером, необходимо научиться распределять данные между этими узлами. Наиболее 
очевидный вариант распределять данные, взяв хеш от ключа, разделив на количество серверов и взяв остаток от деления:

`servers = [‘server1’, ‘server2’, ‘server3’]
server_for_key(key) = servers[hash(key) % servers.length]`

Однако в этом случае, при добавлении/удалении узла потребуется перераспределить большое количество данных. Например, при добавлении в кластер из 4-ех узлов одного
узла, 80% данных поменяют свой сервер. См. https://habr.com/ru/company/badoo/blog/352186/.

Чтобы избежать эту проблему, будем использовать подход согласованного хеширования (constistancy hashing). В нем, данные маппируется не сразу на сервера, а на 
виртуальные ноды, число которых превышает количество машин, заранее известно и неизменяемо. 

`servers = [‘server1’, ‘server2’, ‘server3’]
vbuckets = [0 .. 256]
server_for_key(key) = servers[vbuckets[hash(key) % vbuckets.length]]`

Используя эту функцию вычисляется виртуальная нода на которую должны попасть данные, а затем используется таблица со связями вирт. нод и 
серверов. При добавлении сервера в кластер, таблица перестраивается таким образом, чтобы каждый сервер поделился частью собственных виртуальных нод с новой машиной. 
Добавляя пятую ноду в кластер, каждый сервер отдаст только пятую часть своих данных, и суммарное перемещение данных в кластере будет составлять 20%.
Более подробно здесь: https://dshuplyakov.github.io/token_ring_and_vnodes/



