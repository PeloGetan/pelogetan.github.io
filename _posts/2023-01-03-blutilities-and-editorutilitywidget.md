---
layout: post
title: Blutilities и EditorUtilityWidget
published: true
---
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.1.png)  
И **Blutilities**, и **EditorUtilityWidget** выполняют одну и ту же задачу: экономят нам время. Они призваны автоматизировать ту работу, для которой мы можем составить алгоритм.
Например, покрасить каждое второе дерево на уровне в желтый, поставить лампу на каждую тумбочку, или что угодно еще, что мы можем, но не хотим делать самостоятельно.  
Глобальной разницы между **Blutility** и **EditorUtilityWidget** нет, почти всегда они могут заменить друг друга, но у них есть отличие: **Blutility** всегда нужно размещать на уровне, благодаря чему мы можем использовать его координаты в логике. **EditorUtilityWidget** размещать на уровне нельзя.

# Blutilities
**Blutility** или блютилити - это блюпринт, унаследованный от **AActor**, который имеет один, или несколько специальных ивентов, которые можно вызывать прямо в эдиторе.  
Как сделать:  
1 - Для начала создадим новый **BP Actor**.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.2.png)  
2 - Теперь добавим **Custom Event**.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.3.png)  
3 - Выбрав новый ивент, во вкладке **Details** назначим **Call In Editor - True**.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.4.png)  
4 - Добавим блюпринт на уровень.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.5.png)  
При выборе блюпринта, во вкладке **Details** появится кнопка с названием ивента.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.6.png)  
Блютилити готов и работает, но в настоящий момент ничего не делает.  
Я добавлю блюпринт **BP_Box**, который будет из себя представлять простой **Static Mesh** в виде белого куба.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.7.png)  
1 - В блютилити добавлю простейшую логику:  
  В начало ивента поставлю цикл **For Loop** на десять шагов, в теле цикла будет создаваться **BP_Box** и ему будет назначаться **Transfrom** с вычисленным **Location**. **Location** берет **Y** и **Z** координаты от блютилити, **X** получается умножением индекса цикла на 150 и сложением этого числа с **X** координатой блютилити.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.8.png)  
2 - Комплируем, сохраняем и пробуем запустить блютилити.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.9.png)  
Отлично, все работает.  

Но это можно легко и быстро сделать вручную, скажете вы и будете правы, поэтому вот следующий пример:

Допустим, нам нужно разместить 10 кубов не просто в пространстве, а обязательно положить их на наклонную поверхность с правильным углом. Звучит сложнее, да и времени займет больше, особенно, если количество кубов вырастет с 10 до 100 или 1000.

А теперь доработаем наше блютилити, чтобы оно делало эту работу за нас:  
1 - Добавляем новую логику в блюпринт:  
  После создания куба ставлю **LineTraceByChannel**, который будет делать трейс вниз на 10 метров. Ноде лайнтрейса важно добавить созданный куб в **ActorsToIgnore**, так как действие происходит не в кубе, а в блютилити, и куб, следовательно, игнорироваться не будет. Далее нода **Branch** проверяет, был ли положительным хит лайнтрейса и если да, то назначаем кубу координаты столкновения из **Hit Result**, угол получаем с помощью ноды **RotationFromXVector**, которой даем **Normal** так же из **Hit Result**.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.10.png)  
2 - Жмем кнопку, все работает, отлично!  
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.11.png)  

# EditorUtilityWidget
**EditorUtilityWidget** или ЭдиторЮтилитиВиджет - это блюпринт виджет, наследованный от **UEditorUtilityWidget**, который позволяет запускать его прямо в эдиторе и выполнять любую написанную в нем логику.  
Как сделать:  
1 - Для начала создадим новый **EditorUtilityWidget**.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.12.png)  
2 - Теперь добавим кнопку и текст, напишем что она будет делать.  
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.13.png)  
3 - Откроем **EventGraph** и добавим логику.  
  В начале ивента создам ноду **GetAllActorsOfClass**, которая будет искать все **BP_Box** на уровне. По найденным экторам пройдет цикл, внутри которого нода **FlipFlop** будет вызывать **SetMaterial** через раз, благодаря чему можно добиться изменения материала у каждого второго эктора. 
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.14.png)  
4 - Чтобы запустить виджет, нужно нажать на него **ПКМ** и выбрать "**Run Editor Utility Widget**"  
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.15.png)  
5 - Жмем кнопку и радуемся результату:
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.16.png)  

# Заключение
Используя **EditorUtilityWidget** и **Blutility**, можно создать удивительные и крайне полезные инструменты размером с целые плагины. Их функционал можно сильно расширить, если добавить код в C++, который позволит менять CDO блюпринтов, о чем я обязательно напишу в следующем посте.  
