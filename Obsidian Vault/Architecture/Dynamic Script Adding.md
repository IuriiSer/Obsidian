### What Is 
Чтобы не подключать все скрипты за раз, мы можем подключать их **непосредственно** перед их **выполнением**

### WithOut Principe
-> for MainPage
```
<head>
	<meta charSet="UTF-8" />
	<meta httpEquiv="X-UA-Compatible" content="ie=edge" />
	...
	<script defer src="/js/components/commonLib.js" />
	<script defer src="/js/components/toDoFormHandler.js" />
	<script defer src="/js/components/toDoFormCreator.js" />
	<script defer src="/js/pages/mainPage.js" />
	<script defer src="/js/pages/toDoPage.js" />
	<script defer src="/js/init.js" />
	<title>{title}</title>
	...
</head>
```
### With Principe
```
<head>
	<script defer src="/js/components/commonLib.js" />
	<script defer src="/js/pages/mainPage.js" />
	<script defer src="/js/pages/toDoPage.js" />
	<title>{title}</title>
	...
</head>
```
** toDoPage.js **
```
-> include
	<script defer src="/js/components/toDoFormHandler.js" />
	<script defer src="/js/components/toDoFormCreator.js" />
```
** mainPage.js **
```
-> include
	<script defer src="/js/components/someScript_1.js" />
	<script defer src="/js/components/someScript_2.js" />
```
### How To Use
**Best Practise for [[Page Constructor]]**<br>
**For Array** and **Single** vals
-> Don\'t forget to check -> did we mount scripts!
```
onReady = (scriptsPromises) => Promise.all(scriptsPromises);

static isScriptMounted(src) {
	return !!document.querySelector(`script[src="${src}"]`);
}

function addScripts(...srces) {
	const scriptsPromises = [];
	srces.flat().forEach((src) => {
		if (Page.isScriptMounted(src)) return;
		const scriptPromise = new Promise((resolve, reject) => {
			const script = document.createElement('script');
			script.type = 'text/javascript';
			script.src = src;
			script.onload = resolve;
			script.onerror = reject;
			script.async = true;
			document.head.appendChild(script);
		});
		this.#scriptsPromises.push(scriptPromise);
	});
	return scriptsPromises;
}


```
-> After use `Promise.all` method to call callback func
```
const awaitingScripts = addScripts(src1, src2, src3);
await onReady(awaitingScripts);
... your script here
```