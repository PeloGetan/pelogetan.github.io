---
layout: post
title: UE Slate урок 1
published: false
---
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
			.BorderImage(FCoreStyle::Get().GetBrush("WhiteBrush"))
			[
				SNew(STextBlock)
				.Text(this, &SlateWidget::GetTime)
                .ColorAndOpacity(FLinearColor::Black)
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

**.HAlign(HAlign_Center).VAlign(VAlign_Center)** это расположение **ChildSlot**, в данном случае по центру по вертикали и горизонтали. Сразу замечу, что именно таким образом производится настройка и изменение элементов слейта, через точку и вызов нужного параметра.
Дальше идет символ **\[** который открывает содержимое слота **ChildSlot**, закрывается он в конце обратным символом с точнкой и запятой **];**  

Далее **SNew(SBorder)**, тут мы впервые встречаем важную функцию слейта **SNew**, с его помощью мы создаем новый элемент, будь то слот, или его содержимое. В данном случае мы создаем элемент **SBorder**. Этот элемент хранит в себе изображение в качестве подложки и слот, куда мы можем что-то поместить. Но слот только один. **.HAlign(HAlign_Center).VAlign(VAlign_Center)** так же как и выше определяет расположение слота.  
**.BorderImage(FCoreStyle::Get().GetBrush("WhiteBrush"))** - как я писал выше, **SBorder** имеет изображение, в этой строе мы указываем одну из заготовленных брашей движка, это будет белая заливка.  

Дальше мы попадем в слот внутри **SBorder** и создаем **STextBlock**, которому задаем **.Text** в качестве функции. Можно и просто написать что-то вроде **.Text(FText::FromString("Hello, Slate!"))** и это будет работать, но так как время не стоит на месте (а жаль), его надо постоянно обновлять, а если указать в качестве аргумента функцию, то элемент будет сам обновляться. **.ColorAndOpacity(FLinearColor::Black)** делает цвет шрифта черным. Функцию **SlateWidget::GetTime()** объяснять не буду, так как там простой анриловский код.  

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
	
		TSharedPtr<class SlateWidget> TimeWidget;
	
		virtual void BeginPlay() override;
	};
    
**MyHUD.cpp**

	#include "MyHUD.h"
	#include "SlateWidget.h"
	
	void AMyHUD::BeginPlay()
	{
		Super::BeginPlay();
	
		if(GEngine && GEngine->GameViewport)
		{
			TimeWidget = SNew(SlateWidget);
			GEngine->GameViewport->AddViewportWidgetContent(TimeWidget.ToSharedRef());
		}
	}

В .h файле мы сохранили ссылку на виджет, а в .cpp создали и добавили слейт виджет на экран. Теперь в гейммоде нужно указать MyHUD и можно запускать проект.


# Заключение
На слейте можно написать почти все что угодно, самое главное - перестать его бояться.
