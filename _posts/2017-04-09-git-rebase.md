---
title: Шпаргалки Git rebase
date: 2017-04-09 11:39:18
tags: git
---
**Rebase** - одна и самых известных и в то же время недооценненых команд в гите. А ведь это та команда, которая позволяет управлять коммитами легко и удобно, насколько это, конечно, возможно в консоле. При этом на освоение команды достаточно 20-30 минут. 

## Описание работы команды
Формат команды:
```
git rebase [-i | --interactive] [options] [--exec <cmd>] [--onto <newbase>]	
    [<upstream> [<branch>]]

git rebase [-i | --interactive] [options] [--exec <cmd>] [--onto <newbase>]
	--root [<branch>]
```

Кратко сформулировать задачу команды можно следующим образом: **rebase** выполняет перенос набора коммитов в конец ветки. В том как она это делает есть ряд особенностей: 

- коммиты переносятся ТОЛЬКО в конец ветки;
- изменения полученные после ребейза отобразятся в текущей ветке (т.е. модифицируется только текущая ветка);
- можно ребейзить только диапазон коммитов, который находится в текущей ветки (в этом случаем кусок указыватся диапазоном двух коммитов MW-1572~1 MW-1572, опять же в той ветки, в которой мы находимся) Почему так? Потому что верхний диапазон коммитов, используется для предварительного checkout;
- этот кусок мы можем пересадить кому угодно (параметро --onto), т.е. в любую ветку;
- коммиты можно пересаживать даже в свою ветки (чаще всего это используется для редактирования коммитов)
- есть вот такой интересный коммент в доке, касающийся последнего параметра: If <branch> is specified, git rebase will perform an automatic git checkout <branch> before doing anything else. Otherwise it remains on the current branch;
- ОСНОВНОЕ ПРАВИЛО, ЧТО КОММИТЫ КОТОРЫЕ будут переноситься - НАХОДЯТСЯ В текущей ветке, либо в ветке которая указана последним параметром (и которая при выполнении ребейза зачекаучится);

    Наглядно работа команды выглядит следующим образом (__git rebase feature/JIRA-123 --onto master__)
![](/img/git-rebase-process.gif)
Иллюстрация взята с _https://hackernoon.com/git-in-2016-fad96ae22a15_

## ПРИМЕРЫ

1. Неважно в какой мы ветке.
```
git rebase --onto MW-1655 new-feature~4 new-feature -i
```
     Четыре последних коммита из **new-feature** будут перенесены в конец ветки **MW-1655** и результат сохранится в **new-feature**.

2. _Мы в new-feature._
```
 git rebase --onto MW-1655 new-feature~4 -i
```
     Результат будет тот же. Т.к. мы вытаскиваем последние 4 коммита, то последний параметр можно опустить.

3. _Мы в feature._
```
git rebase master
```
     Взять за основую **master**, и заребейзить поверх **feature**. Изменения в ветке **feature**.

4. _Мы в feature_
```
git rebase master feature
```
     То же самое, что и выше.

5. _Неважно в какой мы ветке._
```
git rebase --onto develop MW-1572~1 MW-1572 -i
```
- сделать чекаут ветки **MW-1572**
- применить последний коммит из ветки **MW-1572** в конец develop
- результат поместить в **MW-1572**

6. _Мы в 2.8.2.3_
```
git rebase -i  --onto develop 2.8.2.3
```
     очень похожа на предыдущую команду.

- взять все коммиты с **2.8.2.3** и помещает их поверх **develop**
- естественно должен быть общий предок между **2.8.2.3** и **develop** чтобы понятно было какие коммиты брать
     
## Интерактивный режим команды

Включается флагом -i и позволяет выполнить дополнительные действия над пересаживаемым набором коммитов. 

Список действий:
```
p, pick = use commit
r, reword = use commit, but edit the commit message
e, edit = use commit, but stop for amending
s, squash = use commit, but meld into previous commit
f, fixup = like "squash", but discard this commit's log message
x, exec = run command (the rest of the line) using shell
d, drop = remove commit
```

## Ссылки:
https://hackernoon.com/git-in-2016-fad96ae22a15



Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
