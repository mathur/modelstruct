modelstruct
=========
A simple Python module to expose SQLAlchemy model structures as a RESTful API.

This is useful for Flask microservices, or other Python applications where the transfer of data stems all the way down to sharing classes (models) as well. If set up correctly, only one copy of these models is required on the main service, and the other service requiring use of the model can automatically generate and update it's side accordingly.

## Example
Python model:

```python
class Book(db.Model):
    __tablename__ = 'books'

    id = db.Column(db.Integer, primary_key=True)
    time_created = db.Column(db.DateTime)
    name = db.Column(db.String(140), unique=True)
    description = db.Column(db.String(320))
    in_inventory = db.Column(db.Boolean, default=True)
    user_id = db.Column(db.Integer)
    author = db.Column(db.String(140))

    def __init__(self, name, author, description, in_inventory, user_id):
        self.name = name
        self.description = description
        self.in_inventory = in_inventory
        self.user_id = user_id
        self.time_created = datetime.datetime.now()
        self.author = author

    def __repr__(self):
        return '<Book %r>' % self.id
```

JSON exposed via a RESTful API for this model:

```json
{
    "structure": {
        "author": "VARCHAR(140)",
        "description": "VARCHAR(320)",
        "id": "INTEGER",
        "in_inventory": "BOOLEAN",
        "name": "VARCHAR(140)",
        "time_created": "DATETIME",
        "user_id": "INTEGER"
    }
}
```

On the other side of things, you can programatically create Python classes with the help of a ClassFactory:

```python
class BaseClass(object):
    def __init__(self, classtype):
        self._type = classtype

def ClassFactory(name, argnames, BaseClass=BaseClass):
    def __init__(self, **kwargs):
        for key, value in kwargs.items():
            if key not in argnames:
                raise TypeError("Argument %s not valid for %s"
                    % (key, self.__class__.__name__))
            setattr(self, key, value)
        BaseClass.__init__(self, name[:-len("Class")])
    newclass = type(name, (BaseClass,),{"__init__": __init__})
    return newclass
```

You can then use the created classes/factory like so:

```python
list_of_fields = (get this list from parsing the JSON)

# first create the fields
SpecialClass = ClassFactory("SpecialClass", list_of_fields)

# then, set the fields with values
s = SpecialClass(a=2)
```

Then you can go ahead and use that new `SpecialClass` with whatever you may need it for.

## Install

```
pip install modelstruct
```

## Usage

* Import the Python module
```python
from modelstruct import get_json
```

* Create a new route for the endpoint you want to be your API (below the example is in Flask, and creates the API at the location `/models`, with the model class name `MODEL_NAME_HERE`)

```python
@app.route('/models', methods=['GET'])
def get_model_structure():
    return get_json(MODEL_NAME_HERE)
```

You can protect this endpoint using whatever type of authentication you'd like (a secret delivered via a parameter assuming you are on HTTPS, a header in the request body, etc).

* That's it! Send a GET request to the endpoint and get the model's structure.

## Contributions
Thanks to [this StackOverflow answer](https://stackoverflow.com/questions/15247075/how-can-i-dynamically-create-derived-classes-from-a-base-class) for an example of how to programatically create Python classes.