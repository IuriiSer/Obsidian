**Intro to [.jsx](https://reactjs.org/docs/introducing-jsx.html)**

### IF condition
```
{unreadMessages.length > 0 &&        
	<h2>
		You have {unreadMessages.length} unread messages.
	</h2>      
}
```
### Close all tags
```
<meta />
<input />
...
```

### Array mapping
-> **Imp** always use key to map element
```
// with IF -> {urls.length && urls.map...}
{urls.map((url) => (
	<div key={Math.random() * 9999}>
		<a href={url.longLink}>{url.longLink}</a>
		<a href={url.longLink}>http://localhost:3000/{url.shortLink}</a>
		<a href={`http://localhost:3000/${url.shortLink}`}>
		<form action={`localhost:3000/${url.shortLink}/delete`} method='get'>
			<input className='removeButton' type='submit' value='Remove Link' />
		</form>
	</div>
))}
```