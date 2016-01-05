---
layout: post
title: Class Creation in Python
tags:
- python
comments: true
---

* generate table of contents
{:toc}
-----
<br><br>

Everything in Python is an *object*. Every object has

- an identity, which never changes once the object has been created. It can be thought as the object's address in memory.
- a type, which is also unchangeable. The `type()` function returns an object's type.
- a value

## 1. Class

Both class types (new-style classes) and class objects (old-style/classic) are typically created by class definitions. A class has a namespace implemented by a dictionary object. Class attribute references are translated to lookups in this dictionary, e.g., `c.x` is translated to `c.__dict__['x']`. When the attribute name is not found there, the attribute search continues in the base classes.

Class attribute assignments updates the class's dictionary, never the dictionary of a base class.

Special attributes:

  - `__name__` is the class name;
  - `__module__` is the module name in which the class was defined;
  - `__dict__` is the dictionary containing the class's namespace;
  - `__base__` is a tuple (possibly empty or a singleton) containing the base classes, in the order of their occurrence in the base class list;
  - `__doc__` is the class's documentation string, or None if undefined.

## 2. Class instances
A class instance is created by calling a class object. A class instance has a namespace implemented as a dictionary which is the first place in which attribute references are searched. When an attribute is not found there, and the instance’s class has an attribute by that name, the search continues with the class attributes. If a class attribute is found that is a user-defined function object or an unbound user-defined method object whose associated class is the class (call it *C*) of the instance for which the attribute reference was initiated or one of its bases, it is transformed into a bound user-defined method object whose `__im_class__` attribute is *C* and whose `__im_self__` attribute is the instance. Static method and class method objects are also transformed, as if they had been retrieved from class *C*. If no class attribute is found, and the object’s class has a `__getattr__()` method, that is called to satisfy the lookup.

Attribute assignments and deletions update the instance’s dictionary, never a class’s dictionary. If the class has a `__setattr__()` or `__delattr__()` method, this is called instead of updating the instance dictionary directly.

Special attributes:

  - `__dict__` is the attribute dictionary;
  - `__class__` is the instance’s class.

## 3.The BIF class type(name, bases, dict)
<a id="biftype"></a> 
With one argument, return the type of an *object*. The return value is a *type* object. The `isinstance()` bif is recommended for testing the type of an object.

With three arguments, return a new *type* object. This is essentially a dynamic form of the `class` statement. The *name* string is the class name and becomes the `__name__` attribute; the *bases* tuple itemizes the base classes and becomes the `__bases__` attribute; and the *dict* dictionary is the namespace containing definitions for class body and becomes the `__dict__` attribute. For example, the following two statements create identical `type` objects:

{% highlight python linenos %}
>>> class X(object):
...     a = 1
...
>>> X = type('X',(object,),dict(a = 1))
{% endhighlight %}

## 4. Special method names

#### 1. object.\_\_new\_\_(*cls*[, ...])

Called to create a new instance of class *cls*. `__new__()` is a static method (special-cased so you need not declare it as such) that takes the class of which an instance was requested as its first argument. The remaining arguments are those passed to the object constructor expression (the call to the class).

The return value of `__new__()` should be the new object instance (usually an instance of *cls*)

Typically implementations create a new instance of the class by invoking the superclass's `__new__()` method using `super(currentclass, cls).__new__(cls[, ...])` with appropriate arguments and then modifying the newly-created instance as necessary before returning it.

If `__new__()` returns an instance of *cls*, then the new instance's `__init__()` method will be invoked like `__init__(self[, ...])`, where *self* is the new instance and the remaining arguments are the same as were passed to `__new__()`.

If `__new__()` does not return an instance of *cls*, then the new instance's `__init__()` method will not be invoked.

`__new__()` is intended mainly to allow subclasses of immutable types (like int, str, or tuple) to customize instance creation. It is also commonly overridden in custom metaclasses in order to customize class creation.

#### 2. object.\_\_init\_\_(*self*[, ...])

Called after the instance has been created (by `__new__()`), but before it is returned to the called. The arguments are those passed to the class constructor expression.

> ***Tips :*** If a base class has an `__init__()` method, the derived class's `__init__()` method, if any, must explicitly call it to ensure proper initialization of the base class part of the instance; for example: `BaseClass.__init__(self, [args ...])`.

Because `__new__()` and `__init__()` are working together in constructing objects (`__new__()` to create it, and `__init__()` to customise it), no non-None value may be returned by `__init__()`; doing so will cause a `TypeError` to be raised at runtime.

## 5. Customizeing class creation (Python 2.7)

By default, new-style classes are constructed using `type()`. A class definition is read into a separate namespace and the value of class name is bound to the result of `type(name, bases, dict)`.

When the class definition is read, if `__metaclass__` is defined then the callable assigned to it will be called instead of `type()`. This allows classes or functions to be written which monitor or alter the class creation process:

  - Modifying the class dictionary prior to the class being created.
  - Returning an instance of another class - essentially performing the role of a factory function.

These steps will have to be performed in the metaclass's `__new__()` method - `type.__new__()` can then be called from this method to create a class with different properties. This example adds a new element to the class dictionary before creating the class:

