### Init
### Create Context Tamplate
/src/lib/Context.js
```
import React from 'react';

export default React.createContext({});
```
### Join Context where you need
```
import * as React from 'react';
import Context from './lib/context';

export default function App() {
	
	const [isAuth, setIsAuth] = React.useState(false);
	// we need to useMemo Hook to know when we shood rerender a page
	const context = React.useMemo(() => ({ isAuth, setIsAuth }), [isAuth]);
	
	return (
		<Context.Provider value={context}>
		// ...
		</Context.Provider>
	);
}
```
### Take Context
```
import Context from '../../lib/context';

// ... some func
const { isAuth } = React.useContext(Context);
// ...
```