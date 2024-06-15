---
layout: post
title: UE Slate урок 1
published: false
---
![]({{site.baseurl}}/images/2023-01-20-change-CDO-by-blueprint/2023-01-20-change-CDO-by-blueprint.prewiev.png)  
К моему удивлению, на просторах интернета практически нет уроков по слейту, а те что есть объясняют только самые основы. Пришло время это исправить. Я постараюсь написать несколько уроков, в которых затрону основные вопросы и проблемы связанные со слейтом.  
**Сразу замечу, что вы должны иметь хотя бы минимальный опыт работы с блюпринтовыми виджетами, иначе вам будет гораздо сложнее в изучении слейта.**

# Slate
**Slate** - это фреймворк анрила для создания пользовательского интерфейса. Все что вы видете на экране при запуске анрила, это слейт. Все меню, все кнопки и тд. все работает на слейте.  
Блюпринтовые виджеты тоже работают на слейте, и любой добавленный элемент на этот виджет тоже работает на слейте. Какие отличия у блюпринтового виджета и чистого слейта расскажу дальше.  

# Заготовка для Slate виджета
Создадим два новых файла SlateWidget.h и SlateWidget.cpp
**SlateWidget.h:**

	#pragma once
	#include "CoreMinimal.h"
	
	class LEARNSLATE_API SlateWidget : public SCompoundWidget
	{
	public:
		SLATE_BEGIN_ARGS(SlateWidget) {}
	
		SLATE_END_ARGS()
		
		void Construct(const FArguments& InArgs);
	};

**SlateWidget.cpp**

	#include "SlateWidget.h"
	
	void SlateWidget::Construct(const FArguments& InArgs)
	{
		
	}

Примерно вот так выглядит пустая заготовка на слейте. Про .cpp пока говорить нечего, там пустой конструктор, а вот про .h есть что рассказать. Внимательные заметят, что тут нет стандартного анриловского макроса **UCLASS()**, а это значит, что слейт низкоуровненый фреймворк и доступа к привычным нам функциям UObject у нас нет.  
Дальше идут макросы **SLATE_BEGIN_ARGS(SlateWidget) {} SLATE_END_ARGS()** они позволяют нам передавать в слейт данные, например текст, текстуру, указатель на эктора и тд.  
Последней идет функция **void Construct(const FArguments& InArgs);** в ней мы будем создавать наш виджет, а в аргументы ей подаются данные, которые мы указали в макросах.

# Первый виджет на Slate
Первым делом мы создадим простой виджет, который будет выводить на экран текущее время:
**SlateWidget.cpp**

	#include "SlateWidget.h"
	
	void SlateWidget::Construct(const FArguments& InArgs)
	{
		ChildSlot
		.HAlign(HAlign_Center)
		.VAlign(VAlign_Center)
		[
				SNew(SBorder)
			.HAlign(HAlign_Center)
			.VAlign(VAlign_Center)
			.BorderImage(FCoreStyle::Get().GetBrush("ToolPanel.GroupBorder"))
			[
				SNew(STextBlock)
				.Text(this, &SlateWidget::GetTime)
			]
		];
	}
	
	FText SlateWidget::GetTime() const
	{
		FDateTime CurrentTime = FDateTime::Now();
		return FText::FromString(CurrentTime.ToString(TEXT("%H:%M:%S")));
	}
Теперь мы видим что-то очень непохожее на обычный анриловский код, но пугаться не надо, все очень просто. В слейте все работает по типу слотов(контейнеров), которые могут содержать что-то, в том числе другие слоты.  
Сначала идет **ChildSlot**, это слот содержащий всех потомков этого виджета, по сути мы крепим к нему все свои виджеты, которые пишем дальше.  
**.HAlign(HAlign_Center).VAlign(VAlign_Center)** это расположение ChildSlot, в данном случае по центру по вертикали и горизонтали.  
Дальше идет символ **\[** который открывает содержимое слота ChildSlot, закрывается он в конце обратным символом с точнкой и запятой **]**




















# Заключение
Возможность менять значения переменных блюпринтов с помощью других блюпринтов открывает большие возможности по написанию инструментов для упрощения разработки. Все ограничивается только вашей фантазией.

# Полезные ссылки:
[Подробный пост про CDO](https://1danielcoelho.github.io/unreal-engine-basics-base-classes/)  
[Unreal Property System (Reflection)](https://www.unrealengine.com/en-US/blog/unreal-property-system-reflection)