{% highlight python linenos %}
class metacls(type):
    def __new__(mcs, name, bases, dict):
        dict['foo'] = 'metacls was here'
        return type.__new__(mcs, name, bases, dict)
{% endhighlight %}

We can of course also override other class methods (or add new methods); for example defining a custom `__call__()` method in the metaclass allows custom behavior when the class is called, e.g. not always creating a instance.

### \_\_metaclass\_\_

This variable can be any callable accepting arguments for `name`,`bases` and `dict`. Upon class creation, callable is used instead of the built-in `type()`. [new in version 2.2].

The appropriate meaclass is determined by the following precedence rules:

  - If `dict['__metaclass__']` exists, it is used.
  - Otherwise, if there is at least one base class, its metaclass is used (this looks for a `__class__` attribute first and if not found, uses its type).
  - Otherwise, if a global variable named `__metaclass__` exists, it is used.
  - Otherwise, the old-style, classic metaclass (types.ClassType) is used.

The potential uses for metaclasses are boundless. Some ideas that have been explored including logging, interface checking, automatic delegation, automatic property creation, proxies, frameworks, and automatic resource locking/synchronization.

## 6. Customizing class creation (Python 3.4)

By default, classes are constructed using `type()`. The class body is executed in a new namespace and the class name is bound locally to the result of `type(name, bases, namespace)`.

The class creation process can be customized by passing the `metaclass` keyword argument in the class definition line, or by inheriting from an existing class that included such an argument. In the following example, both `MyClass` and `MySubclass` are instances of `Meta`:

{% highlight python linenos %}
class Meta(type):
    pass

class MyClass(metaclass = Meta):
    pass

class MySubclass(MyClass):
    pass
{% endhighlight %}

Any other keyword arguments that are specified in the class definition are passed through to all metaclass operations described below.

When a class definition is executed, the following steps occur:

- the appropriate metaclass is determined
- the class namespace is prepared
- the class body is executed
- the class object is created

### 6.1 Determining the appropriate metaclass

The appropriate metaclass for a class definition is determined as follows:

- if no bases and no explicit metaclass are given, the `type()` is used
- if an explicit metaclass is given and it is not an instance of `type()`, then it is used directly as the metaclass
- if an instance of `type()` is given as the explicit metaclass, or bases are defined, then the most derived metaclass is used

The most derived metaclass is selected from the explicitly specified metaclass (if any) and the metaclasses(i.e. `type(cls)`) of all specified base classes. The most derived metaclass is one which is a subtype of all of these candidate metaclasses. If none of the candidate metaclasses meets that criterion, then the class definition will fail with `TypeError`.

### 6.2 Preparing the class namespace

Once the appropriate metaclass has been identified, then the class namespace is prepared. If the metaclass has a `__prepare__` attribute, it is called as `namespace = metaclass.__prepare__(name, bases, **kwds)` (where the additional keyword arguments, if any, come from the class definition).

If the metaclass has no `__prepare__` attribute, then the class namespace is initialized as an empty `dict()` instance.

### 6.2 Executing the class body

The class body is executed (approximately) as `exec(body, globals(), namespace)`. The key difference from a normal class to `exec()` is that lexical scoping allows the class body (including any methods) to reference names from the current and outer scopes when the class definition occurs inside a function.

However, even when the class definition occurs inside the function, methods defined inside the class still cannot see names defined at the class scope. Class variables must be accessed through the first parameter of instance or methods, and cannot be accessed at all from static methods.

### 6.3 Creating the class object

Once the class namespace has been populated by executing the class body, the class object is created by calling `metaclass(name, bases, namespace, **kwds)` (the additional keywords passed here are the same as those passed to `__prepare__`).

This class object is the one that will be reference by the zero-argument form of `super()`. `__class__` is an implicit closure reference created by the compiler if any methods in a class body refer to either `__class__` or `super`. This allows the zero argument form of `super()` to correctly identify the class being defined based on lexical scoping, while the class or instance that was used to make the current call is identified based on the first argument passed to the method.

After the class object is created, it is passed to the class decorators included in the class definition (if any) and the resulting object is bound in the local namespace as the defined class.

###6.4 Metaclass example

Here is an example of a metaclass that uses an `collections.OrderedDict` to remember the order hat class variables are defined:

{% highlight python linenos %}
class OrderedClass(type):
    @classmethod
    def __prepare__(metacls, name, bases, **kwds):
        return collections.OrderedDict()

    def __new__(cls, name, bases, namespace, **kwds):
        result = type.__new__(cls, name, bases, dict(namespace))
        result.members = tuple(namespace)
        return result

class A(metaclass=OrderedClass):
    def one(self): pass
    def two(self): pass
    def three(self): pass
    def four(self): pass

>>> A.members
('__module__', 'one', 'two', 'three', 'four')
{% endhighlight %}

When the class definition for *A* gets executed, the process begins with calling the metaclass's `__prepare__` method which returns an empty `collections.OrderedDict`. That mapping records the methods and attributes of *A* as they are defined within the body of the class statement. Once those definitions are executed, the ordered dictionary is fully populated and the metaclass's `__new__()` method gets invoked. That method builds the new type and it saves the ordered dictionary keys in an attribute called `members`.


## Examples
[singleton](../../../2015/12/13/singleton.html)
