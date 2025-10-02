---
## Front matter
title: " Отчёт по выполнению лабораторной работы 1"
author: " "

## Generic otions
lang: ru-RU
toc-title: "Содержание"

## Bibliography
bibliography: bib/cite.bib
csl: pandoc/csl/gost-r-7-0-5-2008-numeric.csl

## Pdf output format
toc: true # Table of contents
toc-depth: 2
lof: true # List of figures
lot: true # List of tables
fontsize: 12pt
linestretch: 1.5
papersize: a4
documentclass: scrreprt
## I18n polyglossia
polyglossia-lang:
  name: russian
  options:
	- spelling=modern
	- babelshorthands=true
polyglossia-otherlangs:
  name: english
## I18n babel
babel-lang: russian
babel-otherlangs: english
## Fonts
mainfont: PT Serif
romanfont: PT Serif
sansfont: PT Sans
monofont: PT Mono
mainfontoptions: Ligatures=TeX
romanfontoptions: Ligatures=TeX
sansfontoptions: Ligatures=TeX,Scale=MatchLowercase
monofontoptions: Scale=MatchLowercase,Scale=0.9
## Biblatex
biblatex: true
biblio-style: "gost-numeric"
biblatexoptions:
  - parentracker=true
  - backend=biber
  - hyperref=auto
  - language=auto
  - autolang=other*
  - citestyle=gost-numeric
## Pandoc-crossref LaTeX customization
figureTitle: "Рис."
tableTitle: "Таблица"
listingTitle: "Листинг"
lofTitle: "Список иллюстраций"
lotTitle: "Список таблиц"
lolTitle: "Листинги"
## Misc options
indent: true
header-includes:
  - \usepackage{indentfirst}
  - \usepackage{float} # keep figures where there are in the text
  - \floatplacement{figure}{H} # keep figures where there are in the text
---

# Цель работы

Закрепить практические навыки устранения уязвимостей и защиты корпоративных сервисов.

# Задание

Провести анализ уязвимостей и устранить их на примере MS Exchange и сопутствующих сервисов.

# Выполнение лабораторной работы

## Уязвимость «PROXYLOGON»

Proxylogon представляетсобой SSRF- уязвимость, позволяющую обойти аутентификацию и выдать себя за администратора

Сначала мы завели карточку с описание уязвимости, ее индикаторами и рекомендациями по устранению

![карточка](image/1.png){ #fig:001 width=70% }  

Устраняем уязвимость, ограничиваем доступ к панели управления

![1.1](image/02.png){ #fig:002 width=70% } 

Далее нужно обнаружить последствия и убрать их. В нашем случае это - Exchange China Chopper

![удаление файла веб-оболочки](image/01.png){ #fig:003 width=70% } 


## Уязвимости «ROCKETCHAT»
CVE-2021-22911 представляет собой сочетание из двух SQL-инъекций.
CVE-2022-0847(Dirty Pipe) представляет собой уязвимость повышения привилегий, находящаяся в самом ядре Linux версии 5.8 и выше

Завели карточку с описание уязвимости, ее индикаторами и рекомендациями по устранению

![карточка](image/6.png){ #fig:004 width=70% } 

После обнаружения уязвимости (Dirty Pipe) , проверяем версию ядра Линукс. 
![7](image/7.png){ #fig:005 width=70% }  

Далее меняем в файле конфигурации строку GRUB_DEFAULT=0 на GRUB_DEFAULT=”1>X”

![8](image/8.png){ #fig:006 width=70% }  

Перезагружаем систему и проверяем версию ядра

![9](image/9.png){ #fig:007 width=70% }  
![10](image/10.png){ #fig:008 width=70% }  
![11](image/11.png){ #fig:009 width=70% }  
![12](image/12.png){ #fig:010 width=70% }  

Также нам нужно было сброосить пароль, используя данную почту, мы сбросили пароль и и через терминал получили ссылку для сброса пароля. далее мы столкнулись с проблемой непринятия токена, но этоу проблему можно было проигнорировать и войти уже с новым паролем, после чего предстагается проийти двухфакторную аутентификацию.

![13](image/13.png){ #fig:011 width=70% }  

Для устранения второй уязвимости мы ставим запрет выполнения JavaScript на стороне сервера БД, для этого мы отредактировали файл конфигурации БД /etc/mongod.conf, добавив строчку javascriptEnabled: False

![14](image/14.png){ #fig:012 width=70% }  
![15](image/15.png){ #fig:013 width=70% }  

Готово! Уязвимость и последствия на данный узел устранены

![16](image/16.png){ #fig:014 width=70% }  


## Уязвимость «WPDISCUZ»
Это уязвимость в плагине для создания комментариев WpDiscuz версии с 7.0.0 по 7.0.4 включительно. Уязвимость позволяетполучить (удаленное RCE выполнение кода)

Обнаружив уязвимость, мы отключили плагин WpDiscuz, точнее мы его полностью удалили.

![17](image/17.png){ #fig:015 width=70% }  

Далее нам нужно обновить плагин, мы обновили до версии 7.6.34

![18](image/18.png){ #fig:016 width=70% }  

После этого требуется нейтрализовать последствие, для того необходимо сформировать резервную копию с помощью плагина Updraft Backup/Restore
![19](image/19.png){ #fig:017 width=70% }  

Бинго!

![20](image/20.png){ #fig:018 width=70% }  
![21](image/21.png){ #fig:019 width=70% }  

Для того, чтобы уязвимость устранилась, нам потребоваллсь  устранить последствие  в виде вредоносного соединения.
Через терминал мы вывели информацию об активных соединениях и, соответсвенно, закрыли ненужные, тем самым закрыли вредоносный сокет.

![22](image/22.png){ #fig:020 width=70% }  
![23](image/23.png){ #fig:021 width=70% }  
![24](image/24.png){ #fig:022 width=70% }

# Выводы

В ходе выполнения лабораторной работы:  
- Были выявлены и устранены уязвимости на различные узлы и их последствия.  
- Система приведена в безопасное состояние.


# Список литературы{.unnumbered}

::: {#refs}
:::
