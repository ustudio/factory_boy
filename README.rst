factory_boy
===========

factory_boy is a fixtures replacement based on thoughtbot's `factory_girl <http://github.com/thoughtbot/factory_girl>`_ . Like factory_girl it has a straightforward definition syntax, support for multiple build strategies (saved instances, unsaved instances, attribute dicts, and stubbed objects), and support for multiple factories for the same class, including factory inheritance. Django support is included, and support for other ORMs can be easily added.

Credits
-------

This README parallels the factory_girl README as much as possible; text and examples are reproduced for comparison purposes. Ruby users of factory_girl should feel right at home with factory_boy in Python.

factory_boy was written by Mark Sandstrom.

Thank you Joe Ferris and thoughtbot for creating factory_girl.

Download
--------

Github: http://github.com/dnerdy/factory_boy/tree/master

easy_install::
    
    easy_install factory_boy
    
Source::
    
    # Download the source and run
    python setup.py install
    

Defining factories
------------------

Factories declare a set of attributes used to instantiate an object. The name of the factory is used to guess the class of the object by default, but it's possible to explicitly specify it::

    import factory
    from models import User
    
    # This will guess the User class
    class UserFactory(factory.Factory):
        first_name = 'John'
        last_name = 'Doe'
        admin = False
        
    # This will use the User class (Admin would have been guessed)
    class AdminFactory(factory.Factory):
        FACTORY_FOR = User
        
        first_name = 'Admin'
        last_name = 'User'
        admin = True

Using factories
---------------

factory_boy supports several different build strategies: build, create, attributes and stub::

    # Returns a User instance that's not saved
    user = UserFactory.build()
    
    # Returns a saved User instance
    user = UserFactory.create()
    
    # Returns a dict of attributes that can be used to build a User instance
    attributes = UserFactory.attributes()
    
    # Returns an object with all defined attributes stubbed out:
    stub = UserFactory.stub()
    
You can use the Factory class as a shortcut for the default build strategy::

    # Same as UserFactory.create()
    user = UserFactory()
    
The default strategy can be overridden::

    UserFactory.default_strategy = factory.BUILD_STRATEGY
    user = UserFactory()
    
The default strategy can also be overridden for all factories::

    # This will set the default strategy for all factories that don't define a default build strategy
    factory.Factory.default_strategy = factory.BUILD_STRATEGY
    
No matter which strategy is used, it's possible to override the defined attributes by passing keyword arguments::

    # Build a User instance and override first_name
    user = UserFactory.build(first_name='Joe')
    user.first_name
    # => 'Joe'
    
Lazy Attributes
---------------

Most factory attributes can be added using static values that are evaluated when the factory is defined, but some attributes (such as associations and other attributes that must be dynamically generated) will need values assigned each time an instance is generated. These "lazy" attributes can be added as follows::

    class UserFactory(factory.Factory):
        first_name = 'Joe'
        last_name = 'Blow'
        email = factory.LazyAttribute(lambda a: '{0}.{1}@example.com'.format(a.first_name, a.last_name).lower())
        
    UserFactory().email
    # => 'joe.blow@example.com'
    
The function passed to ``LazyAttribute`` is given the attributes defined for the factory up to the point of the LazyAttribute declaration. If a lambda won't cut it, the ``lazy_attribute`` decorator can be used to wrap a function::

    # Stub factories don't have an associated class.
    class SumFactory(factory.StubFactory):
        lhs = 1
        rhs = 1
        
        @lazy_attribute
        def sum(a):
            result = a.lhs + a.rhs  # Or some other fancy calculation
            return result
    
Associations
------------

Associated instances can also be generated using ``LazyAttribute``::

    from models import Post
    
    class PostFactory(factory.Factory):
        author = factory.LazyAttribute(lambda a: UserFactory())
        
The associated object's default strategy is always used::

    # Builds and saves a User and a Post
    post = PostFactory()
    post.id == None           # => False
    post.author.id == None    # => False

    # Builds and saves a User, and then builds but does not save a Post
    post = PostFactory.build()
    post.id == None           # => True
    post.author.id == None    # => False
  
Inheritance
-----------

You can easily create multiple factories for the same class without repeating common attributes by using inheritance::

    class PostFactory(factory.Factory):
        title = 'A title'
        
    class ApprovedPost(PostFactory):
        approved = True
        approver = factory.LazyAttribute(lambda a: UserFactory())
        
Sequences
---------

Unique values in a specific format (for example, e-mail addresses) can be generated using sequences. Sequences are defined by using ``Sequence`` or the decorator ``sequence``::

    class UserFactory(factory.Factory):
        email = factory.Sequence(lambda n: 'person{0}@example.com'.format(n))
        
    UserFactory().email  # => 'person0@example.com'
    UserFactory().email  # => 'person1@example.com'
        
Sequences can be combined with lazy attributes::

    class UserFactory(factory.Factory):
        name = 'Mark'
        email = factory.LazyAttributeSequence(lambda a, n: '{0}+{1}@example.com'.format(a.name, n).lower())
        
    UserFactory().email  # => mark+0@example.com
