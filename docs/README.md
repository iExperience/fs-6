# iXperience Full Stack 2019 - Day 6

[[toc]]


## Express

### Generate package.json

Create package.json
```bash
npm init
```

### Installation

install express
```bash
npm install express --save
```

### Main app entry point - index.js
index.js
```js
const express = require('express');

const app = express();
 
const PORT = process.env.PORT || 5000;

app.listen(PORT,()=>console.log(`Server running on port ${PORT}`));
```

### Run server locally

run server
```bash
node .
```

add run script to package.json 
```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start"  : "node ."
  }
```

run server
```bash
npm start
```

### Basic Route Handling - Hello World

index.js
```js
const express = require('express');

const app = express();

app.get('/', (req,res) => {
    res.send('<h1>Hello World!</h1>')
});
 
const PORT = process.env.PORT || 5000;

app.listen(PORT,()=>console.log(`Server running on port ${PORT}`));
```

### Nodemon

installing nodemon as dev dependacy
```bash
npm install -D nodemon
```

### Nodemon run scripts

adding run script to package.json
```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node .",
    "dev": "nodemon"
  }
```

Run server with nodemon
```bash
npm run dev
```

## Express Route Handling

### Create User routes in express

Add dumy data - index.js
```js
const users = [
    {
        "id": "1",
        "name": "Byron",
        "role": "user",
        "email": "byron@mail.com",
        "password": "P@ssword"
      },
      {
        "id": "2",
        "name": "brett",
        "role": "service provider",
        "email": "brett@mail.com",
        "password": "P@ssword1"
      },
      {
        "id": "3",
        "name": "Riaz",
        "role": "user",
        "email": "riaz@mail.com",
        "password": "P@ssword2"
      },
    ]
```

Get users route - index.js  
```js
app.get('/api/users', (req,res) => {
    res.json(users);
});
```

### Install FS and add data.json

Install fs
```bash
npm install fs
```

Add dumy data - ./data/data.json
```json
{
  "users":[
    {
      "id":"1",
      "name":"Byron",
      "surname":"de Villiers",
      "cellPhone":"0829123465",
      "email":"byron@mail.com",
      "password":"P@ssword",
      "role":"provider"
    }
  ]
}
```

### Update User Route - index.js

Update users route - index.js  
```js
var fs = require("fs");

router.get("/api/users", (req, res) => {
  //res.send('<h1>Hello World!</h1>');
  fs.readFile("./src/data/data.json", function(err, data) {
    if (err) throw err;
    var parseData = JSON.parse(data);
    res.json(parseData.users);
  });
});
```


### Middleware

index.js  
```js
//Middleware function:
const logger = (req,res,next) => {
    console.log(`${req.protocol}://${req.get('host')}${req.originalUrl}`);
    next();
}

//Middleware execue:
app.use(logger)
```

### Middleware

index.js  
```js
//Middleware function:
const logger = (req,res,next) => {
    console.log(`${req.protocol}://${req.get('host')}${req.originalUrl}`);
    next();
}

//Middleware execue:
app.use(logger)
```

### Get user by Id express endpoint

index.js  
```js
app.get("api/users/:id", (req, res) => {
  fs.readFile("./src/data/data.json", function(err, data) {
    if (err) throw err;
    var parseData = JSON.parse(data);
    res.json(users.filter(user => user.id === req.params.id));
  });
});
```

### Error Handling

index.js  
```js
app.get("api/users/:id", (req, res) => {
  fs.readFile("./src/data/data.json", function(err, data) {
    if (err) throw err;
    var parseData = JSON.parse(data);
    const found = parseData.users.some(user => user.id === req.params.id);
    if (found) {
      res.json(users.filter(user => user.id === req.params.id));
    } else {
      res.status(400).json({ msg: "User not found" });
    }
  });
});
```

### Express Router

./src/api/user-routes.js  
```js
const express = require("express");
const router = express.Router();
var fs = require("fs");

router.get("/", (req, res) => {
  //res.send('<h1>Hello World!</h1>');
  fs.readFile("./src/data/data.json", function(err, data) {
    if (err) throw err;
    var parseData = JSON.parse(data);
    res.json(parseData.users);
  });
});

