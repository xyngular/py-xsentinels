# Overview

Various objects that allow for sentinel-like xsentinels for various purposes, including:

- Easily create your own custom singletons/sentinels.
- Use pre-created ones, such as:
  - Default
  - Null


# Install

```bash
# via pip
pip install xsentinels

# via poetry
poetry add xsentinels
```

# Singleton Object

Allows for creation of formal custom special sentinel types,
similar to how `None` and `NoneType` are like.
Ensures there can only ever be one instance, fi you try to allocate a new one or even try to 
copy it, will still be the same single instance, no matter what.

Used for sentinel values `Null`, and `Default`, which are detailed further below in this readme..

The Singleton class can also easily be used as needed to create true singleton for other
purposes than just being sentinels.

## Singleton Example

Here is an example of using Singleton to easily make a safe sentinel object:

```python
from xsentinels import Singleton
import os

class DefaultType(Singleton):
    pass

# You can import this into other places and know there can only ever by one
# instance of DefaultType (and so can safely use the `is` operator with it):
Default = DefaultType()

# Ensures there can only ever by one 'instance' of the Singleton subclass:
assert Default is DefaultType()

...

# Example use Default Sentinel:

def my_method(some_param = Default):
    # Can tell the difference between `None` and `Default` via sentinel object,
    # So we can resolve the `Default` value however we wish that makes sense:
    if some_param is Default:
        some_param = os.environ.get("SOME_PARAM", None)
    # Do something with `some_param`, now that any `Default` value has been resolved.
    ...
```

# Null Object

A sentinel object modeled after `None` called `Null`. 

`Null` evaluates as a `False`-like object. Its purpose is for when you need to know
the difference between Null and None, like for a dataclass that represents a JSON document.

If for example one needs to know if the value was absent from the JSON document vs the value being set to Null.
For a normal object, its simpler to have the attribute set to a Null or None value
then it is to remove the attribute entirely from the object that represents this theoretical
JSON document.

## Null Example

Here is an example of using `Null`:

```python
from xsentinels import Null

class SomeClass:
    nullable_str = 'default-str'
    some_int = 3
    
    def json_dict(self):
        json_dict = {}
        if self.nullable_str is not None:
            json_dict['nullable_str'] = self.nullable_str
        if self.some_int is not None:
            json_dict['some_int'] = self.some_int
        
        return {k: v if v is not Null else None for k, v in json_dict.items()}

obj = SomeClass()
obj.nullable_str = Null
obj.some_int = None

# When getting the json_dict value of object, the original None value in some_int is absent,
# and None replaced the Null; which is what was intended.
# Now when json_dict is converted to json-string, python will convert the `None` to json `null` value.
assert obj.json_dict() == {'nullable_str': None}
```

In the above example, we have a class that can have None or Null values, which control how
the object is in-turn represented in a json-dict value of its self.


# Default Object

A sentinel object modeled after `None` called `Default`. 

`Default` evaluates as a `False`-like object. Its purpose is for when you need to know
if the caller set None on a method param or attribute vs not setting anything at all.

In other-words, if you need to know the difference between the user supplying a value of `None`
or not supplying a value in the first place.

When the user does not supply any value for an attribute or method parameter,
the underlying code that uses that value may want to supply a sensible default value
to use instead.

The `Default` sentential value can help you do this easily.

## Default Example

Here is an example of using `Default`:

```python
from xsentinels import Default
import os

def my_method(some_param = Default):
    # Can tell the difference between `None` and `Default` via sentinel object,
    # So we can resolve the `Default` value however we wish that makes sense:
    if some_param is Default:
        some_param = os.environ.get("SOME_PARAM", None)
    # Do something with `some_param`, now that any `Default` value has been resolved.
    ...
```

The above example will attempt to lookup a default value via a environmental variable
if the user does not pass anything into the function (ie: value for `some_param` is still at `Default`).


# Utilites

## unwrap_union

You can pass a type/type-hint object to `unwrap_union` to get information about if it contains
a Null or Optional type, along with the non-null/default type.

When you do something like this:

```python
from xsentinels import Nullable, unwrap_union
from typing import get_type_hints

class MyClass:
    my_attr: Nullable[str]


my_attr_type = get_type_hints(MyClass)['my_attr']
results = unwrap_union(my_attr_type)

assert results.unwrapped_type is str
assert results.is_nullable is True

# If the type had `Optional[str]` or something like Union[None, NullType, str]; this would be True.
assert results.is_optional is False

```
