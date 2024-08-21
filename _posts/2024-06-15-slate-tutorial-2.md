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
Создадим файл InventoryComponent.h:
**InventoryComponent.h**

```.h
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
struct FItem
{
	GENERATED_BODY()
public:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
	TSubclassOf<AItem> ItemClass;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
	int32 Count;

	FItem()
	{
		ItemClass = nullptr;
		Count = 0;
	}

	FItem(TSubclassOf<AItem> aItemClass, int32 aCount)
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
	UInventoryComponent();

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
	int32 InventorySizeX;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
	int32 InventorySizeY;

private:
	TArray<FItem> Items;
	
};
```  
В этом коде мы добалвяем класс **AItem** который представляет собой класс предмета и пока хранит в себе только иконку, структуру **FItem** которая хранит в себе класс предмета и его кол-во, а так же систему инвентаря **UInventoryComponent**, которая хранит в себе предметы с помощью структуры **FItem**.

## Ячейка инвентаря
Ясное дело, что инвентарь будет состоять из ячеек, по сути мы создадим только одну ячейку и будем дублировать её нужное нам кол-во раз.  
Ячейка будет нести в себе следующую информацию:  
 - Иконка  
 - Количество  

Создами два новых файла ItemSlotWidget.cpp и ItemSlotWidget.h  








ПРО ДОБАВЛЕНИЕ МОДУЛЯ СКАЗАТЬ









## Заключение
С помощью Slate можно создать почти всё, что угодно: меню, инвентарь, блокнот, да хоть целую игру. Главное — перестать его бояться и понять основы. В следующих уроках я покажу, как на Slate создать UI системy инвентаря.

![]({{site.baseurl}}/images/2024-06-15-slate-tutorial-1/2024-06-15-slate-tutorial-1_3.png)
