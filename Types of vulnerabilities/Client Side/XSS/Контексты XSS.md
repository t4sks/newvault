
При тестировании Reflected и Stored XSS ключевая задача это определить контекст возникновения уязвимости.

- Местоположение внутри ответа, где появляются данные, контролируемые атакующим.
- Любая проверка ввода или иная обработка, которую приложение применяет к этим данным

# XSS между HTML тегами
Когда контекст XSS это текст между HTML тегами, вам нужно внедрить новые HTML теги, рассчитанные на запуск JavaScript
Некоторые полезные способы выполнить JavaScript

```
<script>alert(document.domain)</script>
<img src=1 onerror=alert(1)>
```

# Пример: Lab: Reflected XSS into HTML context with most tags and attributes blocked