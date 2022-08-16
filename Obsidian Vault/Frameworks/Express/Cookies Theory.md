### What Is
-> Информация в виде ключ-значение, хранится в браузере  
-> Привязана к домену  
-> Бывают HTTP-only, недоступные для JS
### How to use
#### On server
##### Init Cookie on Server
```
res.cookie('testCookie', Math.floor((Math.random() * 100)), {  
maxAge: 900000, httpOnly: true });
```
##### Reading on Server
```
const cookieParser = require('cookie-parser’)  
...  
app.use(cookieParser());
console.log(req.cookies.testCookie);
```
#### On client
##### Reading on Client
**Важно: http-only не прочитаются**
```
console.log(document.cookie)
```
##### Init on Client
```
document.cookie = 'testCookie=123'  
Не изменяет другие cookie, только устанавливает testCookie.  
Есть дополнительные опции:  
-> document.cookie = "cookiename=cookievalue; expires= Thu, 21 Aug 2014 20:00:00 UTC; path=/ "
```
#### Func to fast write cookies
```
function setCookie(cname, cvalue, exdays) {  
  const d = new Date();  
  d.setTime(d.getTime() + (exdays * 24 * 60 * 60 * 1000));  
  let expires = "expires="+d.toUTCString();  
  document.cookie = cname + "=" + cvalue + ";" + expires + ";path=/";  
}
```