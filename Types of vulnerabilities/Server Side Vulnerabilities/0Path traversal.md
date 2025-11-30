Уязвимость помогает злоумышленнику читать произвольные файлы на сервере такие как
- Код и данные
- Учетные данные
- Конфиденциальные файлы 
пример такой атаки
```
https://insecure-website.com/loadImage?filename=../../etc/passwd #для машин на Linux

https://insecure-website.com/loadImage?filename=..\..\windiws\win.ini
```
дает возможность прочитать любой файл на машине без отсутствия фильтрации
