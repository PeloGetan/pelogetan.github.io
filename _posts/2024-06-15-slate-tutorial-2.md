---
layout: post
title: UE Slate урок 2
---
![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_16.png)  
Это второй урок по Slate, в нем мы начнем создание инвентаря на этом фреймворке.  

## Инвентарь и предметы
Прежде чем приступать к созданию UI ячейки, нужно написать код, который она будет визуализировать. Систему инвентаря я буду разрабатывать постепенно, добавляя новый функционал в каждом уроке. В этом уроке я напишу только те части, которые необходимы для отображения информации в ячейке.  
Создадим файлы **InventoryComponent.h** и **InventoryComponent.cpp**:  

**InventoryComponent.h**
```
#pragma once
#include "CoreMinimal.h"
#include "Components/ActorComponent.h"
#include "InventoryComponent.generated.h"

UCLASS(Blueprintable, BlueprintType)
class LEARNSLATE_API AItem : public AActor
{
	GENERATED_BODY()
public:
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item")
	FSlateBrush ItemIcon;
};

USTRUCT(BlueprintType)
struct FInventoryItem
{
	GENERATED_BODY()
public:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
	TSubclassOf<AItem> ItemClass;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
	int32 Count;

	FInventoryItem()
	{
		ItemClass = nullptr;
		Count = -1;
	}

	FInventoryItem(TSubclassOf<AItem> aItemClass, int32 aCount)
	{
		ItemClass = aItemClass;
		Count = aCount;
	}
};

UCLASS(Blueprintable, BlueprintType, meta = (BlueprintSpawnableComponent))
class LEARNSLATE_API UInventoryComponent : public UActorComponent
{
	GENERATED_BODY()
public:
	virtual void BeginPlay() override;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
	int32 InventorySizeX = 3;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
	int32 InventorySizeY = 3;
	TArray<FInventoryItem>& GetItems() { return Items; }
	
protected:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
	TArray<FInventoryItem> Items;
};
```
**InventoryComponent.cpp**
```
#include "InventoryComponent.h"

void UInventoryComponent::BeginPlay()
{
	Super::BeginPlay();
	for(int32 i = Items.Num(); i < InventorySizeX * InventorySizeY; i++)
	{
		Items.Add(FInventoryItem());
	}
}
```
В этом коде мы добавляем класс **AItem**, который представляет собой класс предмета и пока хранит в себе только иконку, структуру **FInventoryItem**, которая содержит информацию о классе предмета и его количестве, а также систему инвентаря **UInventoryComponent**, которая использует структуру **FInventoryItem** для хранения предметов. В **BeginPlay** мы создаем пустые слоты для предметов. Эти слоты являются частью системы инвентаря, и о них мы поговорим в следующих уроках.  

Теперь нужно добавить этот компонент в класс нашего **Character**. Откроем **"имя_вашего_проекта_Character.h"** и добавим следующий код:  
```
public:
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Inventory")
	class UInventoryComponent* InventoryComponent;
```

В **.cpp** файле в конструкторе **Character** создаем компонент и назначаем его в нашу переменную:  
```
InventoryComponent = CreateDefaultSubobject<UInventoryComponent>(TEXT("InventoryComponent"));
```

## Ячейка инвентаря
Понятно, что инвентарь будет состоять из ячеек, по сути, мы создадим только одну ячейку и будем дублировать её нужное нам количество раз.  
Ячейка будет нести в себе следующую информацию:  
 - Иконка  
 - Количество  

Сейчас нам нужно добавить в модули в файле **название_вашего_проекта.Build.cs** в **PublicDependencyModuleNames** поле "**UMG**", примерно будет выглядеть так:  

```
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "HeadMountedDisplay", "UMG" });  
```

Это необходимо, потому что в этом уроке мы начнем работать с новыми классами пользовательского интерфейса.  
Создами два новых файла ItemSlotWidget.cpp и ItemSlotWidget.h:  

**ItemSlotWidget.h**
```
#pragma once
#include "CoreMinimal.h"

struct FInventoryItem;

class LEARNSLATE_API SItemSlotWidget : public SCompoundWidget
{
public:
	SLATE_BEGIN_ARGS(SItemSlotWidget) {}
	SLATE_ARGUMENT(FInventoryItem*, Item)
	SLATE_END_ARGS()

	FInventoryItem* Item;

	void Construct(const FArguments& InArgs);
	const FSlateBrush* GetItemIcon() const;
	FText GetItemCount() const;
};
```

**.h** файл ничего нового в себе не несет, кроме аргумента **SLATE_ARGUMENT(FInventoryItem*, Item)**, с помощью которого мы передаем в виджет указатель на **Item**. 

