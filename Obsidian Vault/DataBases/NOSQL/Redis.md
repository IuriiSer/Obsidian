## Wtat is [Redis](https://www.npmjs.com/package/redis)
The open source, in-memory data store used by millions of developers as a database, cache, streaming engine, and message broker.
## Instalation
```
sudo apt-get install redis
```
To check redis-server -> `redis-server`<br>
To open redis app -> `redis-cli`<br>
To quit redis app -> `quit`<br>
### Using in termainal
**[guide](https://www.youtube.com/watch?v=jgpVdJB2sKQ&list=PLGw0XpvyEcJuHmUcRaSR8V7S7bVK6UKGg)** on youtube<br>
**after `redis-cli`**<br>
#### Common
`SET key value [...]` <- set key with value <br>
`GET key` <- get key <br>
`DEL key` <- delete key <br>
`EXISTS key [..]` <- show do redis have values on key  ( 1 <- true / 1 <- false )<br>
`KEYS pattern` <- retrun all keys for pattern <br>
`flushall` <- erase all data <br>
`ttl key` <- show remaining life time
#### Expiration time
`EXPIRE key aliveTime` <- set  remaining life time in **seconds**<br>
`SETEX key aliveTime value` <- set  remaining life time<br>
#### Arrays
`l/rpush listName values` <- set new array with values `lpush` <-push in the end and `rpush` <- push before all vals <br>
`lrange listName statInd endIndex` <- get values in list between indexes 0 -1 to get all vals <br>
`LPOP` <- pop from end val from array<br>
`RPOP` <- pop from begin val from array<br>
#### Lists
`SADD setName val` <- add in set new val<br>
`SMEMBERS setName` <- get all vals from set<br>
`SREM setName val` <- remove val from set<br>
#### Hashes
`HSET objName key val` <- set key and val to obj<br>
`HGET objName key` <- get val on key from obj<br>
`HDEL objName key` <- delete key from obj<br>
`HGETALL objName` <- get all keys from obj<br>
`HEXISTS objName key` <- set exist time for key<br>
## Using in project
### Instalation
```
npm i redis --save
```
### Usage
**Before all**
```
const Redis = require('redis');
const redisClient = Redis.createClient({ props });

const CACHE_DEFAULT_ALIVE
```
**props** -> url of redis-server for example
#### Simple usage
```
function getPhoto() {
	redisClient.get('photos', async (err, photos) => {
		if (err) { console.log(err); return; }
		if (photos !== null) { res.json(JSON.parse(photos)); return; }
		const data = ... getting data
		redisClient.SETEX('photos', CACHE_DEFAULT_ALIVE, JSON.strinfify(data))
		res.json(data);
	})

}
```
### Advance
#### Work with 1 value
```
require('dotenv').config();
const Redis = require('redis');

const { CACHE_DEFAULT_ALIVE } = process.env;

module.exports = async function getSetCache(key, callbackFunc) {
	try {
		const redisClient = Redis.createClient({});
		await redisClient.connect(); // before getting data make connect to redis
		const data = await redisClient.get(key);
		if (data) {
			await redisClient.quit();
			return JSON.parse(data);
		}
		const newData = await callbackFunc();
		redisClient.SETEX(key, CACHE_DEFAULT_ALIVE, JSON.stringify(newData));
		await redisClient.quit(); // after work we must close connection
		return newData;
	} catch (error) {
		console.log(error);
		return null;
	}
};

function getPhoto() {
	const photos = await getSetCache('photos', async () => {
		const data = ... getting data
		return data;
	})
	res.json(photos);
}
```
#### Work with array
```
module.exports = async function getSetCacheMulti(keys, callbackFunc) {
	const res = [];
	const lastKey = await keys.reduce(async (memo, key, ind) => {
		try {
			await memo;
			if (ind !== 0) res.push(memo);
			return await callbackFunc(key);
		} catch (error) {
			return null;
		}
	}, Promise.resolve());
	
	res.push(lastKey);
	// res -> [Promise, Promise, ... , Value]
	// thet because we use Promise.all(res)
	const dataToRreturn = await Promise.all(res);
	if (dataToRreturn.length === 1 && !dataToRreturn[0]) return [];
	return dataToRreturn;
};
```