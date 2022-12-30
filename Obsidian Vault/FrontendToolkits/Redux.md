### What Is 
Fast cache storage for Web Browse<br>
[[Redux Thunk]]<br>

### Init in Project
#### Storage
 **/storage** <br>
1-> Create storage arch<br>
a-> **/reducers.js**<br>
```
/* eslint-disable default-param-last */
import { userActionsTypes, noteBookActionsTypes } from './types';

const initState = {
user: { isChecked: false, id: null, email: null },
noteBooks: {},
notes: {},
};

export default function reducers(state = initState, action) {
	
	switch (action.type) {
	case userActionsTypes.SET_SIGN_IN:
		return { ...state, user: action.payload.userInfo };
	case userActionsTypes.SET_SIGN_OUT:
		return { ...state, user: initState.user };
	case noteBookActionsTypes.UPDATE:
		return { ...state, noteBooks: action.payload.noteBooks };
	default:
		return state;
	}
}
```

b-> **/index.js**<br>
```
import { legacy_createStore as createStore } from 'redux';
import { composeWithDevTools } from 'redux-devtools-extension';
import reducers from './reducers';

const composeEnhancers = composeWithDevTools();
const store = createStore(reducers, composeEnhancers);

export default store;
```

c-> **/types.js**
```
export const userActionsTypes = {
	SET_SIGN_IN: 'SET_SIGN_IN',
	SET_SIGN_OUT: 'SET_SIGN_OUT',
};

export const noteBookActionsTypes = {
	UPDATE: 'UPDATE',
};
```
#### Client
d-> **app.js**
```
import { Provider } from 'react-redux';
import store from './store';
...

retrun() {
	<Provider store={store}>
		...
	</Provider>
}
```
e-> **in any place**
```
import { useSelector } from 'react-redux';

export default function Loading() {
	
	const isPageLoading = 
		useSelector((store) => store.isLoading.isPageLoading);
...
```