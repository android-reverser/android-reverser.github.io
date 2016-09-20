---
layout: article
date: 2016-09-20
title: "Обнаружение пиратских и вредоносных андройд приложений с помощью APKiD"
author: "Caleb Fenton"
description: "Android приложения, модифицировать гораздо проще, чем приложения для традиционных настольных операционных систем, таких как Windows или Linux и существует в основном только один способ модификации android приложений, после того как они были скомпилированы"
keywords: "реверс инжиниринг, apkid, вредоносный код"
comments: true
---

[Оригинал](http://rednaga.io/2016/07/31/detecting_pirated_and_malicious_android_apps_with_apkid/)


Android приложения, модифицировать гораздо проще, чем приложения для традиционных настольных операционных систем, таких как Windows или Linux и существует в основном только один способ модификации android приложений, после того как они были скомпилированы: [dexlib](https://mvnrepository.com/artifact/org.smali/dexlib2). Даже если вы иcпользуете [Apktool](https://ibotpeaches.github.io/Apktool/) или [Smali](https://github.com/JesusFreke/smali), то они все равно под капотом используют dexlib. На самом деле, Apktool использует Smali, а Smali и dexlib являются частью одного и того же проекта.

Почему это важно? Любое приложение в которое был встроен вредоносный код или которое было взломано, *возможно* было разобрано и перекомпилировано с помощью dexlib. К тому же, существует очень мало причин по которым разработчики имеющие доступ к исходному коду стали бы использовать dexlib. Следовательно, вы знаете, что приложение, которое было модифицировано с помощью dexlib может быть интересным для вас, если вы беспокоитесь о вредоносном коде или пиратстве.

### APKiD
APKiD может посмотреть на Android APK или DEX файл и определить характерные признаки нескольких различных компиляторов:

* dx - стандартный Android SDK компилятор
* dexmerge - используется для инкрементальной сборки некоторыми IDE (после использования dx)
* dexlib 1.x
* dexlib 2.x beta
* dexlib 2.x

Если хоть какое-нибудь из этих семейств dexlib было использовано для создания DEX файла, то можно справедливо подозревать, что файл был взломан и в него был вставлен вредоносный код. Более подробно о том как мы использовали характерные особенности компиляторов для определения вредоносного кода и кряков, можно прочитать в нашей статье [Android Compiler Fingerprinting](http://rednaga.io/2016/07/30/apkid_and_android_compiler_fingerprinting/).

### Обнаружение dx и dxmerge
Для того чтобы идентифицировать dx или dexmerge нужно посмотреть на то, как упорядочена карта типов в DEX файле

![abnormal type order](http://rednaga.io/images/detecting_pirated_and_malicious_android_apps_with_apkid/abnormal_type_order.png)

Это хорошее место для того чтобы идентифицировать, различные компиляторы, потому что порядок не определен в спецификации и таким образом компилятор сам решает, как упорядочить эти штуки.

Для того чтобы иметь что-то, что можно скопипастить, привожу Java код для нормального порядка типов:
{% highlight java %}
private static final TypeCode[] NORMAL_TYPE_ORDER = new TypeCode[] {
  TypeCode.HEADER_ITEM,
  TypeCode.STRING_ID_ITEM,
  TypeCode.TYPE_ID_ITEM,
  TypeCode.PROTO_ID_ITEM,
  TypeCode.FIELD_ID_ITEM,
  TypeCode.METHOD_ID_ITEM,
  TypeCode.CLASS_DEF_ITEM,
  TypeCode.ANNOTATION_SET_REF_LIST,
  TypeCode.ANNOTATION_SET_ITEM,
  TypeCode.CODE_ITEM,
  TypeCode.ANNOTATIONS_DIRECTORY_ITEM,
  TypeCode.TYPE_LIST,
  TypeCode.STRING_DATA_ITEM,
  TypeCode.DEBUG_INFO_ITEM,
  TypeCode.ANNOTATION_ITEM,
  TypeCode.ENCODED_ARRAY_ITEM,
  TypeCode.CLASS_DATA_ITEM,
  TypeCode.MAP_LIST
};
{% endhighlight %}

Порядок типов для dexmerge был получен из [DexMerger.java](http://osxr.org/android/source/dalvik/dx/src/com/android/dx/merge/DexMerger.java#0111). Я получил порядок идентификаторов типов посмотрев [сюда](http://osxr.org/android/source/dalvik/dx/src/com/android/dx/merge/DexMerger.java#0904).


{% highlight java %}
private static final TypeCode[] DEXMERGE_TYPE_ORDER = new TypeCode[] {
  TypeCode.HEADER_ITEM,
  TypeCode.STRING_ID_ITEM,
  TypeCode.TYPE_ID_ITEM,
  TypeCode.PROTO_ID_ITEM,
  TypeCode.FIELD_ID_ITEM,
  TypeCode.METHOD_ID_ITEM,
  TypeCode.CLASS_DEF_ITEM,
  TypeCode.MAP_LIST,
  TypeCode.TYPE_LIST,
  TypeCode.ANNOTATION_SET_REF_LIST,
  TypeCode.ANNOTATION_SET_ITEM,
  TypeCode.CLASS_DATA_ITEM,
  TypeCode.CODE_ITEM,
  TypeCode.STRING_DATA_ITEM,
  TypeCode.DEBUG_INFO_ITEM,
  TypeCode.ANNOTATION_ITEM,
  TypeCode.ENCODED_ARRAY_ITEM,
  TypeCode.ANNOTATIONS_DIRECTORY_ITEM
};
{% endhighlight %}

В целом, формат DEX файла и элементы внутри него выглядят так:

```
header
  HEADER_ITEM
stringIds
  STRING_ID_ITEM
typeIds
  TYPE_ID_ITEM
protoIds
  PROTO_ID_ITEM
fieldIds
  FIELD_ID_ITEM
methodIds
  METHOD_ID_ITEM
classDefs
  CLASS_DEF_ITEM
wordData (sort by TYPE)
  ANNOTATION_SET_REF_LIST
  ANNOTATION_SET_ITEM
  CODE_ITEM
  ANNOTATIONS_DIRECTORY_ITEM
typeLists (no sort)
  TYPE_LIST
stringData (sort by INSTANCE)
  STRING_DATA_ITEM
byteData (sort by TYPE)
  DEBUG_INFO_ITEM
  ANNOTATION_ITEM
  ENCODED_ARRAY_ITEM
classData (no sort)
  CLASS_DATA_ITEM
map (no sort)
  MAP_LIST
```

Этот список может пригодиться для текущий исследований характерных особенностей различных компиляторов.

### Обнаружение dexlib 1.x
Это первая библиотека, которая позволяет разбирать и собирать DEX без исходного кода. Она была создана Беном Грувером (Ben “Jesus Freke” Gruver). Она определяется прежде всего по физической сортировке строк.

![abnormal string sort](http://rednaga.io/images/detecting_pirated_and_malicious_android_apps_with_apkid/abnormal_string_sort.png)

DEX формат требует, чтобы все таблица строк, которая содержит все строки и их смещения в файле, была отсортирована в алфавитном порядке, но фактический физический порядок строк в файле не обязательно отсортирован. Таким образом dx сортирует строки в алфавитном порядке, хоть это и не обязательно, dexlib же сортирует их физически основываясь на том порядке, в каком он их встречает во время компиляции.

Огромное количество коммерческих упаковщиков, обфускаторов, а также определенные семейства вредоносного ПО, все еще используют dexlib 1.x под капотом, потому что это достаточно надежно, а они слишком ленивы чтобы обновлять ее.

### Обнаружение dexlib 2.x beta
Dexlib 1.x была переписана в dexlib 2 и пока она была в стадии beta-версии, мы заметили, что она делала что-то странное с тем, как она помечала интерфейсы классов.

![abnormal class interfaces](http://rednaga.io/images/detecting_pirated_and_malicious_android_apps_with_apkid/abnormal_class_interfaces.png)

По всему файлу можно увидеть последовательность `AC 27 00 00`. Это смещение для “null” интерфейса для классов, которые не реализуют никаких интерфейсов. Это хороший пример того, насколько гибким является DEX формат, потому что я считаю, что это не должно работать вообще, но оно работает. Компилятор dx просто использует `00` чтобы указать на отсутствие интерфейса.

Это было удалено после того как dexlib 2.x вышел из стадии beta.


### Обнаружение dexlib 2.x
Этот компилятор также определяется по порядку карты типов. Компоновка DEX файла это сложно и существует огромное количество крошечных деталей, которые вам нужно имитировать чтобы получить абсолютно точную копию. Эту огромную кучу дополнительной работы разработчики делать как правило не хотят. 

С другой стороны, я провел много времени используя эту библиотеку и смотря на ее код пока работал над универсальным Android обфускатором, который называется [Simplify](https://travis-ci.org/CalebFenton/simplify). И я должен сказать - это действительно выразительный и чистый код благодаря которому я многому научился. Слава [Бену](https://github.com/JesusFreke).


### Использование APKiD
Пользоваться APKiD очень просто. Вы просто указываете ему на папки или файлы и он будет пытаться найти APK и DEX файлы. Он так же проанализирует APK и попробует найти сжатые APK, DEX и ELF файлы. Вот пример вывода:

{% highlight bash %}
$ apkid test-data/apk test-data/dex
[!] APKiD 0.9.3 :: from RedNaga :: rednaga.io
[*] test-data/dex/dexguard1.dex
 |-> compiler : dexlib 1.x
 |-> obfuscator : DexGuard
[*] test-data/dex/dexguard2.dex
 |-> anti_disassembly : illegal class name
 |-> compiler : dexlib 1.x
 |-> obfuscator : DexGuard
[*] test-data/dex/dexguard3.dex
 |-> anti_disassembly : illegal class name
 |-> compiler : dexlib 1.x
 |-> obfuscator : DexGuard
[*] test-data/dex/dexlib1.dex
 |-> compiler : dexlib 1.x
[*] test-data/dex/dexlib2.dex
 |-> compiler : dexlib 2.x
[*] test-data/dex/dexmerge.dex
 |-> compiler : Android SDK (dexmerge)
[*] test-data/dex/dexprotector1.dex
 |-> compiler : dexlib 1.x
 |-> obfuscator : DexProtect
[*] test-data/dex/dexprotector2.dex
 |-> compiler : dexlib 1.x
 |-> obfuscator : DexProtect
[*] test-data/dex/dexprotector3.dex
 |-> compiler : dexlib 1.x
 |-> obfuscator : DexProtect
[*] test-data/dex/dx.dex
 |-> compiler : Android SDK (dx)
{% endhighlight %}

Можно заметить, что тестовые примеры DexGuard и DexProtector используют dexlib 1.x. APKiD так же поддерживает вывод в формате JSON, что позволяет очень просто интегрировать его в другие утилиты.

### Идеи на будущее
Эта статья не включает в себя информацию о характерных особенностях Android XML изученных [Тимом](https://github.com/strazzere), которые позволяют идентифицировать такие инструменты как Apktool. Нам все еще нужно добавить эти характерные особенности в APKiD.

Существует так же библиотека ASMDEX которая выглядит способной создавать DEX файлы. На момент первоначального исследования несколько лет назад, у меня не было времени чтобы посмотреть на нее и никто не говорил о том как ей пользоваться. Много чего было выше моего понимания, но с тех пор я много практиковался в использовании ASM для создания java class файлов, так что думаю я справлюсь. Так же было бы хорошо добавить характерные особенности для ASMDEX. 

