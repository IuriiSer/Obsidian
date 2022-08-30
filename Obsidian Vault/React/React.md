### What Is 

**React в тезисах** <br>
-> React - JS-библиотека для отрисовки пользовательских интерфейсов (user interfaces, UI)<br>
-> позволяет вставлять данные в заранее подготовленные HTML шаблоны<br>
-> JSX - позволяет писать HTML прямо в js-файлах<br>
-> шаблонизаторы отделяют View от всего остального<br>
-> слой View в [[MVC]]<br>
[[React SSR]]
### Common
#### Require
[[Babel React]] -> [[jsx Syntax]]

#### Syntax
[[React Components]]

### Eslint
To install [Eslint](https://www.npmjs.com/package/eslint-config-react-app) use 
```
npm install --save-dev eslint-config-react-app eslint@^8.0.0

```
Then create a file named `.eslintrc.json` with following contents in the root folder of your project:
```
extends: [
	'react-app',
	'plugin:react/recommended',
	'airbnb',
],
rules: { 'react/jsx-filename-extension': [1, { extensions: ['.js', '.jsx'] }], },
```