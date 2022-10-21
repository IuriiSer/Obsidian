### What Is 
**Babel** <br>
[Babel](https://babeljs.io/docs/en/babel-preset-react) позволяет подключать jsx файлы (JavaScript, в котором можно писать HTML).
Чтобы подключить Babel в главном js-файле сверху нужно добавить строчку ->
`require('@babel/register');`

### Project init
1 -> Init in Console
```
npm install @babel/core @babel/preset-env @babel/preset-react
@babel/register react react-dom

```

2 -> Adding 
-> in .babelrc
```
{ "presets": ["@babel/preset-env", "@babel/preset-react"] }
```
-> in app.js
```
require('@babel/register');
```