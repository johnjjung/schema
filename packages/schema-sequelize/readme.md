# @feathersjs/schema-sequelize

Sequelize converter for `@feathersjs/schema` schemas:

```js
import Joi from '@hapi/joi';
import { schema, property } from '@feathersjs/schema';

@schema({
  name: 'users'
})
export class User {
  @property<Joi.NumberSchema> (validator => validator.integer(), {
    sequelize: {
      primaryKey: true,
      autoIncrement: true
    }
  })
  id: number;

  @property<Joi.StringSchema> (validator => validator.email().required())
  email: string;

  @property<Joi.NumberSchema> (validator => validator.integer())
  age: number;
}

@schema({
  name: 'todos',
  sequelize (TodoModel: any, models: any) {
    TodoModel.belongsTo(models.users, { as: 'user' });
  }
})
export class Todo {
  @property<Joi.NumberSchema> (validator => validator.integer(), {
    sequelize: {
      primaryKey: true,
      autoIncrement: true
    }
  })
  id: number;
  
  @property(validator => validator.required())
  text: string;

  @property()
  userId: number;

  @property({
    async resolve (todo: any, context: any) {
      return context.users.findOne({
        raw: true,
        where: {
          id: todo.userId
        }
      });
    }
  })
  user: User;
}

const client = new Sequelize('sqlite://test-db.sqlite');

const UserModel = convert (User, client);
const TodoModel = convert(Todo, client);

associate(client);

await client.sync();

assert.ok(UserModel);
assert.ok(TodoModel);
```
