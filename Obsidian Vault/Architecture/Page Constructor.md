### What Is 
This is a simple realisation of [[Client-rendered Application]] based on vanilla JS. Includes idea of [[Dynamic Script Adding]]
### How to use
#### 3 main scripts and Structure
-> main Page Class script<br>
-> some Page example<br>
-> init script<br>
```
/public
|-> /Pages
| |-> Page.js
| |-> SomePage.js
|
|-> /Components
| |-> SomePageComponent_1.js
| |-> SomePageComponent_2.js
| |-> Init.js
```
#### Base Page Class script
-> constructor for all pages<br>
##### Includes
-> add scripts based on Promises <br>
-> onReady check
-> change path withOut reloading
```
// Main constructor of all Pages
class Page {
	#scriptsPromises = []; // <- to keep all promises of scripts
	
	/**
	*
	* @param {[srces]/src/null} srces <- in format '/js/script_src'
	*/
	constructor(srces) {
	if (!srces) { this.#scriptsPromises = [Promise.resolve()]; return; }
	if (Array.isArray(srces)) { this.#addScripts(srces); return; }
	this.#addScripts([srces]);
	}
	 
	/**
	* return Promise.all for all awaiting scripts
	* @returns {Promise}
	*/
	onReady = () => Promise.all(this.#scriptsPromises);
	
	/**
	* adding scripts loaders to the awaiting array
	* @param {[srces]} srces <- only Array of srces in format '/js/script_src'
	*/
	#addScripts(srces) {
		srces.forEach((src) => {
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
	}
	 
	/**
	* check did we already mount script on the site
	* @param {src} src
	* @returns {true/false}
	*/
	static isScriptMounted(src) {
		return !!document.querySelector(`script[src="${src}"]`);
	}
	 
	/**
	* change current url for new without page reload, add statement in history
	* @param {string} title
	* @param {string} newUrl
	* @param {{statw_obj}} state
	*/
	static changePathWithOutReload(title, newUrl, state = {}) {
		window.history.pushState(state, title, newUrl);
	}

}
```
#### SomePage based on Page Class Example
```
class ToDoPage extends Page {

	#mountPoint; // <- place where we will draw content

	constructor(mountSource, scriptsSource) {
		super(scriptsSource);
		this.#mountPoint = mountSource;
	}

	/*
	* 1 -> change path/title of the page to new values
	* 2 -> render after page is download and add all depends scripts
	*/
	async render() {
		Page.changePathWithOutReload('Create new Todo', '/todo');
		await this.onReady();
		this.#mountPoint.innerHTML = toDoFormCreator();
		this.init();
	}

	/*
	* Init scripts if necessary
	*/
	init() {
		toDoFormHandler();
	}
}


// init new page
const toDoPage = 
	new ToDoPage(
		document.getElementById('main-container'), // mount point
		[
		'/js/components/toDoFormCreator.js',       // all scripts we need
		'/js/components/toDoFormHandler.js'
		]
	);
```
#### Init page
if we go to the some path **directory** -> **we must Init all scripts by hands**<br>
```
// This script initializes the necessary pages, in case of direct access to them
(async () => {
	const pages = {  // <- obj with all pages we need to init
		'/todo': toDoPage,
	};

	const { pathname } = window.location;
	if (!pages[pathname]) return;
	await pages[pathname].onReady();
	pages[pathname].init();
})();
```