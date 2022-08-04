### What Is 
[**AJAX**](https://www.w3schools.com/js/js_api_fetch.asp) -> **A**synchronous **J**avaScript **A**nd **X**ML<br>
AJAX is a developer's dream, because you can:<br>
->   Read data from a web server - after a web page has loaded<br>
->  Update a web page without reloading the page<br>
->   Send data to a web server - in the background<br>
[**Fetch API**](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) -> The Fetch API provides an interface for fetching resources (including across the network). It will seem familiar to anyone who has used [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest), but the new API provides a more powerful and flexible feature set.
### How to use
-> use in async/await<br>
-> always check req before USAGE <br>

#### Base Syntax
**Imp** -> ALWAYS check status after fetch()
```
// -> on client
async func () {
	const req = await fetch(
		someSource,
		{
			method,   // -> POST/GET/DELETE
			headers:  // -> all types in Advanse in below guide part
				{ 'Content-type': 'application/json' },
			body: ,   // -> put data in your format
				JSON.stringify({ title })
		}
	)

	-> there ypu should check req.status <-

	// if -> you await Obj
	const res = await req.json()

	// if -> you wait StatusJob
	use -> req
}
```
```
// -> on server
router.method(someSource, someCallBack)

async func someCallBack(req, res) {
	try {
		some async funcs...
		conditions...
		and checks...
		
		or res.sedStatus(someGoodStatus)
		or res.json({ someData })
	} catch(error) {
		res.sedStatus(someBadStatus)
	}
}
```
#### Advance Syntax 
-> [MediaTypes](https://www.iana.org/assignments/media-types/media-types.xhtml)<br>
-> [List of HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)<br>

### Example
#### Client side
**Important** USE event.preventDefault() on **client**!
```
async function voteWrapper(formElem) {
	try {
		const request = await fetch(
			`/posts/${formElem.id}/vote`, { method: 'POST' });
		if (request.status !== 200) 
			throw new Error('voteWrapper error -> ', response.status);
		...
		
	} catch (error) {
		throw new Error(error);
	}
}
  
// -> !!! most important example

async function createPost (someData) => {
	try {
		// turn off any other handler
		postCreateEvent.preventDefault();
		
		// fetch data
		const request = fetch('/posts', {
			method: 'POST', <- method post/get/delete/...
			headers: { 'Content-type': 'application/json' },
			body: JSON.stringify({ someData }),
		});
		
		// checking requests
		if (request.status !== 200) {
			if (request.status === 400) {
				alert('form must be not empty');
				throw new Error('can not add empty form -> ', response.status);
			}
			throw new Error('creatingPost error -> ', response.status);
		}
		
		// after we can parse data
		const postData = await response.json();
		
		...
	} catch (error) {
		// catching other errors
		throw new Error(error);
	}
});
```
#### Server side
```
// -> one of Router files
const router = express.Router();

router.post('/posts/:id/vote', async (req, res) => {
	try {
		// -> some ASYNC func
		res.sendStatus(200);
	} catch (error) {
		res.sendStatus(500);
	}
});

router.delete('/posts/:id/delete', async (req, res) => {
	try {
		// -> some ASYNC func
		res.sendStatus(200);
	} catch (error) {
		res.sendStatus(500);
	}
});

router.post('/posts', async (req, res) => {
	const { title } = req.body;
	// some CHECK for input entrys
	if (!title) {
		res.sendStatus(400);
		return;
	}
	// async function
	const newPost = await Post.create({ 
		title, username: 'User',
		commentCount: Math.floor(Math.random() * 1000)
	});
	// send data to client
	res.json(newPost); // -> newPost = { key: val }
});

module.exports = router;
```
