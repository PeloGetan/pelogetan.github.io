---
layout: post
title: UE Slate урок 1
published: false
---
![]({{site.baseurl}}/images/2024-06-15-slate-tutorial-1/2024-06-15-slate-tutorial-1_0.png)  
К моему удивлению, на просторах интернета практически нет уроков по слейту, а те что есть объясняют только самые основы. Пришло время это исправить. Я постараюсь написать несколько уроков, в которых затрону основные вопросы и проблемы связанные со слейтом.  
**Сразу замечу, что вы должны иметь хотя бы минимальный опыт работы с блюпринтовыми виджетами, иначе вам будет гораздо сложнее в изучении слейта.**

## Slate
**Slate** - это фреймворк анрила для создания пользовательского интерфейса. Все что вы видете на экране при запуске анрила, это слейт, все меню, все кнопки и тд. Блюпринтовые виджеты и любые добавленные на них элементы тоже работают на слейте.  


## Заготовка для Slate виджета
Создадим два новых файла MyWidget.h и MyWidget.cpp  
**MyWidget.h:**

	#pragma once
	#include "CoreMinimal.h"
	
	class LEARNSLATE_API SMyWidget : public SCompoundWidget
	{
	public:
		SLATE_BEGIN_ARGS(SMyWidget) {}
	
		SLATE_END_ARGS()
		
		void Construct(const FArguments& InArgs);
	};

**MyWidget.cpp**

	#include "MyWidget.h"
	
	void SMyWidget::Construct(const FArguments& InArgs)
	{
		
	}

Примерно так выглядит пустая заготовка на слейте. Про .cpp пока говорить нечего, там пустой конструктор, а вот про .h есть что рассказать. Внимательные заметят, что тут нет стандартного анриловского макроса **UCLASS()**, все потому, что слейт низкоуровненый фреймворк и доступа к привычным функциям UObject у нас нет.  
Дальше идут макросы **SLATE_BEGIN_ARGS(SMyWidget) {} SLATE_END_ARGS()** они позволяют нам передавать в слейт данные, например текст, текстуру, указатель на эктора и тд.  
Последней идет функция **void Construct(const FArguments& InArgs);** в ней мы будем создавать наш виджет, а в аргументы подаются данные, которые мы указали в макросах.  


## Первый виджет на Slate
Мы создадим простой виджет, который будет выводить на экран текущее время:  
**MyWidget.cpp**

	#include "MyWidget.h"
	
	void SMyWidget::Construct(const FArguments& InArgs)
	{
		ChildSlot
		.HAlign(HAlign_Center)
		.VAlign(VAlign_Center)
		[
			SNew(SBorder)
			.HAlign(HAlign_Center)
			.VAlign(VAlign_Center)
			.BorderImage(FCoreStyle::Get().GetBrush("WhiteBrush"))
			[
				SNew(STextBlock)
				.Text(this, &SMyWidget::GetTime)
                .ColorAndOpacity(FLinearColor::Black)
			]
		];
	}
	
	FText SMyWidget::GetTime() const
	{
		FDateTime CurrentTime = FDateTime::Now();
		return FText::FromString(CurrentTime.ToString(TEXT("%H:%M:%S")));
	}

Видим что-то очень непохожее на обычный анриловский код, но пугаться не надо, все очень просто. В слейте все работает по типу слотов(контейнеров), которые могут содержать что-то, в том числе другие слоты.  
Для наглядности я собрал такой же виджет на блюпринтах, вот как он выглядит:

![]({{site.baseurl}}/images/2024-06-15-slate-tutorial-1/2024-06-15-slate-tutorial-1_2.png)  

Сначала идет **ChildSlot**, это слот содержащий всех потомков этого виджета, мы крепим к нему все элементы, которые пишем дальше.  
**.HAlign(HAlign_Center).VAlign(VAlign_Center)** это расположение **ChildSlot**, в данном случае по центру по вертикали и горизонтали. Сразу замечу, что именно таким образом производится настройка и изменение элементов слейта, через точку и вызов нужного параметра: **.Halight(HAlign_Center) .Text(FText::FromString("Some Text")) .Image(PathToImage)**.
Дальше идет квадратная скобка **\[** который открывает содержимое слота **ChildSlot**, закрывается он в конце обратным символом с точкой и запятой **];** но только главный слот закрывается точкой с запятой, слоты внутри главного слота закрываются просто квадратной скобкой **]**.

Далее **SNew(SBorder)**, тут мы впервые встречаем функцию слейта **SNew**, с её помощью мы создаем новые элементы, будь то слот, или его содержимое. В данном случае мы создаем элемент **SBorder**. Этот элемент хранит в себе изображение в качестве подложки и слот, куда мы можем что-то поместить. Но слот только один. **.HAlign(HAlign_Center).VAlign(VAlign_Center)** так же как и выше определяет расположение слота.  
**.BorderImage(FCoreStyle::Get().GetBrush("WhiteBrush"))** - в этой строке мы указываем одну из заготовленных брашей движка в качестве изображения бордера, это будет белая заливка.  

Дальше мы попадем в слот внутри **SBorder** и создаем **STextBlock**, которому задаем в качестве аргумента **.Text** функцию. Можно и просто написать что-то вроде **.Text(FText::FromString("Hello, Slate!"))** и это будет работать, но так как время не стоит на месте (а жаль), его надо постоянно обновлять, а если указать в качестве аргумента функцию, то элемент будет сам обновляться. **.ColorAndOpacity(FLinearColor::Black)** делает цвет шрифта черным. Функцию **SMyWidget::GetTime()** объяснять не буду, так как там простой анриловский код.  

Отлично, у нас есть виджет, но как его показать на экране? Через HUD, поэтому создаем новый класс:
**MyHUD.h**
	
    #pragma once
	
	#include "CoreMinimal.h"
	#include "GameFramework/HUD.h"
	#include "MyHUD.generated.h"
	
	UCLASS()
	class LEARNSLATE_API AMyHUD : public AHUD
	{
		GENERATED_BODY()
	
	protected:
	
		TSharedPtr<class SMyWidget> TimeWidget;
	
		virtual void BeginPlay() override;
	};
    
**MyHUD.cpp**

	#include "MyHUD.h"
	#include "MyWidget.h"
	
	void AMyHUD::BeginPlay()
	{
		Super::BeginPlay();
	
		if(GEngine && GEngine->GameViewport)
		{
			TimeWidget = SNew(SMyWidget);
			GEngine->GameViewport->AddViewportWidgetContent(TimeWidget.ToSharedRef());
		}
	}

В .h файле мы сохранили ссылку на виджет, а в .cpp создали и добавили слейт виджет на экран. Теперь в гейммоде нужно указать MyHUD как HUD по умолчанию и можно запускать проект.

![]({{site.baseurl}}/images/2024-06-15-slate-tutorial-1/2024-06-15-slate-tutorial-1_1.png)  


## Заключение
На слейте можно написать почти все что угодно, меню, инвентарь, блокнот, да хоть целую игру. Главное перестать его бояться и понять основы.