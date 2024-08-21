---
layout: post
title: UE Slate урок 2
published: false
---
![]({{site.baseurl}}/images/2024-06-15-slate-tutorial-1/2024-06-15-slate-tutorial-1_0.png)  
Это второй урок по Slate, в нем мы начнем создание инвентаря на этом фреймворке.  

## Инвентарь и предметы
Прежде чем делать UI ячейки, нужно написать код, который она будет визуализировать. Я написать примитивную и не до конца функциональную систему инвентаря,
 просто чтобы её хватило для отображения информации в ячейке.  
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

## Ячейка инвентаря
Ясное дело, что инвентарь будет состоять из ячеек, по сути мы создадим только одну ячейку и будем дублировать её нужное нам кол-во раз.  
Ячейка будет нести в себе следующую информацию:  
 - Иконка  
 - Количество  

Первым делом нужно добавить в модули в файле **название_вашего_проекта.Build.cs** в **PublicDependencyModuleNames** поле "**UMG**", примерно будет выглядеть так:  
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

**.h** файл ничего нового в себе не несет, кроме аргумента **SLATE_ARGUMENT(FInventoryItem*, Item)**, с его помощью мы передаем в виджет указатель на Item. А вот **.cpp** сильно возрос по сравнению с прошлым уроком. Хоть он и стал больше, страшного в нем ничего нет, вот как он выглядит:  
- Черный задник размером 110 на 110 выполняющий роль рамки ячейки.  
  /- Белый задник размером 100 на 100 выполняющий роль фона ячейки.  
  /- Иконка предмета  
  /- Текст с количеством предметов  
Так же есть две функции, которые получают информацию из предмета, это **GetItemIcon()** и **GetItemCount()**.

## Панель инвентаря
Теперь когда у нас есть ячейка, нужно сделать виджет, который будет дублировать столько раз, чтобы отобразить весь инвентарь.  
Создадим два файла 




Не забыть добавить инвентарь игроку








## Заключение
С помощью Slate можно создать почти всё, что угодно: меню, инвентарь, блокнот, да хоть целую игру. Главное — перестать его бояться и понять основы. В следующих уроках я покажу, как на Slate создать UI системy инвентаря.

![]({{site.baseurl}}/images/2024-06-15-slate-tutorial-1/2024-06-15-slate-tutorial-1_3.png)
