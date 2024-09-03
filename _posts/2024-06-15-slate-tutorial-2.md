---
layout: post
title: UE Slate урок 2
published: false
---
![]({{site.baseurl}}/images/2024-06-15-slate-tutorial-1/2024-06-15-slate-tutorial-1_0.png)  
Это второй урок по Slate, в нем мы начнем создание инвентаря на этом фреймворке.  

## Инвентарь и предметы
Прежде чем делать UI ячейки, нужно написать код, который она будет визуализировать. Систему инвентаря я буду писать постепенно в каждом уроке добавляя новый функционал. В этом уроке я напишу только то, что необходимо для отображения информации в ячейке.
Создадим файлы InventoryComponent.h и InventoryComponent.cpp:  

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
В этом коде мы добалвяем класс **AItem** который представляет собой класс предмета и пока хранит в себе только иконку, структуру **FInventoryItem** которая хранит в себе класс предмета и его кол-во, а так же систему инвентаря **UInventoryComponent**, которая хранит в себе предметы с помощью структуры **FInventoryItem**. На BeginPlay мы создаем пустые слоты предметов, они являются частью системы инвентаря, про них будет в следующих уроках.  

Теперь нужно добавить этот компонент в класс нашего **Character**. Открываем **"имя_вашего_проекта_Character.h"** и добавляем код:  
```
public:
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Inventory")
	class UInventoryComponent* InventoryComponent;
```

В **.cpp** в конструкторе **Character** создаем компонент и назначаем в нашу переменную:  
```
InventoryComponent = CreateDefaultSubobject<UInventoryComponent>(TEXT("InventoryComponent"));
```

## Ячейка инвентаря
Ясное дело, что инвентарь будет состоять из ячеек, по сути мы создадим только одну ячейку и будем дублировать её нужное нам кол-во раз.  
Ячейка будет нести в себе следующую информацию:  
 - Иконка  
 - Количество  

Сейчас нам нужно добавить в модули в файле **название_вашего_проекта.Build.cs** в **PublicDependencyModuleNames** поле "**UMG**", примерно будет выглядеть так:  
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "HeadMountedDisplay", "**UMG**" });  
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

**.h** файл ничего нового в себе не несет, кроме аргумента **SLATE_ARGUMENT(FInventoryItem*, Item)**, с его помощью мы передаем в виджет указатель на Item.  

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

**.cpp** сильно возрос по сравнению с прошлым уроком. Хоть он и стал больше, страшного в нем ничего нет, вот как он выглядит:  
- Черный задник размером 110 на 110 выполняющий роль рамки ячейки.  
  /- Белый задник размером 100 на 100 выполняющий роль фона ячейки.  
  /- Иконка предмета  
  /- Текст с количеством предметов  
Так же есть две функции, которые получают информацию из предмета, это **GetItemIcon()** и **GetItemCount()**.

## Панель инвентаря
Теперь когда у нас есть ячейка, нужно сделать виджет, который будет дублировать столько раз, чтобы отобразить весь инвентарь.  
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

В **.h** появился новый класс **UWidget**, его можно назвать прослойкой между чистым **C++ Slate** и **Bluepint Widgets**. UWidget наследник UVisual, который в свою очередь наследник UObject и для нас это отличная новость, потому что теперь нам доступен весь его функционал. Кроме того, UWidget можно добавлять напрямую в Blueprint виджете прямо в эдиторе, что позволяет нам заниматься версткой этих элементов мышкой в окне, а не кодом в файле. Далее о функциях:  
**virtual void ReleaseSlateResources(bool bReleaseChildren)** нужна для очистки нашего Slate виджета из памяти.  
**virtual const FText GetPaletteCategory()** позволяет нам задать категорию этому виджету, по которой мы потом сможем найти его в Blueprint виджете.  
**virtual TSharedRef/<SWidget/> RebuildWidget()** создает Slate виджет.  

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

В **.cpp** файле обратим внимание на функцию **void SInventoryMainBar::Construct(const FArguments& InArgs)** :
Так как нам нужно будет циклом создать нужное кол-во ячеек, то **GridPanel** создаем заранее, при том важно создать и назначить его в переменную внутри **ChildSlot**. Теперь мы можем делать с ним все что хотим и вызывать где хотим. Далее я делаю важную проверку на валидность указателя на инвентарь, потому что **во время настройки виджета в эдиторе инвентаря не существует!**  
Далее мы циклом добавляем в **GridPanel** ячейки инвентаря и назначаем им указатели на структуры элементов инвентаря, которые они будут отображать, опять же делая проверку на валидность инвентаря и в случае не валидности, даем ячейке **nullptr**.  

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
Он небольшой и по сути у нас тут кроме самого класса есть два указателя **UCanvasPanel RootWidget;** и **UWInventoryMainBar InventoryMainBar;**, но самое главное тут, это **meta=(BindWidget)** в **UPROPERTY**. Эта мета работает следующим образом: в **Blueprint** виджете обязан быть дочерний виджет, на который есть указатель, иначе **Blueprint** не скомпилируется и будет выдавать ошибку. Это гарантирует, что у нас не возникнет ситуации, когда мы обращаемся к виджету через указатель, а там **nullptr**. В нашем случае мы обязаны добавить в **Blueprint** виджет **UCanvasPanel** с именем **RootWidget** и **UWInventoryMainBar** с именем **InventoryMainBar**.  


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
А тут в начале игры мы создаем и добавляем виджет на экран.  
Писать код мы закончили, осталось связать все это в блюпринтах.  

## Последние шаги в блюпринтах  
Запускаем движок. Сначала создадим дочерний класс Item чтобы нам было что положить в инвентарь.  
В этом классе меняем Image на любой наш, я использую изображение из движка: T_UE4Logo_Mask.  

Теперь открываем нашего игрока **ThirdPersonCharacter**, выбираем **InventoryComponent** и добавляем в массив предметов наш новый класс, количество можно выбрать произвольное.  

Теперь создадим дочерний класс нашего виджета **UWMain** и назовем **W_Main**. Открываем класс и видим те ошибки, о которых я говорил:  
```
A required widget binding "RootWidget" of type  Canvas Panel  was not found.
A required widget binding "InventoryMainBar" of type  WInventory Main Bar  was not found.
A required widget binding "RootWidget" of type  Canvas Panel  was not found.
A required widget binding "InventoryMainBar" of type  WInventory Main Bar  was not found.
```
Виджет не будет скомпилирован, пока мы не добавим сюда наши виджеты. Добавляем виджет **UCanvasPanel** с именем **RootWidget** и **UWInventoryMainBar** с именем **InventoryMainBar**.  

Кроме того, я дополнительно немного изменю настройки **InventoryMainBar**, чтобы он находился по центру левого края экрана. Настройки на изображении ниже:  

Теперь нужно создать дочерний класс HUD. Назовем его **HUD_Base** и укажем в нем класс виджета **UWMain**.  

И наконец последнее, что нам нужно создать, это дочерний **GameMode** от класса "**"имя_вашего_проекта_Character.h"**"


Создаем HUD и добавляем в него наш класс виджета

Создаем гейм мод  

Добавляем в него HUD и игрока  

Запускаем и радуемся







Не забыть добавить инвентарь игроку








## Заключение
С помощью Slate можно создать почти всё, что угодно: меню, инвентарь, блокнот, да хоть целую игру. Главное — перестать его бояться и понять основы. В следующих уроках я покажу, как на Slate создать UI системy инвентаря.

![]({{site.baseurl}}/images/2024-06-15-slate-tutorial-1/2024-06-15-slate-tutorial-1_3.png)
