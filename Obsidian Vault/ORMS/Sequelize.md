### What Is 
[Sequelize](https://sequelize.org/docs/v6/) is **a promise-based, Node.js ORM (Object-relational mapping) for Postgres, MySQL, MariaDB, SQLite and Microsoft SQL Server**. It features solid transaction support, relations, eager and lazy loading, read replication, and more.

**Standart port -> 5432**

### Project init
1 -> Init in Console
```
npm i sequelize pg pg-hstore --save
npm i sequelize-cli --save

```

2 -> Create .sequelizerc and fill it
```
const path = require('path');  
  
module.exports = {  
'config': path.resolve('db', 'database.json'),  
'models-path': path.resolve('db', 'models'),  
'seeders-path': path.resolve('db', 'seeders'),  
'migrations-path': path.resolve('db', 'migrations')  
};
```

3-> Change database.json to 
- your db userName 
- your db userPassword 
- your db Type (Postrgres)

4-> Init sequelize-cli
```
npx sequelize-cli init

```


5-> Change packages.
```
"initdb": "npx sequelize db:create\nnpx sequelize-cli db:migrate:undo:all\nnpx sequelize-cli db:migrate\nnpx sequelize-cli db:seed:all\n",
```

### Start Project
#### Migrations
-> Creating Migration
```
npx sequelize-cli model:generate --name ModelName --attributes modelAttr_1:string,modelAttr_2:integer

```

-> Creating Seed
```
npx sequelize-cli seed:generate --name SeedName

```
-> change seed file
```
module.exports = {  
	up: (queryInterface, Sequelize) => {  
		return queryInterface.bulkInsert('Users', [{  
			firstName: 'John',  
			lastName: 'Doe',  
			email: 'example@example.com',  
			createdAt: new Date(),  
			updatedAt: new Date()  
		}]);  
	},  
	down: (queryInterface, Sequelize) => {  
		return queryInterface.bulkDelete('Users', null, {});  
	}  
};
```

#### Associations
[Associations Docs](https://sequelize.org/docs/v6/core-concepts/assocs/)

-> The `HasOne` association<br>
-> The `BelongsTo` association<br>
-> The `HasMany` association<br>
-> The `BelongsToMany` association<br>

The foreign key being defined in **the target model** (`B`)

```
A.hasOne(B, { foreignKey: 'B.key' });
	// A HasOne B
A.belongsTo(B, { foreignKey: 'A.key' });
	// A BelongsTo B
A.hasMany(B, { foreignKey: 'B.key' });
	// A HasMany B
A.belongsToMany(B, { through: 'C', foreignKey: 'C.key' });
	// A BelongsToMany B through the junction table C
```

#### Finders
Docs -> https://sequelize.org/docs/v6/core-concepts/model-querying-finders/

To disable this wrapping and receive a plain response instead, pass `{ raw: true }` as an option to the finder method.

-> `findAll()`
-> `findOne()`
```
const project = await Project.findOne({ where: { title: 'My Title' } });
if (project === null) {
  console.log('Not found!');
} else {
  console.log(project instanceof Project); // true
  console.log(project.title); // 'My Title'
}
```
-> `findByPk()`
```
const project = await Project.findByPk(123);
if (project === null) {
  console.log('Not found!');
} else {
  console.log(project instanceof Project); // true
  // Its primary key is 123
}
```
-> `findOrCreate()`
```
const [user, created] = await User.findOrCreate({
  where: { username: 'sdepold' },
  defaults: {
    job: 'Technical Lead JavaScript'
  }
});
console.log(user.username); // 'sdepold'
console.log(user.job); // This may or may not be 'Technical Lead JavaScript'
console.log(created); // The boolean indicating whether this instance was just created
if (created) {
  console.log(user.job); // This will certainly be 'Technical Lead JavaScript'
}
```
-> `findAndCountAll()`
```
const { count, rows } = await Project.findAndCountAll({
  where: {
    title: {
      [Op.like]: 'foo%'
    }
  },
  offset: 10,
  limit: 2
});
console.log(count);
console.log(rows);
```