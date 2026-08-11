![[Pasted image 20260528180930.png]]

Формат отчета: 
Хороший отчет составленный по принципу 5 вопросов

1. Кто: 
2. Что: Какой именно действие или последовательность событий были выполнены
3. Когда: когда именно началась и закончилась подозрительная активность
4. Где: какое устройство, IP адрес или веб сайт были задействованы в оповещении
5. Почему: Самый важный вопрос - обоснование вашего окончательного вывода


NT AUTHORITY\SYSTEM на DMZ-MSEXCHANGE-2013 выполнял команды для повышения привилегий
Было запущен C:\Windows\System32\inetsrv\w3wp.exe который породил C:\Users\Public\revshell.exe и он уже создал cmd.exe
Mar 27th 2025 at 19:56
использовались команды: dir hostname whoami /priv net group "Domain Admins" /domain nltest /dclist:tryhackme.thm

Description:

Detects a spike of commands like whoami, net user, and Get-ADUser, often used during AD domain discovery. Unless the commands are confirmed to be a part of IT activity or legitimate scripts, the device is likely compromised and requires immediate containment.

Invoked Commands:

dir hostname whoami /priv net group "Domain Admins" /domain nltest /dclist:tryhackme.thm

Host Name:

DMZ-MSEXCHANGE-2013

Host OS:

Windows Server 2012 R2

User:

NT AUTHORITY\SYSTEM

Source Process:

C:\Windows\System32\cmd.exe

Parent Process:

C:\Users\Public\revshell.exe

Grandparent Process:

C:\Windows\System32\inetsrv\w3wp.exe


---
# Как понять что инцидент стоит передавать выше
1. оповещение указывает на крупную кибератаку требующую более глубокого расследования
2. Необходимы действия по устранению последствий, такие как удаление вредоносного ПО, изоляция Хоста или сброс пароля, необходима связь с клиентами, партнерами, руководством или правоохранительными органами
3. Вы просто не до конца понимаете оповещение и нуждаетесь в помощи более опытных аналитиков 

---
# Playbook для **Unusual Login Location Workbook** 
![[Pasted image 20260528201947.png]]

---
# EDR - endpoint detection and responce
Три основные функции EDR
![[Pasted image 20260529141818.png]]

- Видимость (Visibility) - обнаружение любых изменений таких как модификации процессов, изменения реестра, изменения файлов и папок, действия пользователя и еще многое другое 
- Детектирование (Detection) - Функция обнаружения EDR превосходит традиционные способы обнаружения на основе сигнатур она так же включает в себя обнаружение на основе поведения
- Ответ (response) - EDR так же позволяет аналитикам применять меры в отношении найденных угроз. Эти действия можно выполнить в любой конечной точки с помощью консоли EDR. 

> [!attention] ВАЖНО!!!!
> EDR - это решение на уровне хоста оно не может детектировать сетевые атаки

Собранная телеметрия:

 - Выполнение и завершение процессов
 - Сетевые подключения
 - все команды в командной строке
 - изменения в файлах и папках
 - изменения в реестре
 
---
SIEM - Security Information and Event managment system

Собирает логи на винде из Event Viewer

Собирает логи на linux из:
- /var/log/httpd - логи http-запросов/ответов и ошибок
- /var/log/cron -  логи cron
- /var/log/auth.log и /var/log/secure - хранят журналы которые связаны с аутентификацией
- /var/log/kern - события связанные с ядром
- /var/log/apache - логи апачи

---
# SOAR
Система которая объединяет все имнструменты сока в одном приложении и позволяет работать только в нем