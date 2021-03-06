# Model API

## Built in functionality

### Instance-level methods

These methods are called on concrete instances of a model:

- `inst.values()`: Returns the model data as a plain JS object. This
preserves any ImmutableJS values you may have set inside a model.
- `inst.toJS()`: Deeply coverts the model data to plain JS objects. This
does not preserve any ImmutableJS values you may have set.
- `inst.unsetId()`: Unsets the `idField` of the model instance

### Class-level methods

These methods are called on the classes themselves, not on instances:

- `Model.blank()`: returns a new instance of the model with default values
specified in `fields`
- `Model.fieldNames()`: returns all field names as an array
- `Model.submodelFieldDNames()`: A class-level method which returns all field
names of nested classes
- `Model.item()`: returns a new `Provider` asserting
that the API response returns a single model
- `Model.list()`: Returns a new `Provider` asserting that the API response
returns a list of models
- `Model.getItem(params<Object>)`: returns a new `Query` requsting a single
item to be loaded
- `Model.getList(params<Object>)`: returns a new `Query` requsting a list of
items to be loaded


## Defining props

Models are a core part of tectonic, and define all of the attributes for your
resources.

In order to define a model you must extend Tectonic's base class. There are
three main class variables you need to define:

```js
import { Model } from 'tectonic';

class UserModel extends Model {
  // 'modelName' is a required field and must be unique among all models
  static modelName = 'user';

  // 'idField' is the attribute which holds the model ID; this defaults to 'id'
  // if left undefined.  This must be set if the ID is in another field.
  static idField = 'id';

  // 'fields' define the attributes that this model can hold, along with
  // any default values for a new, empty model.  These default vaues are
  // also passed to a component when data is pending.
  static fields = {
    id: undefined,
    name: 'Anonymoose',
    email: '',
  }
}
```

Models can be nested:

```js
class PostModel extends Model {
  static modelName = 'post';

  static fields = {
    id: undefined,
    slug: '',
    body: '',
    // ensure the 'author' object is converted into a UserModel
    author: new UserModel(),
  }
}
```


### Data manipulation

Often you'll need to manipulate a model's data to show correctly within a
component.  You can define custom methods on your class to manipulate data
so that you keep your components decoupled from data logic:

```js
class User extends Model {
  static modelName = 'user';

  static fields = {
    id: undefined,
    email: '',
    avatar: '',
  }

  getAvatar() {
    if (this.avatar === '') {
      return 'https://placehold.it/500x500';
    }
    return this.avatar
  }
}
```

Now we can use `this.props.user.getAvatar()` to show a placeholder or their
avatar.


### Filtering data

Models can also define a filter function.  All data passed to a model during
construction will be filtered via this function before being set:

```js
class User extends Model {
  // this needs to be a static function so that it's set on the class
  // as a prototypical method, not the instance.
  static filter(data) {
    // The data returned from this function will be used to set the model's
    // fields.
    return {
      ...data,
      fullName: `${data.firstName} ${data.lastName}`,
    };
  }
}
```

Note that often using instance methods as explained above in the `data
manipulation` section is often a better idea.

### Gotchas

Models can't have a `size` attribute due to conflicting field names with
ImmutableJS.  If you have a `size` attribute from your API you should rename
this within your frontend model and use filtering to rename the field before
data is set.

We know this kinda sucks. Sorry.
