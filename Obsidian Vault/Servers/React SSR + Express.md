### What Is
Advance usage of React SSR and Express<br>
Project will use principes of [MVC](https://towardsdatascience.com/everything-you-need-to-know-about-mvc-architecture-3c827930b4c1#:~:text=%2DMVC%20is%20an%20architectural%20pattern,the%20view%20whenever%20data%20changes.)

### Project Init
-> [[React SSR]]<br>
-> [[Express]] - [docs](https://expressjs.com/)<br>
-> [[Sequelize]] - [docs](https://sequelize.org/docs/v6/)<br>
-> [[Morgan]] - [docs](https://www.npmjs.com/package/morgan)<br>
-> [[Nodemon]] - [docs](https://www.npmjs.com/package/nodemon)<br>
-> a lot of other staff<br>
`npm i` to init all depends<br>
**Imp** -> Please go to [Fontawesome](https://fontawesome.com/) and create an account to use icons in footer<br>

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
| | |->renderTemplate.js <- used to render all pages on server builded on React Components
| |
| |-> /middlewares <- used to keep all middlewares in one place
| |
| |-> /routes
| | |->errorController.js <- routes for ErrorPage
| | |->mainController.js <- routes for MainPage
| |
| |-> /views
| | |-> /Components
| | | |-> Header.jsx <- header module is common for all pages
| | | |-> Layout.jsx <- layout module is common for all pages
| | |
| | |-> /Pages
| | | |-> MainPage.jsx <- Home page view
| | | |-> ErrorPage.jsx <- Error page view (for example 404)
|
|-> .babel.rc <- to use .jsx files
|-> .envTamplate <- rename to .env and change entries for your values
|-> .gitignore <- standart .gitignore file to Node project
|-> .eslintrs.js <- standart .eslintrs config for AirBnb formatter
|-> .sequelizerc <- standart .sequelizerc file
|-> app.js <- Main app
|-> package.json
```

**app.js**<br>
-> add new routes from .src/routes<br>
**/src/routes**<br>
-> add new routes<br>
**/src/controllers**<br>
-> add new controllers for routes<br>
**/src/lib**<br>
-> common funcs of our app<br>
**/src/views**<br>
-> add new Pages/Components templates<br>
**/src/middlewares**<br>
-> add new middlewares for your routes<br>