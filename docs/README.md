# iXperience Full Stack 2019 - Day 6

[[toc]]

## LoopBack

### Installation

Enter in command terminal (make sure you have the lastest version of node or >v8)
```bash
npm i -g @loopback/cli
```

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

### Generate LoopBack Model

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
### Anatomy of user model

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
### Generate Repository

Generate Datasource
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

### Generate Controller

Generate controller
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

### Anatomy of Controller

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

  