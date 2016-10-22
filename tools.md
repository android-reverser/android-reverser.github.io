---
layout: page
title: Инструменты
permalink: /tools/
---

## Декомпиляторы & Анализаторы & Конвертеры
* [Apktool](https://ibotpeaches.github.io/Apktool/) - полная декомпиляция apk файлов.
* [dex2jar](https://github.com/pxb1988/dex2jar) - преобразует apk(dex) в jar для дальнейшей декомпиляции в java.
* [Bytecode viewer](http://bytecodeviewer.com/) - монструозный комбайн для работы с APK. Хороший, но местами не очень стабильный.
* [JD-GUI](http://jd.benow.ca/) - проверенная временем утилита для просмотра *class*(+jar) файлов.
* [Procyon](https://bitbucket.org/mstrobel/procyon/wiki/Home) - набор Java утилит для метапрограммирования. В первую очередь интересен своей (пока эксперементальной) функцией декомпиляции jar файлов.
* [Luyten](https://github.com/deathmarine/Luyten) - GUI для Procyon Java Decompiler.
* [APK Studio](http://www.vaibhavpandey.com/apkstudio/) - кроссплатформенная(Qt) IDE для работы с APK.
* [Enjarify](https://github.com/google/enjarify) - транслятор Dalvik байткода в Java байткод. Официально гугловый.
* [dexterity](https://github.com/rchiossi/dexterity) - достаточно старая (мб уже и мертвая) C библиотека для работы с DEX файлами. Питонячьи биндинги в комплекте.
* [CLASSYSHARK](http://classyshark.com/) - удобный просмотрщик apk-шек и java-бинарей.
* [APKiD](https://github.com/rednaga/APKiD) - позволяет определить применялись ли для apk файла технологии упаковки, обфускации. Так же определяет был ли модифицирован код.
* [JADX](https://github.com/skylot/jadx) - декомпилятор понимающий *apk*, *dex*, *jar*, *class*, *zip* и *aar* файлы. Есть консольный и GUI интерфейсы. Работает хорошо, но весьма требователен к ресурсам.

## Деобфускаторы
* [Simplify](https://github.com/CalebFenton/simplify) - простой и мощный деобфускатор.
* [dex-oracle](https://github.com/CalebFenton/dex-oracle) - еще один деобфускатор вдохновленный предыдущим =)

## Отладчики
* [GikDbg.ART](http://gikir.com/product.php) - OllyDbg адаптированный под отладку нативного Android кода.
* [smalidea](https://github.com/JesusFreke/smali/wiki/smalidea) - плагин к AndroidStudio/InjelliJIDEA который позволяет отлаживать smali код. Помимо этого имеет так же ряд приятных плюшек вроде подсветки синтаксиса и перехода к определению функции. 

## Дизассемблеры
* [IDA Pro](https://www.hex-rays.com/products/ida/) - дизассемблер-легенда. В новых версиях появилась возможность отладки нативного Android кода.
* [Hopper](https://www.hopperapp.com/) - может выступать как альтернатива предыдущему. Имеет гораздо меньше возможностей, но и цена значительно ниже.

## HEX редакторы
* [WinHex](https://www.x-ways.net/winhex/) - достаточно старый и мощный hex-редактор. Поддерживается до сих пор.
* [Synalyze It](https://www.synalysis.net/) - весьма интересный и удобный hex-редактор. Для macOS пожалуй лучший.
