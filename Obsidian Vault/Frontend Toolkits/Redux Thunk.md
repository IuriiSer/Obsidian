### What is
In simple redux you have problem: we need make handlers for all actions in differnt files. Redux Thunk is **a middleware that lets you call action creators that return a function instead of an action object**. That allow us to make good structure of project
### Project init
### index.js
**index.js**
```
import { legacy_createStore as createStore, applyMiddleware, combineReducers } from 'redux';

import { composeWithDevTools } from 'redux-devtools-extension';
	// for dev and catch current redux state in browser in dev tools
import thunkMiddleware from 'redux-thunk';
	// join middleware
	
import userReducers from './user/userReducers';

import loadingReducers from './loading/loadingReducers';
import toDoReducers from './toDo/toDoReducers';
		// refucers for all parts of store
	
import { getAuthStatus } from './user/userActions';

const reducers = combineReducers({
	isLoading: loadingReducers,
	user: userReducers,
	toDo: toDoReducers,
});

const composeEnhancer = process.env.NODE_ENV === 'production'
	? applyMiddleware(thunkMiddleware)
	: composeWithDevTools(applyMiddleware(thunkMiddleware)); 
		// здесь у нас dev-режим

const store = createStore(reducers, composeEnhancer);

export default store;

store.dispatch(getAuthStatus());
```
### part of store /user
**user is for example**
#### /user/userActions.js
```
import userActionsTypes from './userTypes';
import { parseErr, setErr } from '../error/errorActions';
import { setLoading, resetLoading } from '../loading/loadingActions';

// standart funcs to set data in redux
export const setData = 
	(user) => ({ type: userActionsTypes.SET_DATA, payload: { user } });
export const resetData = 
	() => ({ type: userActionsTypes.RESET_DATA });

// func with Thunk
export const signIn = (loginData) => async (dispatch) => {
	const isLoginRequest = !loginData.passwordRepeat;
	dispatch(setData(isLoginRequest));
	...
	
}
```
#### /user/userReducer.js
default reducer but only for user
```
/* eslint-disable default-param-last */
import userActionsTypes from './userTypes';

const initState = {
	isResGetted: false,
	id: null,
	email: null,
};

export default function userReducers(state = initState, action) {
	switch (action.type) {
		
		case userActionsTypes.SET_DATA:
			return { ...state, isResGetted: true, ...action.payload.user };
		
		case userActionsTypes.RESET_DATA:
			return { ...state, ...initState, isResGetted: true };
		
		default:
			return state;
	}
}
```
#### /user/userTypes.js
default action but only for user
```
const userActionsTypes = {
	SET_DATA: 'SET_DATA',
	RESET_DATA: 'RESET_DATA',
};

export default userActionsTypes;
```
### how to use in project
```
export default function RegLogForm({ isLogForm }) {
	const dispatch = useDispatch();
	// some action handler
	const submitHandler = (e) => {
		e.preventDefault();
		const rawData = e.target;
		const data = {
			email: rawData.email.value,
			password: rawData.password.value,
			passwordRepeat: isLogForm ? null : rawData.passwordRepeat.value,
		};
		// set dispatch with middleware
		dispatch(signIn(data));
	};
}
```