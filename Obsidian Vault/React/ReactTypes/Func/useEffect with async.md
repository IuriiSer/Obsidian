```
// ...
useEffect(() => {
	(async () => {
		const res = await fetch('http://localhost:3100/api/auth', {
			method: 'GET',
			credentials: 'include',
		});
		const data = await res.json();
		setIsAuth(data.isAuth);
	})();
}, []);
// ...
```