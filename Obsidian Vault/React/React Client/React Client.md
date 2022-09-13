[[Client-rendered Application]] <- Idea<br>
[[Frontend Toolkits]] <- UI Biblio<br>
[[ReactRouter]] <- Make MultiPage app<br>
[[Tips]]<br>
### First Steps
#### Init project
```
npx create-react-app app-name
```

#### Eslint
To install [Eslint](https://www.npmjs.com/package/eslint-config-react-app) use 
```
npm install --save-dev eslint-config-react-app eslint@^8.0.0
npm init @eslint/config

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