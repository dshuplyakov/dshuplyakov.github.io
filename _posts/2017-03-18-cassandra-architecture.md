---
title: Архитектура Cassandra
author: Дмитрий Шупляков
tags: Cassandra
date: 2017-03-18 22:21:10
---
Архитектура - фундаментальные принципы и свойства системы воплощенные в ее элементах, связях, принципах дизайна и эволюции.

<!-- more -->

## Дата-центры и стойки
При хранении данных Кассандра оперируется двумя понятиями: Дата-центр (ДЦ) и стойка. Стойка - это логический набор нод Кассандры физически распложенных близко к друг другу. ДЦ - это логический набор стоек в одном здании соединенных между собой надежной сетью.

По умолчанию Кассандра считает что все ноды находятся в одном ДЦ и в одной стойке.

Кассандра использует информацию о топологии кластера, чтобы определить где хранить данные и как эффективнее выполнять запросы. Так, например, Кассандра пытается хранить реплики в разных ДЦ для увеличения доступности и устойчивости к расщеплению (partition tolerance) и выполняет запросы в пределах одного ДЦ для увеличения производительности.

## Snitches
Задача снитча определить близость узла друг к другу в кластере. Эта информация позволяет определить с каких нод предпочительнее читать данные и на какие записывать. Снитчи собирают информацию о топологии кластера, позволяя лучше маршрутизировать запросы. По сути снитч выясняет где располагается нода относительно других.

Например, рассмотрим ситуацию как снитч используется в операции чтения. Когда Кассандра выполняет чтение, она должна выполнить запросы к нескольких нодам (ко скольки определяется уровнем согласованности - consistency level). Для того, чтобы обеспечить максимальную скорость чтения Кассандра выполняет запрос к одной ноде и дополнительно получает хеш с других, чтобы удостовериться в консистентности данных. Задача снитча в этом случае - определить самую быструю реплику, чтобы именно к ней был отправлен запрос.

Снитч, который используется по умолчанию - SimpleSnitch ничего не знает ни о стойках, ни о Дата-центрах, поэтому он не подходит для использования в кластерах с несколькими ДЦ. Для этого в Кассандре использует другие снитчи, специально разработанные для таких сервисов как Amazon EC2, Googgle Cloud и Apache Cloudstack.

Помимо этих снитчей, в которых статически описана топология кластера, есть еще один снитч - DynamicEndpointSnitch. Этот снитч отслеживает производительность запросов к нодам, учитывая дополнительные параметры, например на каких нодах выполняется compaction. Такая информация позволяет выбрать лучшую реплику для каждого запроса исключив непроизводительные. В этой стратегии нодам, демонстрирующим высокую скорось назаначается более высокий приоритет. Однако периодически эта информация обнуляется, давая возможность  низкоприоритетным нодам  продемострировать восстановление производительности.

## Токены
Распределение данных в кластере выполнено на основе _кольца_. Ноде назначается один или несколько диапазонов данных, каждый из которых имеет свой токен. Токено предсталяет собой число от -2^63 до 2^63.

Таким образом токены описывают полное кольцо данных. Для того, что понять в какую ноду вставлять новые данные (или читать существующие), Кассандра берет хеш-функцию от первичного ключа, получившееся значение является токеном, на основе которого она однозначано может идентицировать ноду и позицию где должны располагатся данные.

![cassandra-token-ring.png]({{ site.baseurl }}/images/7-chapter-token-ring.png)

### Ссылки
1. https://www.amazon.com/Cassandra-Definitive-Guide-Distributed-Scale/dp/1491933666