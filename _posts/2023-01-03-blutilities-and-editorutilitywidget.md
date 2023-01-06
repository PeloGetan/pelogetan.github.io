---
layout: post
title: Blutilities и EditorUtilityWidget
published: false
---
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.1.png)  
И **Blutilities**, и **EditorUtilityWidget** выполняют одну и ту же задачу, экономят нам время. Они призываны автоматизировать ту работу, которой мы можем составить алгоритм.
Например, покрасить каждое второе дерево на уровне в желтый, поставить лампу на каждую тумбочку, или что угодно еще, что мы можем, но не хотим делать самостоятельно.

# Blutilities
**Blutility** или блютилити - это блюпринт наследованный от **AActor**, который имеет одну, или несколько, специальных ивентов, которые можно вызывать прямо в эдиторе.  
Как сделать:  
1 - Для начала создадим новый **BP Actor**.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.2.png)  
2 - Теперь добавим **Custom Event**.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.3.png)  
3 - Выбрав новый ивент, во вкладке **Details** назначить **Call In Editor - True**.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.4.png)  
4 - Добавим блюпринт на уровень.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.5.png)  
При выборе блюпринта, во вкладке **Details** появится кнопка с названием ивента.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.6.png)  
Блютилити создан и работает, но в настоящий момент ничего не делает.  
Я создам блюпринт **BP_Box**, который будет из себя представлять простой **Static Mesh** в виде белого куба.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.7.png)  
В блютилити добавлю простейшую логику, которая будет создавать десять **BP_Box** друг за другом.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.8.png)  
Комплируем, сохраняем и пробуем запустить блютилити.
![]({{site.baseurl}}/images/2023-01-03-blutilities-and-editorutilitywidget/2023-01-03-blutilities-and-editorutilitywidget.9.png)  
Отлично, все работает.  

Но это можно легко и быстро сделать вручную, скажите вы и будете правы, поэтому вот следующий пример.  
Допустим, нам нужно разместить 10 кубов не просто в пространстве, а обязательно положить их на наклонную поверхность с правильным углом.  
Звучит сложнее, да и времени займет больше, особенно, если количество кубов вырастет с 10 до 100 или 1000.  
А теперь доработаем наше блютилити, чтобы оно делало эту работу за нас:  
1 - Добавляем наклонную поверхность.  
2 - Добавляем новую логику в блюпринт.  
Жмем кнопку, все работает, отлично!  

# EditorUtilityWidget
**EditorUtilityWidget** или ЭдиторЮтилитиВиджет - это блюпринт виджет, наследованный от **UEditorUtilityWidget**, который позволяет запускать его прямо в эдиторе и выполнять любую написанную в нем логику.  
Как сделать:  
1 - Для начала создадим новый **EditorUtilityWidget**.  
2 - Теперь добавим кнопку и текст, напишем что она будет делать.  
3 - Откроем **EventGraph** и добавим логику.  
В моем случае, нажатие кнопки приведет к изменению материала всех BP_Box на материал травы.  
Чтобы запустить виджет, нужно нажать на него **ПКМ** и выбрать "**Run Editor Utility Widget**"  

# Заключение
Используя **EditorUtilityWidget** и **Blutility**, можно создать удивительные и крайне полезные инструменты размером с целые плагины.  
Я не нашел разницы между **Blutility** и **EditorUtilityWidget** кроме того, что один размещается на уровне и запускается через **Details** и другой открывается в отдельно окне.  
Свое предпочтение отдаю **EditorUtilityWidget**, так как можно делать красивые виджеты с наборами разных функций и не нужно каждый раз размещать на уровне специальный блюпринт.
