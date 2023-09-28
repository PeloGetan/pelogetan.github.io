---
layout: post
title: Unreal Engine 4.27 настройка Android билда
published: true
---
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.preview.png)
К сожалению [официальный гайд](https://docs.unrealengine.com/4.27/en-US/SharingAndReleasing/Mobile/Android/Setup/) по настройке Android билда на UE4.27 устарел и если сейчас полностью следовать ему, то вы гарантированно получите такие ошибки:  
- Could not determine the dependencies of task ':app:compileDebugJavaWithJavac'.  
- Installed Build Tools revision 34.0.0 is corrupted. Remove and install again using the SDK Manager.  
- ERROR: cmd.exe failed with args /c "C:\PeloGetan\ProjectName\Intermediate\Android\armv7\gradle\rungradle.bat" :app:assembleDebug.  
Я потратил много своего времени на попытки исправить эти и многие последующие ошибки. Поэтому написал этот гайд, в котором подробно объясню, что и куда надо нажимать, чтобы ничего не сломать.  

# Если вы следовали точно по официальному гайду и еще не успели сделать что-то еще
На настоящий момент решение довольно простое и быстрое:  
**1.** Открываем папку по пути: **"C:\Users\user\AppData\Local\Android\Sdk\build-tools\34.0.0"**.  

**2.** Переименовываем файл **"d8.bat"** на **"dx.bat"**.  

**3.** Там же открываем папку **"lib"** и переименовываем файл **"d8.jar"** на **"dx.jar"**.  

**4.** Запускаем движок и открываем вкладку **Edit->Project Settings** в левом меню окна ищем **Android SDK**, в поле **"SDK API Level"** вводим **"android-29"**.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.16.png)

Запускаем движок и пробуем билдить под андройд, все должно заработать. Если вы получаете ту же, или новую ошибку, значит где-то было отклонение от официально гайда, читайте дальше.  

# Если вы уже устанавливали Android Studio и пытались в нем что-то менять
Анрил очень прихотливый в плане настройки SDK и NDK, чуть шаг влево, шаг вправо, получите сто ошибок во время компиляции. Поэтому первым делом, если вы уже пользовались официальным или, не дай бог, неофициальным гайдом, сносим все до основания. **Это важно!**  
**1.** Удаляем Android Studio с пк через панель управления.  

**2.** Удаляем папки:
- C:\Program Files\Android  
- C:\Users\user\.android  
- C:\Users\user\.gradle  
- C:\Users\user\AppData\Local\Android  
- C:\[Ваш проект на UE4]\Build\  
- C:\[Ваш проект на UE4]\Intermediate\  
- C:\[Ваш проект на UE4]\DerivedDataCache  

**3.** Открываем **"Этот компьютер"**, жмем ПКМ по пустому пространству внутри окна и выбираем **"Свойства"**.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.2.png)

**4.** В открывшемся окне выбираем **"Дополнительные параметры системы"**->**"Переменные среды"**  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.3.png)

**5.** Удаляем следующие переменные среды:  
  - ANDROID_HOME  
  - JAVA_HOME  
  - NDK_ROOT  
  - NDRROOT  
  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.4.png)

**4.** Перезапускаем ПК.  
Ура, мы избавились от мусора, который остался от предыдущих попыток.  

# Установка Android Studio 4 и настройка
В официальном гайде Эпики просят устанавливать **"Android Studio 4.0"**, так и делаем, скачать его можно [с официального сайта](https://developer.android.com/studio/archive), **нужен Android Studio 4.0 May 28, 2020**.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.5.png)
**1.** Запускаем установщик. На всех шагах все оставляем как есть.  

**2.** Запускаем **Android Studio**. Так же на всех шагах все оставляем как есть.  

**3.** Когда настройка завершена и **Android Studio** запущен, жмем **Configure->SDK Manager**.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.6.png)

**4.** Открываем вкладку **SDK Tools** и жмем галочку напротив **Show Package Details**.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.7.png)

**5.** Ищем в списке **"Android SDK Command-line Tools 8.0"** и выбираем его.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.8.png)

**6.** Жмем **Apply->OK->Accept** и ждем окончания установки.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.9.png)

**7.** Когда установка завершилась, жмем **Finish** и закрываем студию, она нам больше не понадобится.  

**8.** Теперь нам нужно перейти в папку с движком, в моем случае это **"C:\UE4.27\UnrealEngine\Engine\Extras\Android"**. Тут нам нужен один из файлов **SetupAndroid**, выберете его в зависимости от вашей ОС (**SetupAndroid.bat** для **Windows**, **SetupAndroid.command** для **Mac**, и **SetupAndroid.sh** для **Linux**).  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.10.png)

**9.** Далее одно из действий в зависимости от файла:  
- **SetupAndroid.bat**, меняем все строки **"SDKMANAGER="** на **"SDKMANAGER=%STUDIO_SDK_PATH%\cmdline-tools\8.0\bin\sdkmanager.bat"**  
- **SetupAndroid.command** или **SetupAndroid.sh**, меняем все строки **"SDKMANAGERPATH="** на **"SDKMANAGERPATH="$STUDIO_SDK_PATH/cmdline-tools/8.0/bin""**  
По сути такие строки там всего две и они рядом.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.11.png)
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.12.png)

**10.** Запускаем измененный файл и ждем, когда прогресс завершится успешно.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.13.png)

**11.** Открываем папку по пути: **"C:\Users\user\AppData\Local\Android\Sdk\build-tools\34.0.0"**.  

**12.** Переименовываем файл **"d8.bat"** на **"dx.bat"**.  

**13.** Там же открываем папку **"lib"** и переименовываем файл **"d8.jar"** на **"dx.jar"**.  

**14.** Перезапускаем ПК.  

**15.** Запускаем движок, открываем вкладку **Edit->Project Settings** в левом меню окна ищем **Android** и в правом жмем **Configure Now**, если не сделали это ранее.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.14.png)

**16.** Листаем ниже до Google Play Servicess, тут тоже жмем **Configure Now**.  
![]({{site.baseurl}}images\2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.15.png)

**17.** В левом меню ищем **Android SDK**, тут можно указать путь к SDK, NDK, Java и выбрать версию. Часто в гайдах говорят сюда что-то вписывать. Если у вас стандартные пути, ничего не меняйте, в поле **"SDK API Level"** введите **"android-29"**.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.16.png)

**18.** Начинаем билд под андройд. Если все было сделано по инструкции, то никаких ошибок быть не должно.  
![]({{site.baseurl}}/images/2023-09-19-android-setup-and-fixing-errors/2023-09-19-android-setup-and-fixing-errors.17.png)

# Если вдруг возникла возникла ошибка
Может быть такое что в какой-то момент во время билда появится какая-нибудь ошибка и чтение лога не дает нормального понимания в чем дело, то нужно снова выполнить этот шаг:  
Удаляем папки:  
- C:\Users\user\.gradle  
- C:\[Ваш проект на UE4]\Build\  
- C:\[Ваш проект на UE4]\Intermediate\  
- C:\[Ваш проект на UE4]\DerivedDataCache  
Запускаем анрил и билдим.  
Если это не помогает, значит переходим в начало этого гайда и сначала удаляем, а потом устанавливаем Android Studio.

# Заключение
Я очень надеюсь, что этот пост поможет кому-то избежать потери времени и нервов. Методом проб и ошибок мне удалось добился стабильной работы UE4.27 с андройдом и не могу не поделиться этой информацией, так как сам не нашел этой информации нигде.
