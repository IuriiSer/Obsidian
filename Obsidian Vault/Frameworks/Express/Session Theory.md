### What Is
Можно хранить на сервере данные, связанные с  определённым пользователем  Связь обычно происходит по id сессии, случайной строке,  хранящейся в cookie
### How to use
```
let session = require('express-session’)  
let sessionConfig = {  
	secret: 'keyboard cat',  
	cookie: {},  
	resave: false,  
	saveUninitialized: true,  
}  
...  
app.use(session(sessionConfig))
```
-> Появляется объект req.session, который сохраняется между запросами  
```
app.get('/', function (req, res) {  
	if(req.session.count) {  
		req.session.count++;  
	} else {  
		req.session.count = 1;  
	}  
	res.send(JSON.stringify(req.session));  
});
```