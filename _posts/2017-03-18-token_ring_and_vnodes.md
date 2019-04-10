---
title: Token ring и виртуальные нод
author: Дмитрий Шупляков
tags: Cassandra
date: 2017-03-18 22:21:10
---
Как распределяются данные между нодами. На какую ноду попадут данные при записи? К какой ноде обращаться при чтении? Как происходит ребалансинг?

<!-- more -->

## Token Ring
Распределение данных в кластере выполнено на основе _кольца_. Ноде назначается один или несколько диапазонов данных, каждый из которых имеет свой токен. Токено предсталяет собой число от -2^63 до 2^63.

Таким образом токены описывают полное кольцо данных. Для того, что понять в какую ноду вставлять новые данные (или читать существующие), Кассандра берет хеш-функцию от первичного ключа, получившееся значение является токеном, на основе которого она однозначано может идентицировать ноду и позицию где должны располагатся данные.

![cassandra-token-ring.png]({{ site.baseurl }}/images/7-chapter-token-ring.png)

## Виртуальные ноды (vnodes)
Виртуальные ноды или vnodes - способ улучшить перебалансировку данных между нодами. Если при перебалансировке опираться только на token ring, то при добавлении новой ноды потребуется переместить значительно больше данных, чем требуется. Это вызвано тем, что интервалы владения данными высчитыывают заново (с учетом новой ноды) и смещаются. Идея виртуальных нод заключаетя в том, чтобы добавить новый уровень абстракции между маппингом token ring на ноды. Этот слой содержит виртуальные ноды, добавляя дополнительную гранулярность данным. По умолчанию создается 256 таких виртуальных нод, которые распределяются между реальными нодами. При добавлении новой ноды в кластер, каждая существующая делится своими виртуальными нодами с новыми участником. Таким образом глобального перераспределения данных не происходит, что снижает пересылку данных по сети и ускоряет ввод новых нод. 

### Ссылки
1. https://www.datastax.com/dev/blog/virtual-nodes-in-cassandra-1-2