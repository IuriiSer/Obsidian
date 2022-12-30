### What is
if your column in model keep data in format `JSON.stringify([int1, int2, int3])`
use func below to work fast
### Funcs
#### addValByKeyToUser.js
```
const { User } = require('../../db/models');
const updateCache = require('./updateCache');

/**
* write newVal to key for user and update data in db and cache
* @param {userId}
* @param {key} <- property key
* @param {newId}
* @return {number}
* 500 <- serverError
* 400 <- input is not Number
* 409 <- value is already in array
* 200 <- OK
*/
module.exports = async function addValByKeyToUser(userId, key, _newId) {
	try {
		const newId = Number(_newId); // checking for NaN
		if (Number.isNaN(newId)) { return 400; }
		
		const user = await User.findByPk(userId);
		const array = JSON.parse(user[key]);
		if (array.includes(newId)) { return 409; }
			// check do we have ID in array
		array.push(newId);
		user[key] = JSON.stringify(array);
		await user.save();
		await updateCache(`user?id=${user.id}`, user.dataValues);
			// for Redis
		return 200;
	} catch (error) {
		return 500;
	}
};
```
#### removeValByKeyFromUser.js
```
const { User } = require('../../db/models');
const updateCache = require('./updateCache');
/*
* remove val to key from user and update data in db and cache
* @param {userId}
* @param {key} <- property key
* @param {newId}
* @return {number}
* 500 <- serverError
* 400 <- input is not Number
* 409 <- value is already in array
* 200 <- OK
*/
module.exports = async function removeValByKeyFromUser(userId, key, _idToRemove) {
	try {
		const idToRemove = Number(_idToRemove);
		if (Number.isNaN(idToRemove)) { return 400; }
		
		const user = await User.findByPk(userId);
		const array = JSON.parse(user[key]);
		const indInArray = array.indexOf(idToRemove);
		if (indInArray === -1) { return 409; }
			// check do we have ID in array
		array.splice(indInArray, 1);
		user[key] = JSON.stringify(array);
		await user.save();
		await updateCache(`user?id=${user.id}`, user.dataValues);
			// for Redis
		return 200;
	} catch (error) {
		return 500;
	}
};
```
#### inSomeFunc
```
...
async function removeRecieptFromFavorite(req, res) {
	try {
		const userId = res.locals.user.id;
		const status = 
			await removeValByKeyFromUser(userId, 'favReciepts', req.body.id);
		if (status === 200) return res.sendStatus(200);
		if (status === 409) return res.status(409)
				.json({ err: 'There is no recipe in Favorites' });
		if (status === 400) return res.status(400)
				.json({ err: 'Wrong receipt ID' });
		if (status === 500) return res.sendStatus(500);
	} catch (error) {
		res.sendStatus(500);
	}
}
```