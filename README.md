modelstruct
=========
A simple Python module to expose SQLAlchemy model structures as a RESTful API.

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

## Install

```
pip install modelstruct
```

## Usage

1. Import the Python module
```python
from modelstruct import get_json
```

2. Create a new route for the endpoint you want to be your API (below the example is in Flask, and creates the API at the location `/models`, with the model class name `MODEL_NAME_HERE`)

```python
@app.route('/models', methods=['GET'])
def get_model_structure():
    return get_json(MODEL_NAME_HERE)
```

3. That's it! Send a GET request to the endpoint and get the model's structure.