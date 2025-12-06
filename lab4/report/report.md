---
## Front matter
title: "Лабораторная работа 4-D"
subtitle: "ЗАХВАТ DNS-СЕРВЕРА"
author: "НФИбд-01-22 (E)"


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
mainfont: IBM Plex Serif
romanfont: IBM Plex Serif
sansfont: IBM Plex Sans
monofont: IBM Plex Mono
mathfont: STIX Two Math
mainfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
romanfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
sansfontoptions: Ligatures=Common,Ligatures=TeX,Scale=MatchLowercase,Scale=0.94
monofontoptions: Scale=MatchLowercase,Scale=0.94,FakeStretch=0.9
mathfontoptions:
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
listingTitle: "Листинг"
lofTitle: "Список иллюстраций"
lolTitle: "Листинги"
## Misc options
indent: true
header-includes:
  - \usepackage{indentfirst}
  - \usepackage{float} # keep figures where there are in the text
  - \floatplacement{figure}{H} # keep figures where there are in the text
---

# Цель работы

Закрепить практические навыки получения доступа к DNS-серверу и нахождения флага в одной из DNS-записей.


# Задание

Получить доступ DNS-СЕРВЕРА и завладеть флагом DNS-записей.

# Выполнение лабораторной работы

## Получение meterpreter-сессии

Получаем meterpreter-сессии с почтовым сервером с помощью модуля exchange_proxyshell_rce

![Параметры модуля ](image/1.jpg){#fig:001 width=70%}

Настраиваем и запускаем meterpreter-сессии с почтовым сервером с помощью 
exchange_proxyshell_rce

![Настройка и запуск ](image/2.jpg){#fig:002 width=70%}

![Настройка и запуск](image/3.jpg){#fig:003 width=70%}

## Получение флага

### Создаем сессию

После получения сессии переходим к процедуре поиска нужного сервера. Выполняем проброс портов во внутреннюю сеть с помощью команды autoroute и запустить данную сеть – run
autoroute -s 10.10.10.0/24

![Регистрация подсети во фреймворке Metasploit](image/4.png){#fig:004 width=70%}

После сворачиваем сессию с помощью команды bg

![Команда bg](image/5.png){#fig:005 width=70%}

После переходим в модуль multi/gather/ping_swee, в котором будем сканировать внутреннюю сеть организации для выбора адреса, который может быть использован для дальнейшей атаки. Далее Проводим
поиск DNS-сервера

![Модуль multi/gather/ping_swee](image/6.png){#fig:006 width=70%}

![Маршрут до внутренней сети](image/7.png){#fig:007 width=70%}

![Завершение поиска](image/8.png){#fig:008 width=70%}

С помощью команды route распечатать маршруты, которые обнаружены Metasploit и возвращаемся в сессию с почтовым сервером (sessions 1)

![Маршруты](image/9.png){#fig:009 width=70%}

Отображаем хосты, которые находятся во внутренней сети организации

![Таблица маршрутизации на хосте MS Exchange](image/10.png){#fig:010 width=70%}


Далее проверяем наличие открытых портов на хостах, которые находятся во внутренней сети организации.
Поскольку сканируемые хосты находятся во внутренней сети, в первую очередь необходимо настроить прокси, через который будут проходить все запросы при сканировании.
Сворачиваем сессию (команда bg) и находим модуль metasploit auxiliary/server/socks_proxy

![Поиск и выбор модуля metasploit auxiliary/server/socks_proxy](image/11.png){#fig:011 width=70%}

Настраиваем модуль в соответствии с параметрами, которые указаны в конфигурационном файле /etc/proxychains.conf

![Конфигурационный файле /etc/proxychains.conf](image/12.png){#fig:012 width=70%}

![Настройки и запуск модуля](image/13.png){#fig:013 width=70%}

Открываем новый терминал kali. В новом терминале запустить сканирование 100 самых часто используемых портов с помощью команды proxychains nmap –n –sT –Pn --top-ports 100 {IP}

![Результаты сканирования портов](image/14.png){#fig:014 width=70%}

По стандарту RFC 1035 все DNS-серверы отвечают на порту 53 TCP и
UDP. По результатам сканирования можно сделать вывод, что узел 10.10.10.15 является целью атаки – DNS-сервером с открытым 22 портом SSH.

![Идентификация DNS-сервера](image/15.png){#fig:015 width=70%}


### Bruteforce пароля

Для реализации атаки перебором паролей использовать словарь rockyou.txt, который находится по пути 
/usr/share/wordlists

![Результаты сканирования портов (2)](image/16.png){#fig:016 width=70%}

Логин пользователя можно получить с помощью файла userlist в директории /usr/share/wordlists с именами пользователей. Выбрать пользователя «user», далее запустить утилиту hydra с помощью команды
proxychains hydra -V -f -l user -P rockyou.txt -t 32 10.10.10.15 ssh

![Запуск атаки перебором](image/17.png){#fig:017 width=70%}

![Результат атаки перебором](image/18.png){#fig:018 width=70%}

Для получения доступа к DNS-серверу подключаем по SSH по полученным учетным данным или модулем
metasploit auxiliary/scanner/ssh/ssh_login с указанием параметров
для входа. Подключение по SSH по полученным учетным данным осуществляем
с помощью команды proxychains ssh user@10.10.10.15 и вводим пароль.

![Подключение по SSH](image/19.png){#fig:019 width=70%}

Для получения флага необходимо вывести содержимое файла /etc/hosts с помощью команды cat /etc/hosts

![Просмотр флага](image/20.png){#fig:020 width=70%}

Введём номер флага во вкладку "Задания" на сайте и увидим, что лабораторная работа успешно выполнена

![Атака совершена](image/21.jpg){#fig:021 width=70%}

# Вывод

В результате лабораторной работы мы получили доуступ DNS-СЕРВЕРА и завледели флагом DNS-записей.