router.get("/:id", (req, res) => {
  fs.readFile("./src/data/data.json", function(err, data) {
    if (err) throw err;
    var parseData = JSON.parse(data);
    const found = parseData.users.some(user => user.id === req.params.id);
    if (found) {
      res.json(parseData.users.filter(user => user.id === req.params.id));
    } else {
      res.status(400).json({ msg: "User not found" });
    }
  });
});

module.exports = router;
```

### Express Router - index.js

Updating express router - index.js  
```js
//Routes:
app.use("/api/users", require("./src/api/users-routes"));
```

## CRUD

### Create User

index.js
```js
//Body Parser Middlware:
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
```

user-routes.js
```js
router.post("/", (req, res) => {
  const user = req.body;
  fs.readFile("./src/data/data.json", function(err, data) {
    var error = false;
    var errMsg = "";
    if (err) {
      error = true;
      throw err;
    } else {
      var count = 0;

      if (data.length > 0) {
        var parseData = JSON.parse(data);
        parseData.users.forEach(existingUser => {
          if (existingUser.email === user.email) {
            throw new Error("This email address already been used");
          }
          count++;
        });
      } else {
        parseData = {
          users: []
        };
      }

      const newUser = {
        id: (count + 1).toString(),
        name: user.name,
        surname: user.surname,
        cellPhone: user.cellPhone,
        email: user.email,
        password: user.password,
        role: user.role
      };

      parseData.users.push(newUser);
      fs.writeFile("./src/data/data.json", JSON.stringify(parseData), function(
        err
      ) {
        if (err) {
          error = true;
          throw err;
        }
        res.json(newUser);
      });
    }

    if (error) {
      res.status(400).json({ msg: errMsg });
    } else {
      res.json(user);
    }
  });
});
```

### Update User

user-routes.js
```js
router.post("/update", (req, res) => {
  const user = req.body;
  fs.readFile("./src/data/data.json", function(err, data) {
    var error = false;
    if (err) {
      error = true;
      throw err;
    } else {
      if (data.length > 0) {
        var parseData = JSON.parse(data);
      } else {
        throw Error("No Users");
      }

      parseData.users = parseData.users.filter(existingUser => {
        return existingUser.email !== user.email;
      });

      const updateUser = {
        id: user.id,
        name: user.name,
        surname: user.surname,
        cellPhone: user.cellPhone,
        email: user.email,
        password: user.password,
        role: user.role
      };

      parseData.users.push(updateUser);
      fs.writeFile("./src/data/data.json", JSON.stringify(parseData), function(
        err
      ) {
        if (err) {
          error = true;
          throw err;
        }
        res.json(updateUser);
      });
    }

    if (error) {
      res.status(400).json({ msg: err.msg });
    } else {
      res.json(user);
    }
  });
});
```


### Delete User 

user-route.js
```js
router.get("/delete/:id", (req, res) => {
  fs.readFile("./src/data/data.json", function(err, data) {
    var error = false;
    if (err) {
      error = true;
      throw err;
    }
    var parseData = JSON.parse(data);
    parseData.users = parseData.users.filter(user => user.id === req.params.id);
    fs.writeFile("./src/data/data.json", JSON.stringify(parseData), function(
      err
    ) {
      if (err) {
        error = true;
        throw err;
      }
      res.json({ status: "User deleted" });
    });
    if (error) {
      res.status(400).json({ msg: err.message });
    } else {
      res.json({ status: "User deleted" });
    }
  });
});
```

## LoopBack

### Installation

Install LoopBack
```bash
npm i -g @loopback/cli
```
## LoopBack CLI

### Generate new LoopBack project

Generate new loopback project
```bash
lb4 app fs-bnb-api
```

### Run new LoopBack project

Open project in VS Code and run
```bash
cd fs-bnb-api
code .
npm start
```

## LoopBack Project Anatomy

### TODO

Make slides to go with a breif over view of files and what that do


## Generate LoopBack Model 

### Generate User Model

Generate user model
```bash
lb4 model
? Model class name: user
? Please select the model base class: Entity
? Allow additional (free-form) properties? No

Let's add a property to user
Enter an empty property name when done

