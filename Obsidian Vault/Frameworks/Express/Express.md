### What Is 
[Express](http://expressjs.com/en/5x/api.html#express) - fast, unopinionated, minimalist web framework for [[Node.js]]

### Project init
1 -> Init in Console
```
npm install express --save

```
opt -> #Middlewares

### Start Project
#### Init app.js
2 -> Creating app.js
```
const express = require('express');
const app = express();

// this place for midlware modules
// -> Morgan for example

app.use(express.static('public'));

app.use(express.urlencoded({ extended: true })); 
	// читать данные из тела запросов <- POST request
app.use(express.json());
	// читать JSON-данные из тела запросов

// this place for routes
// -> some Routes

app.listen(3000, () => { console.log('Server started') });
```

### Routes
**-> GET <-** data in **req.query**
Use to send data in address line (query). Google seqrch for example
```
//address -> .../?key=someVal
app.get('/test', function (req, res) {
	const data = req.query; // ! <- req.query = { key: 'someVal' }
	res.send(JSON.stringify(data));
});
```
```
//address -> .../someVal
app.get('/:key', function (req, res) {
	const data = req.params; // ! <- req.params = { key: 'someVal' }
	res.send(JSON.stringify(data));
});
```

**-> POST <-** data in **req.body**
Use to send data directly. Uses in different forms like login form.<br>
**The main idea** -> hide information you want to sent
```
app.use(express.static('public'));
app.use(express.urlencoded({ extended: false }));
// extended` is `false` -> can be a string or array 
// extended` is `true` -> any type 

app.post('/test', function (req, res) {
	const data = req.body;
	res.send(JSON.stringify(req.body));
});
```

#### Request answers
```
app.get('/test', function (req, res) {
	const data = req.query;
	...
});

res.send(text) 
	// послать текст с кодом 200 + завершить ответ
res.json({ user: 'tobi' }) 
	// послать json с кодом 200 + завершить ответ
res.end() 
	// завершить ответ
res.status(403) 
	// установить код, но НЕ завершить ответ

res.status(500).json({ error: 'message' })
	// установить код 500, послать json + завершить ответ
res.status(404).end() 
	// установить код + завершить ответ
res.redirect('/other-route') 
	// переадресовать клиента + завершить ответ
res.render(view, {data}) 
	// рендер шаблона + послать html + завершить ответ
```

### Local Data
`app.locals` <- значение, привязанное **ко всей жизни** всего объекта app<br>
app.locals мы **можем достать** в любой ручке
через объект<br> `req.app.locals` или <br>`res.app.locals`
**Important**
Alive time -> for one page
### MiddleWares
**Standart** GET request
```
app.get('/', routerWrapper)
```
**With MiddleWares** GET request
```
app.use('/', middleWaresChecker)
app.get('/', routerWrapper)

app.use('/somePage', middleWaresChecker)
app.get('/somePage', routerWrapper)

```
**MiddleWares FUNC example**
```
const middlewareFunc = function (res, req, next){
	if(condition) {
		next();
		return;
	}
	res.redirect('/');
}
```
#### Example
```
// from -> app.js -> before ALL
app.use((req, res, next) => {
	req.isAdmin = Math.round(Math.random());
	next();
});
```
```
// from -> app.js
app.get('/secret', adminCheck, (req, res) => {
	console.log('=====>', req.isAdmin);
	renderTemplate(Secret, null, res);
});

```
```

// from -> ./middlewares/adminCheck
const adminCheck = (req, res, next) => {
	const { isAdmin } = req;
	if (isAdmin) {
		next();
	} else {
		res.redirect('/')
	}
}
module.exports = { adminCheck }
```
### Architecture
**app.js** -> adding routes
```
const express = require('express');

const app = express();
const PORT = 3000;

app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(express.static('public'));

// Join Routes
const mainPage = require('./src/routes/mainRouter');
const errorPage = require('./src/routes/errorRouter');

app.use('/', mainPage);
app.use('/', errorPage);

app.listen(PORT);
```
**/routes/router.js**
```
const express = require('express');

// adding Controllers
const { renderMain } = require('../controllers/mainController');
const router = express.Router();

// rout address -> '/' with controller -> 'renderMain'
router.get('/', renderMain);

module.exports = router;
```
**/controllers/controller.js**
```
// adding View layer from MVC
const renderTemplate = require('../lib/renderTemplate');
// adding page template
const MainPage = require('../views/MainPage');

// write what to do for controller
const renderMain = async (req, res) => {
	renderTemplate(MainPage, { title: 'Some project' }, res);
};

  

module.exports = { renderMain };
```


### [[Sessions and Cookies]]
### [[WebSockets]]