**ItemSlotWidget.cpp**
```
void SItemSlotWidget::Construct(const FArguments& InArgs)
{
	Item = InArgs._Item;

	ChildSlot
	[
		SNew(SBox)
		.WidthOverride(110)
		.HeightOverride(110)
		[
			SNew(SOverlay)
			+ SOverlay::Slot()
			.HAlign(HAlign_Fill)
			.VAlign(VAlign_Fill)
			[
				SNew(SImage)
				.Image(FCoreStyle::Get().GetBrush("BlackBrush")) // Slot background
			]
			+ SOverlay::Slot()
			.HAlign(HAlign_Center)
			.VAlign(VAlign_Center)
			[
				SNew(SBox)
				.WidthOverride(100)
				.HeightOverride(100)
				[
					SNew(SOverlay)
					+ SOverlay::Slot()
					.HAlign(HAlign_Fill)
					.VAlign(VAlign_Fill)
					[
						SNew(SImage)
						.Image(FCoreStyle::Get().GetBrush("WhiteBrush")) // Item background
				  ]
					+ SOverlay::Slot()
					.HAlign(HAlign_Center)
					.VAlign(VAlign_Center)
					[
						SNew(SImage)
						.Image(this, &SItemSlotWidget::GetItemIcon) // Item icon
					]
					+ SOverlay::Slot()
					.HAlign(HAlign_Right)
					.VAlign(VAlign_Bottom)
					[
						SNew(STextBlock)
						.Text(this, &SItemSlotWidget::GetItemCount) // Item count
						.ColorAndOpacity(FSlateColor(FLinearColor::Black))
						.Font(FSlateFontInfo(FPaths::EngineContentDir() / TEXT("Slate/Fonts/Roboto-Regular.ttf"), 24))
					]
				]
			]
		]
	];
}

const FSlateBrush* SItemSlotWidget::GetItemIcon() const
{
	if(Item && Item->ItemClass)
	{
		return &Item->ItemClass.GetDefaultObject()->ItemIcon;
	}
	return FCoreStyle::Get().GetBrush("WhiteBrush");
}

FText SItemSlotWidget::GetItemCount() const
{
	if(Item && Item->ItemClass)
	{
		return FText::FromString(FString::FromInt(Item->Count));
	}
	return FText::FromString("");
}
```

**.cpp** файл заметно увеличился по сравнению с прошлым уроком. Несмотря на его возросший объем, в нем нет ничего сложного. Вот как он устроен: 
- Черный задник размером 110 на 110 выполняющий роль рамки ячейки.  
  /- Белый задник размером 100 на 100 выполняющий роль фона ячейки.  
  /- Иконка предмета  
  /- Текст с количеством предметов  
Также есть две функции, которые получают информацию из предмета: **GetItemIcon()** и **GetItemCount()**.

## Панель инвентаря
Теперь, когда у нас есть ячейка, нужно сделать виджет, который будет дублироваться столько раз, чтобы отобразить весь инвентарь.  
Создадим два файла WInventoryMainBar.h и WInventoryMainBar.cpp  
**WInventoryMainBar.h**
```
#pragma once

#include "CoreMinimal.h"
#include "Components/Widget.h"
#include "WInventoryMainBar.generated.h"

class LEARNSLATE_API SInventoryMainBar : public SCompoundWidget
{
public:
	SLATE_BEGIN_ARGS(SInventoryMainBar) {}
	SLATE_ARGUMENT(TWeakObjectPtr<class UInventoryComponent>, Inventory)
	SLATE_END_ARGS()

	void Construct(const FArguments& InArgs);
	TWeakObjectPtr<class UInventoryComponent> Inventory;
	TArray<TSharedPtr<class SItemSlotWidget>> MainInventoryItemsWidgets;
};

UCLASS(Blueprintable, BlueprintType)
class LEARNSLATE_API UWInventoryMainBar : public UWidget
{
	GENERATED_BODY()
public:
	virtual void ReleaseSlateResources(bool bReleaseChildren) override;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Meta = (ExposeOnSpawn = true), Category = "InventoryMainBar")
	class UInventoryComponent* InventoryComponent;

	virtual const FText GetPaletteCategory() override;
protected:
	virtual TSharedRef<SWidget> RebuildWidget() override;
	TSharedPtr<SInventoryMainBar> InventoryMainBarWidget;
};
```

В **.h** появился новый класс **UWidget**, его можно назвать прослойкой между чистым **C++ Slate** и **Bluepint Widgets**. **UWidget** является наследником **UVisual**, который в свою очередь наследуется от **UObject**. Это хорошая новость для нас, так как теперь нам доступен весь его функционал. Кроме того, **UWidget** можно добавлять напрямую в **Blueprint** виджете прямо в редакторе, что позволяет заниматься версткой этих элементов мышкой в окне, а не кодом в файле.  

