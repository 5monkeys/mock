mock is a library for testing in Python. It allows you to replace parts of
your system under test with mock objects and make assertions about how they
have been used.

mock provides a core `MagicMock` class removing the need to create a host of
stubs throughout your test suite. After performing an action, you can make
assertions about which methods / attributes were used and arguments they were
called with. You can also specify return values and set needed attributes in
the normal way.

mock is tested on Python versions 2.4-2.7 and Python 3. mock is also tested
with the latest versions of Jython and pypy.

The mock module also provides utility functions / objects to assist with
testing, particularly monkey patching.

* `PDF documentation for 1.0 beta 1
  <http://www.voidspace.org.uk/downloads/mock-1.0b1.pdf>`_
* `mock on google code (repository and issue tracker)
  <http://code.google.com/p/mock/>`_
* `mock documentation
  <http://www.voidspace.org.uk/python/mock/>`_
* `mock on PyPI <http://pypi.python.org/pypi/mock/>`_
* `Mailing list (testing-in-python@lists.idyll.org)
  <http://lists.idyll.org/listinfo/testing-in-python>`_

Mock is very easy to use and is designed for use with
`unittest <http://pypi.python.org/pypi/unittest2>`_. Mock is based on
the 'action -> assertion' pattern instead of 'record -> replay' used by many
mocking frameworks. See the `mock documentation`_ for full details.

Mock objects create all attributes and methods as you access them and store
details of how they have been used. You can configure them, to specify return
values or limit what attributes are available, and then make assertions about
how they have been used::

    >>> from mock import Mock
    >>> real = ProductionClass()
    >>> real.method = Mock(return_value=3)
    >>> real.method(3, 4, 5, key='value')
    3
    >>> real.method.assert_called_with(3, 4, 5, key='value')

`side_effect` allows you to perform side effects, return different values or
raise an exception when a mock is called::

   >>> mock = Mock(side_effect=KeyError('foo'))
   >>> mock()
   Traceback (most recent call last):
    ...
   KeyError: 'foo'
   >>> values = {'a': 1, 'b': 2, 'c': 3}
   >>> def side_effect(arg):
   ...     return values[arg]
   ...
   >>> mock.side_effect = side_effect
   >>> mock('a'), mock('b'), mock('c')
   (3, 2, 1)
   >>> mock.side_effect = [5, 4, 3, 2, 1]
   >>> mock(), mock(), mock()
   (5, 4, 3)

Mock has many other ways you can configure it and control its behaviour. For
example the `spec` argument configures the mock to take its specification from
another object. Attempting to access attributes or methods on the mock that
don't exist on the spec will fail with an `AttributeError`.

The `patch` decorator / context manager makes it easy to mock classes or
objects in a module under test. The object you specify will be replaced with a
mock (or other object) during the test and restored when the test ends::

    >>> from mock import patch
    >>> @patch('test_module.ClassName1')
    ... @patch('test_module.ClassName2')
    ... def test(MockClass2, MockClass1):
    ...     test_module.ClassName1()
    ...     test_module.ClassName2()

    ...     assert MockClass1.called
    ...     assert MockClass2.called
    ...
    >>> test()

.. note::

   When you nest patch decorators the mocks are passed in to the decorated
   function in the same order they applied (the normal *python* order that
   decorators are applied). This means from the bottom up, so in the example
   above the mock for `test_module.ClassName2` is passed in first.

   With `patch` it matters that you patch objects in the namespace where they
   are looked up. This is normally straightforward, but for a quick guide
   read `where to patch
   <http://www.voidspace.org.uk/python/mock/patch.html#where-to-patch>`_.

As well as a decorator `patch` can be used as a context manager in a with
statement::

    >>> with patch.object(ProductionClass, 'method') as mock_method:
    ...     mock_method.return_value = None
    ...     real = ProductionClass()
    ...     real.method(1, 2, 3)
    ...
    >>> mock_method.assert_called_once_with(1, 2, 3)

There is also `patch.dict` for setting values in a dictionary just during the
scope of a test and restoring the dictionary to its original state when the
test ends::

   >>> foo = {'key': 'value'}
   >>> original = foo.copy()
   >>> with patch.dict(foo, {'newkey': 'newvalue'}, clear=True):
   ...     assert foo == {'newkey': 'newvalue'}
   ...
   >>> assert foo == original

Mock supports the mocking of Python magic methods. The easiest way of
using magic methods is with the `MagicMock` class. It allows you to do
things like::

    >>> from mock import MagicMock
    >>> mock = MagicMock()
    >>> mock.__str__.return_value = 'foobarbaz'
    >>> str(mock)
    'foobarbaz'
    >>> mock.__str__.assert_called_once_with()

Mock allows you to assign functions (or other Mock instances) to magic methods
and they will be called appropriately. The MagicMock class is just a Mock
variant that has all of the magic methods pre-created for you (well - all the
useful ones anyway).

The following is an example of using magic methods with the ordinary Mock
class::

    >>> from mock import Mock
    >>> mock = Mock()
    >>> mock.__str__ = Mock(return_value = 'wheeeeee')
    >>> str(mock)
    'wheeeeee'

For ensuring that the mock objects your tests use have the same api as the
objects they are replacing, you can use "auto-speccing". Auto-speccing can
be done through the `autospec` argument to patch, or the `create_autospec`
function. Auto-speccing creates mock objects that have the same attributes
and methods as the objects they are replacing, and any functions and methods
(including constructors) have the same call signature as the real object.

This ensures that your mocks will fail in the same way as your production
code if they are used incorrectly::

   >>> from mock import create_autospec
   >>> def function(a, b, c):
   ...     pass
   ...
   >>> mock_function = create_autospec(function, return_value='fishy')
   >>> mock_function(1, 2, 3)
   'fishy'
   >>> mock_function.assert_called_once_with(1, 2, 3)
   >>> mock_function('wrong arguments')
   Traceback (most recent call last):
    ...
   TypeError: <lambda>() takes exactly 3 arguments (1 given)

`create_autospec` can also be used on classes, where it copies the signature of
the `__init__` method, and on callable objects where it copies the signature of
the `__call__` method.

The distribution contains tests and documentation. The tests require
`unittest2 <http://pypi.python.org/pypi/unittest2>`_ to run.

Docs from the in-development version of `mock` can be found at
`mock.readthedocs.org <http://mock.readthedocs.org>`_.
