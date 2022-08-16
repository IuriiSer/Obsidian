### What Is 
Interaction [[Cookies Theory]] and [[Session Theory]] fo #LogIn and #RegIn system<br>
### Best Practice
-> use middlewares to **protect** hands
-> use middlewares to check sessions
### Project Init
```
npm i express-session --save

```
#### App.js
```
// -> init all biblio
const session = require('express-session');
const sessionsConfig = require('./sessionsConfig');
app.use(session(sessionsConfig));

// -> using middleware to check user by cookies
// -> use BEFORE all routes
const sesionHandler = require('./src/middlewares/sesionHandler');
app.use(sesionHandler);
```
#### sessionConfig
```
require('dotenv').config();

const { SESSION_SECRET_DIMA } = process.env;
const session = require('express-session');
const FileStore = require('session-file-store')(session);

const sessionsConfig = {
	// * Название куки
	name: 'login', 
	// * подключение стора (БД для куки) для хранения
	store: new FileStore(), 
	// * ключ для шифрования куки
	secret: SESSION_SECRET_DIMA, 
	// * если true, пересохраняет сессию, даже если она не поменялась
	resave: false, 
	// * Если false, куки появляются только при установке req.session
	saveUninitialized: false, 
	cookie: {
		maxAge: 1000 * 60 * 60 * 24 * 10, // * время жизни в ms (10 дней)
		httpOnly: true, // * куки только по http
	},
};

module.exports = sessionsConfig;
```
#### /middlewares/sesionHandler.js
```
// if we recieve cookie -> try to parse it and get info about User
// -> write it to lacals if OK
const sesionHandler = (req, res, next) => {
	const user = req.session?.user; // <- tale all info from session
	res.locals.eventEmitter.emit('sesionHandler', req.session?.user.id);
	if (!user) { next(); return; }
		res.locals.user = user;
	next();
};

module.exports = sesionHandler;
```
#### LogIn / RegIn <- parts of script
```
...
const bcrypt = require('bcrypt');
const { Op } = require('sequelize');
const { User } = require('../../db/models');

async function loginUser(req, res) {
	try {
		// -> get info from req
		const { user: { email, password } } = req.body;
		// checking in db -> do we have this user
		const user = await User.findOne({ where: { email } });
		const passCheck = await bcrypt.compare(password, user.password);
		// checking for password
		if (!passCheck) { res.status(400).redirect('/login'); return; }
		// if OK -> create new session
		req.session.user = user; 
			// <- you can put in seassion any information you want
		// after session.save() -> make all you want
		req.session.save(() => res.redirect('/'));
	} catch (error) {
		renderPage(Error, { message: 'Не удалось залогиниться.', error: {} }, res);
	}
}

async function registrNewUser(req, res) {
	try {
		// -> get info from req
		const { user: { email, password, nickName } } = req.body;
		// checking in db -> do we have an user with info 
		const findedUser = 
			await User.findOne({ where: { [Op.or]: [{ nickName }, { email }] } });
		// if yes -> sendStatus BAD
		if (findedUser) { res.status(400).redirect('/users/registrate'); return; }
		// if OK -> create newUser
		const hash = await bcrypt.hash(password, 10);
		const newUser = await User.create({ email, password: hash, nickName });
		// create session with newUser
		req.session.user = newUser;
			// <- you can put in seassion any information you want
		// after session.save() -> make all you want
		req.session.save(() => res.redirect('/'));
	} catch (error) {
		renderPage(Error, {}, res);
	}
}
```
#### Logout 
**не забудь удалить куки на клиенте**
```
async function userLogOut(req, res) {
	req.session.destroy(() => {
		res.clearCookie('OwlCookie');
		res.redirect('/');
	});
}
```