Функции:  
- **virtual void ReleaseSlateResources(bool bReleaseChildren)** нужна для очистки нашего Slate виджета из памяти.  
- **virtual const FText GetPaletteCategory()** позволяет нам задать категорию этому виджету, по которой мы потом сможем найти его в Blueprint виджете.  
- **virtual TSharedRef/<SWidget/> RebuildWidget()** создает Slate виджет.  

**WInventoryMainBar.cpp**  
```
#include "WInventoryMainBar.h"
#include "InventoryComponent.h"
#include "ItemSlotWidget.h"
#include "Widgets/Layout/SGridPanel.h"

void SInventoryMainBar::Construct(const FArguments& InArgs)
{
	Inventory = InArgs._Inventory;

	TSharedPtr<SGridPanel> GridPanel;
	ChildSlot
	.HAlign(HAlign_Left)
	.VAlign(VAlign_Center)
	[
			SAssignNew(GridPanel, SGridPanel)
	];

	int32 SlotsCountX = Inventory.IsValid() ? Inventory->InventorySizeX : 3;
	int32 SlotsCountY = Inventory.IsValid() ? Inventory->InventorySizeY : 3;
	int32 Counter = 0;
	for (int32 x = 0; x < SlotsCountX; x++)
	{
		for (int32 y = 0; y < SlotsCountY; y++)
		{
			TSharedPtr<SItemSlotWidget> NewWidget;
			GridPanel->AddSlot(y, x).Padding(10)
			[
				SAssignNew(NewWidget, SItemSlotWidget)
				.Item(Inventory.IsValid() ? &Inventory->GetItems()[Counter] : nullptr)
			];
			MainInventoryItemsWidgets.Add(NewWidget);
			Counter++;
		}
	}
}

void UWInventoryMainBar::ReleaseSlateResources(bool bReleaseChildren)
{
	InventoryMainBarWidget.Reset();
}

TSharedRef<SWidget> UWInventoryMainBar::RebuildWidget()
{
	APlayerController * Controller = GetOwningPlayer();
	if(Controller)
	{
		APawn * Pawn = Controller->GetPawn();
		if(Pawn)
		{
			InventoryComponent = Pawn->FindComponentByClass<UInventoryComponent>();
		}
	}

	InventoryMainBarWidget = SNew(SInventoryMainBar).Inventory(InventoryComponent);
	return InventoryMainBarWidget.ToSharedRef();
}

const FText UWInventoryMainBar::GetPaletteCategory()
{
	return FText::FromString("Inventory");
}

```

В **.cpp** файле обратите внимание на функцию **void SInventoryMainBar::Construct(const FArguments& InArgs)** :
Так как нам нужно будет циклом создать нужное количество ячеек, **GridPanel** создается заранее, и его важно назначить в переменную внутри **ChildSlot**. Далее идет важная проверка на валидность указателя на инвентарь, потому что **во время настройки виджета в эдиторе инвентаря не существует!**  
Затем мы циклом добавляем в **GridPanel** ячейки инвентаря и назначаем им указатели на структуры элементов инвентаря, которые они будут отображать, снова проверяя валидность инвентаря. В случае невалидности ячейке присваивается **nullptr**.

Далее наш **UWidget** класс **UWInventoryMainBar**, функции **ReleaseSlateResources()** и **GetPaletteCategory()** очень просты, поэтому перейдем к **RebuildWidget()** :  
Наша цель тут, создать Slate виджет и дать ему указатель на инвентарь, поэтому пытаемся получить его из **Pawn** игрока. Если мы не получим инвентарь, что обязательно произойдет во время настройки **Blueprint** виджета, то **Slate** виджет создатся с нулевым указателем на инвентарь, что мы предусмотрели, сделав проверки на это.  

