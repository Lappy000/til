# Metaclasses for Dynamic Class Creation

Using metaclasses to dynamically generate classes, validate attributes, and implement registries.

## Basic Metaclass

```python
class ValidatedMeta(type):
    def __new__(mcs, name, bases, namespace):
        # Add validation to all properties
        cls = super().__new__(mcs, name, bases, namespace)
        if name != 'BaseModel':
            required = namespace.get('_required', [])
            for field in required:
                if field not in namespace:
                    raise TypeError(f"{name} missing required field: {field}")
        return cls

class BaseModel(metaclass=ValidatedMeta):
    _required = []

class User(BaseModel):
    _required = ['name', 'email']
    name = None
    email = None
```

## Registry Pattern

```python
class RegistryMeta(type):
    _registry = {}

    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if bases:  # skip base class
            key = namespace.get('_type', name.lower())
            mcs._registry[key] = cls
        return cls

    @classmethod
    def get(mcs, key):
        return mcs._registry.get(key)

    @classmethod
    def all(mcs):
        return dict(mcs._registry)

class Handler(metaclass=RegistryMeta):
    def process(self, data):
        raise NotImplementedError

class JSONHandler(Handler):
    _type = 'json'
    def process(self, data):
        return f"Processing JSON: {data}"

class XMLHandler(Handler):
    _type = 'xml'
    def process(self, data):
        return f"Processing XML: {data}"

# Auto-registered
print(RegistryMeta._registry)
# {'json': <class JSONHandler>, 'xml': <class XMLHandler>}

handler = RegistryMeta.get('json')()
handler.process({'key': 'value'})
```

## Dynamic Class Generation

```python
def create_model(name, fields):
    attrs = {f: None for f in fields}
    attrs['__annotations__'] = {f: str for f in fields}
    attrs['__init__'] = lambda self, **kw: [setattr(self, k, v) for k, v in kw.items()]
    return type(name, (object,), attrs)

UserModel = create_model('UserModel', ['id', 'name', 'email'])
```
