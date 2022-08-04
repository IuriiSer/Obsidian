### What Is
Advance usage of React SSR and Express<br>
Project #ServerTemplate will use principes of [[MVC]]

### Project Init
#### How to Install
to download -> `git clone git@github.com:IuriiSer/Express_React_SSR_projectTemplate.git`<br>
to init all depends -> `npm i` 
#### Main Depends
-> [[React SSR]]<br>
-> [[Express]]<br>
-> [[ORMS]] -> Sequelize for example<br>
-> [[Middlewares]] -> Morgan / Nodemon<br>
-> [[Environment]]<br>
-> a lot of other staff<br>

#### All Depends
```
"dependencies": {
	"@babel/core": "^7.17.10",
	"@babel/preset-env": "^7.17.10",
	"@babel/preset-react": "^7.16.7",
	"@babel/register": "^7.17.7",
	"debug": "^4.1.1",
	"express": "^4.17.1",
	"http-errors": "^1.8.0",
	"morgan": "^1.10.0",
	"pg": "^8.7.3",
	"pg-hstore": "^2.3.4",
	"react": "^18.1.0",
	"react-dom": "^18.1.0",
	"sequelize": "^6.21.3",
	"sequelize-cli": "^6.4.1"
},

"devDependencies": {
	"eslint": "^8.14.0",
	"eslint-config-airbnb": "^19.0.4",
	"eslint-config-airbnb-base": "^15.0.0",
	"eslint-plugin-import": "^2.26.0",
	"eslint-plugin-jsx-a11y": "^6.5.1",
	"eslint-plugin-react": "^7.29.4",
	"eslint-plugin-react-hooks": "^4.4.0",
	"nodemon": "^2.0.7"
}
```

### Project Structure
```
/
|-> /db <- Based on Sequelize
| |->/migrations
| |->/models
| |->/seeders
| |->database.json <- change user/pass and dbName
|
|-> /public <- Client side files
| |->/js
| |->/css
|-> /routes <- Routes for our Server by pages
| |->main.js
|
|-> /src
| |-> /controllers
| | |->errorController.js <- controller for Errors
| | |->mainController.js <- controller for MainPage
| |
| |-> /lib
| | |->renderTemplate.js <- used to render pages SSR builded on React Components
| |
| |-> /middlewares <- used to keep all middlewares in one place
| |
| |-> /routes
| | |->errorController.js <- routes for ErrorPage
| | |->mainController.js <- routes for MainPage
| |
| |-> /views
| | |->Layout.jsx <- Main Layout of our App meta/header/footer
| | |->Main.jsx <- module for MainPage
| | |->Error.jsx <- module for ErrorPage
|
|-> .babel.rc <- to use .jsx files
|-> .gitignore <- standart .gitignore file to Node project
|-> .eslintrs.js <- standart .eslintrs config for AirBnb formatter
|-> .sequelizerc <- standart .sequelizerc file
|-> app.js <- Main app
|-> package.json
```

**app.js**
-> add new routes from .src/routes

**/src/routes**
-> add new routes

**/src/controllers**
-> add new controllers for routes

**/src/lib**
-> common funcs of our app

**/src/views**
-> add new pages/modules templates

