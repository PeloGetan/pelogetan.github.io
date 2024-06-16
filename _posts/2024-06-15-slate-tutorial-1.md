---
layout: post
title: UE Slate урок 1
published: true
---
![]({{site.baseurl}}/images/2024-06-15-slate-tutorial-1/2024-06-15-slate-tutorial-1_0.png)  
К моему удивлению, на просторах интернета практически нет уроков по Slate, а те, что есть, объясняют только самые основы. Пришло время это изменить. Я постараюсь написать несколько уроков, в которых затрону основные вопросы и проблемы, связанные со Slate. 
**Отметим, что у вас должен быть хотя бы минимальный опыт работы с виджетами в Blueprints, иначе вам будет гораздо сложнее в изучении Slate.**

## Slate
**Slate** - это фреймворк Unreal Engine для создания пользовательского интерфейса. Всё, что вы видите на экране при запуске Unreal Engine, создано с помощью Slate, все меню, все кнопки и тд. Blueprint виджеты и любые добавленные на них элементы тоже работают на Slate.  


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

Примерно так выглядит пустая заготовка на Slate. Пока что .cpp не содержит ничего примечательного, так как там только пустой конструктор, а вот про .h есть что рассказать. Внимательные заметят, что здесь нет стандартного UE макроса **UCLASS()**, все потому, что Slate низкоуровневый фреймворк и доступа к привычным функциям UObject у нас нет.  
Дальше идут макросы **SLATE_BEGIN_ARGS(SMyWidget) {} SLATE_END_ARGS()** они позволяют нам передавать в Slate данные, например текст, текстуру, указатель на эктора и тд.  
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

Этот код сильно отличается от привычного кода на Unreal Engine, не стоит пугаться, всё достаточно просто. В Slate все работает по типу слотов(контейнеров), которые могут содержать что-то, в том числе другие слоты.  
Для наглядности я собрал такой же виджет на Blueprint, вот как он выглядит:

![]({{site.baseurl}}/images/2024-06-15-slate-tutorial-1/2024-06-15-slate-tutorial-1_2.png)  

Сначала идет **ChildSlot**, это слот содержащий всех потомков этого виджета, мы крепим к нему все элементы, которые будут написаны далее.  
**.HAlign(HAlign_Center).VAlign(VAlign_Center)** — это расположение **ChildSlot**, в данном случае по центру по вертикали и горизонтали. Сразу замечу, что именно таким образом производится настройка и изменение элементов **Slate**: через точку и вызов нужного параметра, например: .HAlign(HAlign_Center) .Text(FText::FromString("Some Text")) .Image(PathToImage).
Дальше идет квадратная скобка **\[** который открывает содержимое слота **ChildSlot**, закрывается он в конце обратным символом с точкой и запятой **];** но только главный слот закрывается точкой с запятой, слоты внутри главного слота закрываются просто квадратной скобкой **]**.

Далее **SNew(SBorder)**, тут мы впервые встречаем функцию Slate **SNew**, с её помощью мы создаем новые элементы, будь то слот, или его содержимое. В данном случае мы создаем элемент **SBorder**. Этот элемент хранит в себе изображение в качестве подложки и слот, куда мы можем что-то поместить. Но слот только один. **.HAlign(HAlign_Center).VAlign(VAlign_Center)** так же как и выше определяет расположение слота.  
**.BorderImage(FCoreStyle::Get().GetBrush("WhiteBrush"))** - в этой строке мы указываем одну из заготовленных брашей движка в качестве изображения бордера, это будет белая заливка.  

Дальше мы попадем в слот внутри **SBorder** и создаем **STextBlock**, которому задаем в качестве аргумента **.Text** функцию. Можно и просто написать что-то вроде **.Text(FText::FromString("Hello, Slate!"))** и это будет работать, но так как время не стоит на месте (а жаль), его надо постоянно обновлять, а если указать в качестве аргумента функцию, то элемент будет сам обновляться. **.ColorAndOpacity(FLinearColor::Black)** делает цвет шрифта черным. Функцию SMyWidget::GetTime() объяснять не буду, так как там простой UE код.  

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

В .h файле мы сохранили ссылку на виджет, а в .cpp создали и добавили Slate виджет на экран. Теперь в гейммоде нужно указать MyHUD как HUD по умолчанию и можно запускать проект.

![]({{site.baseurl}}/images/2024-06-15-slate-tutorial-1/2024-06-15-slate-tutorial-1_1.png)  


## Заключение
С помощью Slate можно создать почти всё, что угодно, меню, инвентарь, блокнот, да хоть целую игру. Главное — перестать его бояться и понять основы. В следующих уроках я покажу как на Slate создать UI системы инвентаря.