? Enter the property name: id
? Property type: number
? Is id the ID property? Yes
? Is it required?: No
? Default value [leave blank for none]:

Let's add another property to user
Enter an empty property name when done

? Enter the property name: name
? Property type: string
? Is it required?: Yes
? Default value [leave blank for none]:

Let's add another property to user
Enter an empty property name when done

? Enter the property name: lastName
? Property type: string
? Is it required?: No
? Default value [leave blank for none]:

Let's add another property to user
Enter an empty property name when done

? Enter the property name: email
? Property type: string
? Is it required?: Yes
? Default value [leave blank for none]:

Let's add a property to user
Enter an empty property name when done

? Enter the property name: cellPhone
? Property type: number
? Is id the ID property? Yes
? Is it required?: No
? Default value [leave blank for none]:

Let's add another property to user
Enter an empty property name when done

? Enter the property name:

   create src/models/user.model.ts
   update src/models/index.ts

Model user was created in src/models/
```
### Anatomy of User Model

user.model.ts
```ts
import {Entity, model, property} from '@loopback/repository';

@model({settings: {}})
export class User extends Entity {
  @property({
    type: 'number',
    id: true,
  })
  id?: number;

  @property({
    type: 'string',
    required: true,
  })
  name: string;

  @property({
    type: 'string',
  })
  lastName?: string;

  @property({
    type: 'string',
    required: true,
  })
  email: string;

  @property({
    type: 'string',
  })
  cellPhone?: string;


  constructor(data?: Partial<User>) {
    super(data);
  }
}
```

notice it was added to src/models/index.ts
```ts
export * from './user.model';
```

## Add Datasource to LoopBack project

### Generate LoopBack Datasource files

Generate Datasource files
```bash
lb4 datasource
? Datasource name: db
? Select the connector for db: In-memory db (supported by StrongLoop)
? window.localStorage key to use for persistence (browser only):
? Full path to file for persistence (server only): ./data/db.json

  create src/datasources/db.datasource.json
  create src/datasources/db.datasource.ts
  update src/datasources/index.ts

Datasource db was created in src/datasources/
```

### Anatomy of dataSource

db.datasource.ts
```ts
import {inject} from '@loopback/core';
import {juggler} from '@loopback/repository';
import * as config from './db.datasource.json';

export class DbDataSource extends juggler.DataSource {
  static dataSourceName = 'db';

  constructor(
    @inject('datasources.config.db', {optional: true})
    dsConfig: object = config,
  ) {
    super(dsConfig);
  }
}
```

db.datasource.json
```json
{
  "name": "db",
  "connector": "memory",
  "localStorage": "",
  "file": "./data/db.json"
}
```
notice it was added to src/datasources/index.ts
```ts
export * from './db.datasource';
```

### Adding dumy data

./data/db.json
```json
{
  "ids": {
    "User": 6
  },
  "models": {
    "User": {
      "1": "{\"name\":\"Joe\",\"lastName\":\"Soap\",\"email\":\"joe@mail.com\",\"cellPhone\":\"0821234567\",\"id\":1}",
      "2": "{\"name\":\"John\",\"lastName\":\"Doe\",\"email\":\"john@mail.com\",\"cellPhone\":\"0831234567\",\"id\":2}",
      "3": "{\"name\":\"Jane\",\"lastName\":\"Smith\",\"email\":\"jane@mail.com\",\"cellPhone\":\"0741234567\",\"id\":3}",
      "4": "{\"name\":\"Sam\",\"lastName\":\"Hill\",\"email\":\"sam@mail.com\",\"cellPhone\":\"0829876543\",\"id\":4}",
      "5": "{\"name\":\"Byron\",\"lastName\":\"de Villiers\",\"email\":\"byron@mail.com\",\"cellPhone\":\"0741234567\",\"id\":5}"
    }
  }
}
```
### Generate User Repository

### Generate User Repository

generate user repository
```bash
lb4 repository
? Please select the datasource DbDatasource
? Select the model(s) you want to generate a repository User
? Please select the repository base class DefaultCrudRepository (Legacy juggler bridge)
   create src\repositories\user.repository.ts
   update src\repositories\index.ts