Теперь у нас есть виджет, который мы можем добавить в **Blueprint** виджет, но чтобы получить полный контроль над ним в коде, лучше и создать его в коде. Нам нужен класс **UUserWidget**.  
Создадим файл UWMain.h:  
**UWMain.h**
```
#pragma once
#include "CoreMinimal.h"
#include "Blueprint/UserWidget.h"
#include "UWMain.generated.h"

class UCanvasPanel;
class UWInventoryMainBar;

UCLASS(Blueprintable, BlueprintType)
class LEARNSLATE_API UWMain : public UUserWidget
{
	GENERATED_BODY()
public:

	UPROPERTY(BlueprintReadOnly, Category = "UWMain", meta=(BindWidget))
	UCanvasPanel* RootWidget;
	UPROPERTY(BlueprintReadOnly, Category = "UWMain", meta=(BindWidget))
	UWInventoryMainBar* InventoryMainBar;
};
```
Этот файл небольшой и содержит, помимо самого класса, два указателя: **UCanvasPanel RootWidget** и **UWInventoryMainBar InventoryMainBar**. Самое важное здесь - это **meta=(BindWidget)** в **UPROPERTY**. Эта мета работает следующим образом: в **Blueprint** виджете обязательно должен быть дочерний виджет, на который есть указатель, иначе **Blueprint** не скомпилируется и будет выдавать ошибку. Это гарантирует, что у нас не возникнет ситуации, когда мы обращаемся к виджету через указатель, а там **nullptr**. В нашем случае мы обязаны добавить в **Blueprint** виджет **UCanvasPanel** с именем **RootWidget** и **UWInventoryMainBar** с именем **InventoryMainBar**.  


## Изменение кода HUD
Теперь нужно добавить этот виджет в **HUD**, поэтому немного изменим код в **MyHUD.h** и **MyHUD.cpp**:  
**MyHUD.h**
```
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/HUD.h"
#include "MyHUD.generated.h"

class UWMain;

UCLASS()
class LEARNSLATE_API AMyHUD : public AHUD
{
	GENERATED_BODY()

public:
	UPROPERTY(EditAnywhere, Category = "Widgets")
	TSubclassOf<UUserWidget> MainWidgetClass;

protected:
	virtual void BeginPlay() override;
	UWMain* MainWidget;
};
```

Мы добавили ссылку на наш класс виджета.  

**MyHUD.cpp**
```
#include "MyHUD.h"
#include "UWMain.h"
#include "Blueprint/UserWidget.h"

void AMyHUD::BeginPlay()
{
	Super::BeginPlay();
	MainWidget = CreateWidget<UWMain>(GetWorld(), MainWidgetClass);
	if(MainWidget)
	{
		MainWidget->AddToViewport();
	}
}
```
Здесь в начале игры мы создаем и добавляем виджет на экран.

## Последние шаги в блюпринтах  
Запускаем движок. Сначала создадим дочерний класс **AItem**, чтобы у нас был предмет, который можно будет положить в инвентарь.  

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_5.png)  

В этом классе меняем изображение на любое подходящее. Я использую изображение из движка: **T_UE4Logo_Mask**.

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_6.png)  

Теперь открываем нашего игрока **ThirdPersonCharacter**, выбираем **InventoryComponent** и добавляем в массив предметов наш новый класс, количество можно выбрать произвольное, но **элементов в массиве не должно быть больше 9ти**, так как это размер нашего инвентаря.  

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_7.png)  

Теперь создадим дочерний класс нашего виджета **UWMain** и назовем **W_Main**.  

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_8.png) 

Открыв класс, мы видим те ошибки, о которых я говорил:  
```
A required widget binding "RootWidget" of type  Canvas Panel  was not found.
A required widget binding "InventoryMainBar" of type  WInventory Main Bar  was not found.
A required widget binding "RootWidget" of type  Canvas Panel  was not found.
A required widget binding "InventoryMainBar" of type  WInventory Main Bar  was not found.
```
Виджет не будет скомпилирован, пока мы не добавим соответствующие виджеты. Добавляем виджет **UCanvasPanel** с именем **RootWidget** и **UWInventoryMainBar** с именем **InventoryMainBar**.  

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_9.png) 

Кроме того, я немного изменю настройки **InventoryMainBar**, чтобы он находился по центру левого края экрана. Настройки указаны на изображении ниже:  

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_10.png) 

Теперь нужно создать дочерний класс HUD.  

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_11.png) 

Назовем его **HUD_Base** и укажем в нем класс виджета **UWMain**.  

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_12.png) 

И наконец, последнее, что нам нужно сделать, это создать дочерний **GameMode** от класса "**имя_вашего_проекта_GameMode.h**" и назвать его **GM_Base**.

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_13png) 

В нем укажем наш **HUD_Base**. Все остальные настройки можно оставить по умолчанию.   

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_14.png) 

Осталось указать наш новый **GameMode** в настройках проекта.  

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_15.png) 

Поздравляю, вы справились! Теперь можно запускать проект, и на экране должен появиться инвентарь с предметами.  

![]({{site.baseurl}}/images/2024-09-03-slate-tutorial-2/2024-09-03-slate-tutorial-2_16.png) 

## Заключение
После этого урока у вас будет хорошее представление о возможностях Slate и понимание того, как его использовать. Самое сложное уже позади, впереди лишь появление новых функций, таких как обработка нажатий на виджет или операции Drag-and-Drop, но основа останется прежней.
