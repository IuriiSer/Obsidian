### What is [ReactRouter](https://v5.reactrouter.com/web/api/)
React - a single page app. To immitate page changing we should use routes
### How to use
#### Project Init
```
npm install react-router-dom@6

```
#### Init in App
```
import * as React from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

import Context from './lib/context';

import Components from './view/components';
import Pages from './view/pages';

function App() {
  const [isAuth, setIsAuth] = React.useState(false);

  React.useEffect(() => { // get isAut token for example }, []);
	
  const context = React.useMemo(() => ({ isAuth, setIsAuth }), [isAuth]);
  return (
  <Context.Provider value={context}>
	// context should be outer of BrowserRouter to keep data through renders
	
	<BrowserRouter> 
		// <- where will be all links and routes
	  <Components.Header />
		// <- where we will change components on page
		// routes which will be rendere depends on path
	  <Routes>
		<Route path="/"                element={<Pages.Restaurants />} />
		<Route path="/restaraunts/:id" element={<Pages.RestarauntFullInfo />} />
		<Route path="/restaraunts"     element={<Pages.Restaurants />} />
		<Route path="/login"           element={<Pages.Login />} />
	  </Routes>
	  
	  <Footer />
	</BrowserRouter>
	
  </Context.Provider>
  );
}

export default App;
```
#### Init Links
```
import { Link } from 'react-router-dom';

...
<Link to="/">Home</Link>
<Link to="/some-page">Home</Link>
<Link to={location => `${location.pathname}?sort=name`} />

<Link
  to={{
    pathname: "/courses",
    search: "?sort=name",
    hash: "#the-hash",
    state: { fromDashboard: true }
  }}
/>
```
```

```
#### Redirects
```
const { isAuth, setIsAuth } = React.useContext(Context);
const location = useLocation();
  
export default function LoginComponent() {
  const { isAuth, setIsAuth } = React.useContext(Context);
  const location = useLocation();
  
  const handleSubmit = (event) => {
	// some handle
	setIsAuth(true);
  });

  retrun (
  <>
	{isAuth && <Navigate to="/" from={{ location }} />}
		// navigate to another page

	<Container component="main" maxWidth="xs">
		// other Staff
	</Container>
  );
};
```