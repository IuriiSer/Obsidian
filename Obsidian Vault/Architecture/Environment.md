### What Is 
 [Dotenv](https://www.freecodecamp.org/news/how-to-use-node-environment-variables-with-a-dotenv-file-for-node-js-and-npm/#:~:text=DotEnv%20is%20a%20lightweight%20npm,this%3A%20require('dotenv').) -> lightweight npm package that automatically loads environment variables from a .env file into the process.env object

### Project init
1 -> Init in Console
```
npm install dotenv --save

```

2 -> Create and fill .env file
```
# Environment variables.
STATUS=development 
DEV_PORT  = 7000 #Development port 
PROD_PORT = 8000 #Production port 

#DB CONFIG FOR DEVELOPMENT
DB_HOST     = db.host 
DB_USERNAME = root 
DB_PASSWORD = db.password 
DB_DATABASE = db.name 
DB_DIALECT  = postgres
```

### How to use
-> in app.js
```
require('dotenv').config();

const PORT =
	process.env.NODE_ENV === 'production'
	? process.env.PROD_PORT
	: process.env.DEV_PORT;
```