Repository UserRepository was created in src\repositories/
```

### Anatomy of Repository

user.repository.ts
```ts
import {DefaultCrudRepository} from '@loopback/repository';
import {User} from '../models';
import {DbDataSource} from '../datasources';
import {inject} from '@loopback/core';

export class UserRepository extends DefaultCrudRepository<
  User,
  typeof User.prototype.id
> {
  constructor(
    @inject('datasources.db') dataSource: DbDataSource,
  ) {
    super(User, dataSource);
  }
}
```

notice it was added to /src/repositories/index.ts
```ts
export * from './user.repository';
```

## Generate User Controller

### Generate CRUD User Controller

Generate user controller
```bash
lb4 controller
? Controller class name: user
? What kind of controller would you like to generate? REST Controller with CRUD functions
? What is the name of the model to use with this CRUD repository? User
? What is the name of your CRUD repository? UserRepository
? What is the type of your ID? number
? What is the base HTTP path name of the CRUD operations? /users
   create src\controllers\user.controller.ts
   update src\controllers\index.ts

Controller user was created in src\controllers/
```

### Anatomy of User Controller

user.controller.ts
```ts
import {
  Count,
  CountSchema,
  Filter,
  repository,
  Where,
} from '@loopback/repository';
import {
  post,
  param,
  get,
  getFilterSchemaFor,
  getWhereSchemaFor,
  patch,
  put,
  del,
  requestBody,
} from '@loopback/rest';
import {User} from '../models';
import {UserRepository} from '../repositories';

export class UserController {
  constructor(
    @repository(UserRepository)
    public userRepository : UserRepository,
  ) {}

  @post('/users', {
    responses: {
      '200': {
        description: 'User model instance',
        content: {'application/json': {schema: {'x-ts-type': User}}},
      },
    },
  })
  async create(@requestBody() user: User): Promise<User> {
    return await this.userRepository.create(user);
  }

  @get('/users/count', {
    responses: {
      '200': {
        description: 'User model count',
        content: {'application/json': {schema: CountSchema}},
      },
    },
  })
  async count(
    @param.query.object('where', getWhereSchemaFor(User)) where?: Where,
  ): Promise<Count> {
    return await this.userRepository.count(where);
  }

  @get('/users', {
    responses: {
      '200': {
        description: 'Array of User model instances',
        content: {
          'application/json': {
            schema: {type: 'array', items: {'x-ts-type': User}},
          },
        },
      },
    },
  })
  async find(
    @param.query.object('filter', getFilterSchemaFor(User)) filter?: Filter,
  ): Promise<User[]> {
    return await this.userRepository.find(filter);
  }

  @patch('/users', {
    responses: {
      '200': {
        description: 'User PATCH success count',
        content: {'application/json': {schema: CountSchema}},
      },
    },
  })
  async updateAll(
    @requestBody() user: User,
    @param.query.object('where', getWhereSchemaFor(User)) where?: Where,
  ): Promise<Count> {
    return await this.userRepository.updateAll(user, where);
  }

  @get('/users/{id}', {
    responses: {
      '200': {
        description: 'User model instance',
        content: {'application/json': {schema: {'x-ts-type': User}}},
      },
    },
  })
  async findById(@param.path.number('id') id: number): Promise<User> {
    return await this.userRepository.findById(id);
  }

  @patch('/users/{id}', {
    responses: {
      '204': {
        description: 'User PATCH success',
      },
    },
  })
  async updateById(
    @param.path.number('id') id: number,
    @requestBody() user: User,
  ): Promise<void> {
    await this.userRepository.updateById(id, user);
  }

  @put('/users/{id}', {
    responses: {
      '204': {
        description: 'User PUT success',
      },
    },
  })
  async replaceById(
    @param.path.number('id') id: number,
    @requestBody() user: User,
  ): Promise<void> {
    await this.userRepository.replaceById(id, user);
  }

  @del('/users/{id}', {
    responses: {
      '204': {
        description: 'User DELETE success',
      },
    },
  })
  async deleteById(@param.path.number('id') id: number): Promise<void> {
    await this.userRepository.deleteById(id);
  }
}

```

notice it was added to /src/controllers/index.ts
```ts
export * from './ping.controller';
export * from './user.controller';
```

  