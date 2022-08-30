### What Is 
[Nodemon](https://www.npmjs.com/package/nodemon) is a tool that helps develop Node.js based applications by automatically restarting the node application when file changes in the directory are detected.

### Project init
1 -> Init in Console
```
npm i -D nodemon --save-dev

```

### Start Project
2-> Add to **package.json** (don\`t forget change app name)<br>
for express-generator ->
```
"start": "if [[ $NODE_ENV == 'production' ]]; then node ./bin/www; else nodemon ./bin/www --ext js,jsx; fi"
```
for express-react server -> 
```
"start": "if [[ $NODE_ENV == 'production' ]]; then node ./app.js; else nodemon ./app.js --ext js,jsx; fi"
```