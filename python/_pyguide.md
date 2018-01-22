Google Python Style Guide
=================
https://google.github.io/styleguide/pyguide.html

Python Guidelines
=================
https://gist.github.com/federico-lox/65ede8a05c89ebca7be3

This document is a list of dos and don'ts for the development of Python scripts
and programs based on [Google's Python Style Guide], revision 2.59 and is to be
considered an addition to [PEP8] and [PEP257].

This is **version 1.0** of the documentation.

[google's python style guide]: http://google-styleguide.googlecode.com/svn/trunk/pyguide.html
[pep8]: http://www.python.org/dev/peps/pep-0008/
[pep257]: http://www.python.org/dev/peps/pep-0257/

Quick Reference
---------------
<!-- MarkdownTOC -->
- [1. Project Guidelines](#project-rules)
    - [1. Version](#version): Use Python 2.7
- [2. Language Guidelines](#language-rules)
    - [1. Lint](#lint): Run _pylint_ over your code
    - [2.  Imports](#imports): Use imports for packages and modules only
    - [3.  Packages](#packages): Import each module using the full pathname
      location of the module
    - [4.  Exceptions](#exceptions): Exceptions are allowed but must be used
      carefully
    - [5.  Global Variables](#global-variables): Avoid global variables
    - [6.  Local Classes and Functions](#local-classes-and-functions):
      Local/nested/inner classes and functions are fine
    - [7.  List Comprehensions](#list-comprehensions): Okay to use for simple cases
    - [8.  Default Iterators and Operators](#default-iterators-and-operators):
      Use default iterators and operators for types that support them, like lists, dictionaries, and files
    - [9.  Generators](#generators): Use generators as needed
    - [10. Lambda Functions](#lambda-functions): Use only for one-liners
    - [11. Conditional Expressions](#conditional-expressions): Use only for
      one-liners
    - [12. Default Parameters](#default-parameters): Okay in most cases
    - [13. Properties](#properties): Use to access or set data instead of simple
      accessor or setter methods
    - [14. True and False Evaluations](#true-and-false-evaluations): Use the
      "implicit" false if at all possible
    - [15. Deprecated Language Features](#deprecated-language-features):
      - Use string methods instead of the `string` module where possible
      - Use function call syntax instead of `apply`
      - Use list comprehensions and for loops instead of `filter` and `map` when
      the function argument would have been an inlined lambda anyway
      - Use for loops instead of `reduce`
    - [16. Lexical Scoping](#lexical-scoping): Okay to use
    - [17. Function and Method Decorators](#function-and-method-decorators):
      Use decorators judiciously when there is a clear advantage
    - [18. Threading](#threading): Do not rely on the atomicity of built-in types
    - [19. Power Features](#power-features): Avoid using these features.
- [3. Style Guidelines](#style-rules)
    - [1. Semicolons](#semicolons): Avoid using semicolons in  your code.
    - [2. Line Length](#line-length): Maximum line length is 80 characters.
    - [3. Parentheses](#parentheses): Use parentheses sparingly.
    - [4. Indentation](#indentation): Indent your code blocks with 4 spaces.
    - [5. Blank Lines](#blank-lines)
    - [6. Whilespace](#whitespace)
    - [7. Shebang Line](#shebang-line)
    - [8. Comments](#comments)
    - [9. Classes](#classes)
    - [10. Strings](#strings)
    - [11. File and Sockets](#files-and-sockets)
    - [12. Imports formatting](#imports-formatting)
    - [13. Statements](#statements)
    - [14. Access Control](#access-control)
    - [15. Naming](#naming)
<!-- /MarkdownTOC -->
Project Guidelines
------------------
### Version ###

**Decision**: Use Python 2.7.

**Pros**: Stable syntax and feature set, high avalability, no issues with
packages incompatibility, some of the new features from the 3.x line can be
imported selectively from the _[futures](builtin-futures)_ module or via
backported and 3rd party packages, e.g. _[concurrent.futures](concurrent-futures)_
and _[future]_.

**Cons**: Some runtime, syntax and standard library improvements from the
3.x line aren't available, if the 2.x line will ever become deprecated the code
will need to be ported to the new version, although automated tools can help
mitigating the problem, e.g. _[2to3]_ and _[futurize]_.


[builtin-futures]: http://docs.python.org/2/library/__future__.html
[concurrent-futures]: http://pythonhosted.org/futures/
[future]: http://python-future.org
[2to3]: http://docs.python.org/3/library/2to3.html
[futurize]: http://python-future.org/automatic_conversion.html


Language Guidelines
-------------------

### Lint ###

**Decision**: Make sure you run _[pylint]_ on your code, suppress warnings if
they are inappropriate so that other issues are not hidden.

To suppress warnings, you can set a line-level comment:

```python
dict = 'something awful'  # Bad Idea... pylint: disable=redefined-builtin
```

_[pylint]_ warnings are each identified by a alphanumeric code (C0112) and a
symbolic name (empty-docstring). Prefer the symbolic names in new code or when
updating existing code; if the reason for the suppression is not clear from the
symbolic name, add an explanation, uppressing in this way has the advantage
that we can easily search for suppressions and revisit them.

You can get a list of pylint warnings by doing `pylint --list-msgs`. To get
more information on a particular message, use `pylint --help-msg=C6409`.

Prefer `pylint: disable` to the deprecated older form `pylint: disable-msg`.

Unused argument warnings can be suppressed by using `_` as the identifier for
the unused argument or prefixing the argument name with `unused_`.
In situations where changing the argument names is infeasible, you can mention
them at the beginning of the function. For example:

```python
def foo(a, unused_b, unused_c, d=None, e=None):
    _ = d, e
    return a
```

**Pros**: Catches easy-to-miss errors like typos, using-vars-before-assignment,
etc.

**Cons**: _[pylint]_ isn't perfect, to take advantage of it we'll need
to sometimes:

- Write around it
- Suppress its warnings or
- Improve it

[pylint]: http://www.pylint.org


### Imports ###

**Decision**:

- Use `import x` for importing packages and modules
- Use `from x import y` where _x_ is the package prefix and _y_ is the module
  name with no prefix
- use `from x import y as z` if two modules named _y_ are to be imported or if
  _y_ is an inconveniently long name.

For example the module `sound.effects.echo` may be imported as follows:

```python
from sound.effects import echo
# ...
echo.EchoFilter(input, output, delay=0.7, atten=4)
```

Do not use relative names in imports; even if the module is in the same
package, use the full package name. This helps prevent unintentionally
importing a package twice.

**Pros**: The namespace management convention is simple. The source of each
identifier is indicated in a consistent way; `x.Obj` says that object _Obj_ is
defined in module _x_.

**Cons**: Module names can still collide, some module names are
inconveniently long.


### Packages ###

**Decision**: All new code should import each module by its full package name,
imports should be as follows:

```python
# Reference in code with complete name.
import sound.effects.echo

# Reference in code with just module name (preferred).
from sound.effects import echo
```

**Pros**: Avoids conflicts in module names, makes it easier to find modules.

**Cons**: Makes it harder to deploy code because you have to replicate the
package hierarchy.


### Exceptions ###

**Decision**: Exceptions must follow certain conditions:

- Raise exceptions as `raise MyException('Error message')` or
  `raise MyException`, do not use the two-argument form
  `(raise MyException, 'Error message')` or deprecated string-based exceptions
  `(raise 'Error message')`
- Modules or packages should define their own domain-specific base exception
  class, which should inherit from the built-in `Exception` class. The base
  exception for a module should be called _Error_

```python
class Error(Exception):
    pass
```

- Never use `catch-all except:` statements or
  `catch Exception or StandardError` unless you are re-raising the exception or
  in the outermost block in your thread (and printing an error message); Python
  is very tolerant in this regard and `except:` will really catch everything
  including misspelled names, `sys.exit()` calls, _Ctrl+C_ interrupts, unittest
  failures and all kinds of other exceptions that you simply don't want to
  catch
- Minimize the amount of code in a try/except block; the larger the body of the
  try, the more likely that an exception will be raised by a line of code that
  you didn't expect to raise an exception. In those cases, the try/except block
  hides a real error
- Use the finally clause to execute code whether or not an exception is raised
  in the try block; this is often useful for cleanup, i.e. closing a file
- When capturing an exception, use as rather than a comma; for example:

```python
try:
    raise Error
except Error as error:
    pass
```

**Pros**: The control flow of normal operation code is not cluttered by
error-handling code; it also allows the control flow to skip multiple frames
when a certain condition occurs, e.g. returning from N nested functions in one
step instead of having to carry-through error codes.

**Cons**: May cause the control flow to be confusing. Easy to miss error cases
when making library calls.


### Global Variables ###

**Decision**: Avoid global variables in favor of class variables,
some exceptions are:

- Default options for scripts
- Module-level constants, e.g. `PI = 3.14159`; constants should be named using
  all caps with underscores, see _[Naming]_ below
- It is sometimes useful for globals to cache values needed or returned by
  functions
- If needed, globals should be made internal to the module and accessed through
  public module level functions, see _[Naming]_ below

**Pros**: Occasionally useful.

**Cons**: Has the potential to change module behavior during the import,
because assignments to module-level variables are done when the module
is imported.

[naming]: #naming


### Local Classes and Functions ###

**Decision**: A class can be defined inside of a method/function/class,
a function can be defined inside a method or function, nested functions have
read-only access to variables defined in enclosing scopes.

**Pros**: Allows definition of utility classes and functions that are only used
inside of a very limited scope.

**Cons**: Instances of nested or local classes cannot be pickled.


### List Comprehensions ###

**Decision**: Okay to use for simple cases, each portion must fit on one line
(mapping expression, for clause, filter expression); multiple for clauses or
filter expressions are not permitted, use loops instead when things get more
complicated.

```python
# Yes

result = []
for x in range(10):
    for y in range(5):
        if x * y > 10:
            result.append((x, y))

for x in xrange(5):
    for y in xrange(5):
        if x != y:
            for z in xrange(5):
                if y != z:
                    yield (x, y, z)

return ((x, complicated_transform(x))
    for x in long_generator_function(parameter)
        if x is not None)

squares = [x * x for x in range(10)]

eat(jelly_bean for jelly_bean in jelly_beans if jelly_bean.color == 'black')

# No

result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]

return ((x, y, z)
    for x in xrange(5)
    for y in xrange(5)
    if x != y
    for z in xrange(5)
    if y != z)
```

**Pros**: Simple list comprehensions can be clearer and simpler than other list
creation techniques; generator expressions can be very efficient, since they
avoid the creation of a list entirely.

**Cons**: Complicated list comprehensions or generator expressions can be hard
to read.


### Default Iterators and Operators ###

**Decision**: Use default iterators and operators for types that support them,
like lists, dictionaries, and files; The built-in types define iterator
methods too, prefer these methods to methods that return lists, except that you
should not mutate a container while iterating over it.

```python
# Yes

for key in adict:
    pass

if key not in adict:
    pass

if obj in alist:
    pass

for line in afile:
    pass

for k, v in dict.iteritems():
    pass

# No

for key in adict.keys():
    pass

if not adict.has_key(key):
    pass

for line in afile.readlines():
    pass
```

**Pros**: The default iterators and operators are simple and efficient, they
express the operation directly, without extra method calls. A function that
uses default operators is generic, it can be used with any type that supports
the operation.

**Cons**: You can't tell the type of objects by reading the method names
(e.g. `has_key()` means a dictionary), this is also an advantage.


### Generators ###

**Decision**: Use whenever appropriate, the doc strings for generator functions
sould include `Yields:` rather than `Returns:`.

**Pros**: Simpler code, because the state of local variables and control flow
are preserved for each call, generators also use less memory than functions
creating an entire list of values at once.

**Cons**: None.


### Lambda Functions ###

**Decision**: Use _lambda functions_ only for one-liners but if the code inside
function is any longer than 60–80 chars, then it's probably better to define
it as a regular (nested) function.

For common operations like multiplication, use the functions from the
_operator module_ instead of _lambda functions_, e.g. prefer `operator.mul` to
`lambda x, y: x * y`.

**Pros**: Convenient.

**Cons**: Harder to read and debug than local functions; the lack of names
means stack traces are more difficult to understand, expressiveness is also
limited because the function may only contain an expression.


### Conditional Expressions ###

**Decision**: Use only for one-liners, in other cases prefer to use a complete
_if statement_.

**Pros**: Shorter and more convenient than an _if statement_.

**Cons**: May be harder to read than an _if statement_, the condition may be
difficult to locate if the expression is long.


### Default Argument Values ###

**Decision**: Okay to use with the exception of mutable objects as default
values in the function or method definition.

```python
# Yes

def foo(a, b=None):
    if b is None:
        b = []

# No

def foo(a, b=[]):
    pass

def foo(a, b=time.time()): # The time the module was loaded???
    pass

def foo(a, b=FLAGS.my_thing): # sys.argv has not yet been parsed...
    pass
```

**Pros**: Often you have a function that uses lots of default values,
but—rarely—you want to override the defaults. Default argument values provide an
easy way to do this, without having to define lots of functions for the rare
exceptions. Also, Python does not support overloaded methods/functions and
default arguments are an easy way of "faking" the overloading behavior.

**Cons**: Default arguments are evaluated once at module load time. This may
cause problems if the argument is a mutable object such as a list or a
dictionary. If the function modifies the object (e.g., by appending an item to a
list), the default value is modified.


### Properties ###

**Decision**: Use properties to access or set data instead of accessor or setter
methods; read-only properties should be created with the `@property` decorator.

Inheritance with properties can be non-obvious if the property itself is not
overridden, thus one must make sure that accessor methods are called indirectly
to ensure methods overridden in subclasses are called by the property (using the
Template Method DP).

```python
# Yes

import math

class Square(object):
    """A square with two properties: a writable area and a read-only perimeter.

    To use:
    >>> sq = Square(3)
    >>> sq.area
    9
    >>> sq.perimeter
    12
    >>> sq.area = 16
    >>> sq.side
    4
    >>> sq.perimeter
    16
    """

    def __init__(self, side):
        self.side = side

    def __get_area(self):
        """Calculates the 'area' property."""
        return self.side ** 2

    def ___get_area(self):
        """Indirect accessor for 'area' property."""
        return self.__get_area()

    def __set_area(self, area):
        """Sets the 'area' property."""
        self.side = math.sqrt(area)

    def ___set_area(self, area):
        """Indirect setter for 'area' property."""
        self.__set_area(area)

    area = property(___get_area, ___set_area,
                    doc="""Gets or sets the area of the square.""")

    @property
    def perimeter(self):
        return self.side * 4
```

**Pros**: Properties are considered the pythonic way to maintain the interface
of a class allowing accessor methods to be added in the future without breaking
the interface increasing readability. In terms of performance, allowing
properties bypasses needing trivial accessor methods when a direct variable
access is reasonable and calculations can be delayed/made lazy.


**Cons**: Properties are specified after the getter and setter methods are
declared, requiring one to notice they are used for properties farther down in
the code (except for readonly properties created with the @property decorator),
and classes must inherit from object. This can also hide side-effects much like
operator overloading and could be confusing for subclasses.


### True and False Evaluations ###

**Decision**: Use the "implicit" false if at all possible, e.g. `if foo:`
rather than `if foo != []:`, there are a few caveats that you should keep in mind though:

- Never use `==` or `!=` to compare singletons like `None`, use `is` or
  `is not`
- Beware of writing `if x:` when you really mean `if x is not None:`, e.g. when
  testing whether a variable or argument that defaults to `None` was set to
  some other value; the other value might be a value that's false in a boolean
  context!
- Never compare a boolean variable to `False` using `==`, use `if not x:`
  instead; if you need to distinguish `False` from `None` then chain the
  expressions, such as `if not x and x is not None:`
- For sequences (strings, lists, tuples), use the fact that empty sequences are
  false, so `if not seq:` or `if seq:` is preferable to `if len(seq):` or
  `if not len(seq):`
- When handling integers, implicit false may involve more risk than benefit
  (i.e. accidentally handling `None` as `0`); you may compare a value which is
  known to be an integer (and is not the result of `len()`) against the integer
  `0`

```python
# Yes

if not users:
    print 'no users'

if foo == 0:
    self.handle_zero()

if i % 10 == 0:
    self.handle_multiple_of_ten()

# No

if len(users) == 0:
    print 'no users'

if foo is not None and not foo:
    self.handle_zero()

if not i % 10:
    self.handle_multiple_of_ten()
```

- Note that `'0'` (i.e., `0` as string) evaluates to true

**Pros**: Conditions using Python booleans are easier to read and less
error-prone, in most cases they're also faster.

**Cons**: May look strange to C/C++ developers.


### Deprecated Language Features ###

**Decision**:

- Use string methods instead of the `string` module where possible
- Use function call syntax instead of `apply`
- Use list comprehensions and for loops instead of `filter` and `map` when the
  function argument would have been an inlined lambda anyway
- Use for loops instead of `reduce`

```python
# Yes

words = foo.split(':')

[x[1] for x in my_list if x[2] == 5]

map(math.sqrt, data) # No inlined lambda expression

fn(*args, **kwargs)

# No

import string
words = string.split(foo, ':')

map(lambda x: x[1], filter(lambda x: x[2] == 5, my_list))

apply(fn, args, kwargs)
```

**Pros**: Cleaner, more readable code, in most cases faster.

**Cons**: We do not use any Python version which does not support these
features, so there is no reason not to use the new styles.


### Lexical Scoping ###

**Decision**: Okay to use, any assignment to a name in a block will cause
Python to treat all references to that name as a local variable, even if the
use precedes the assignment; if a global declaration occurs, the name is
treated as a global variable.

Try to avoid reusing names in both local and global scopes to avoid bugs:

```python
# Yes

def get_adder(summand1):
    """Returns a function that adds numbers to a given number."""

    def adder(summand2):
        return summand1 + summand2

    return adder

# No

i = 4 # i is global
def foo(x):
    def bar():
        print i, # No, wait... is local!

    # ...
    # A bunch of code here
    # ...
    for i in x:
        print i,

    bar()

# foo([1, 2, 3]) prints 1 2 3 3, not 1 2 3 4
```

**Pros**: Often results in clearer, more elegant code.

**Cons**: Can lead to confusing bugs (see example above), when used in local
functions that are returned could lead to references not being garbage
collected.


### Function and Method Decorators ###

**Decision**: Use decorators judiciously when there is a clear advantage;
decorators should follow the same import and naming guidelines as functions,
their pydoc string should clearly state that the function is a decorator and
they should be accompanied by unit tests.

Avoid external dependencies in the decorator itself (e.g. don't rely on files,
sockets, database connections, etc.) since they might not be available when the
decorator runs (at import time, perhaps from pydoc or other tools); a decorator
that is called with valid parameters should be guaranteed to succeed in all
cases.

Decorators are a special case of "top level code", see [main] for more discussion.

**Pros**: Elegantly specifies some transformation on a method; the
transformation might eliminate some repetitive code, enforce invariants, etc.

**Cons**: Decorators can perform arbitrary operations on a function's arguments
or return values, resulting in surprising implicit behavior; additionally,
decorators execute at import time, failures in decorator code are pretty much
impossible to recover from.


[main]: #main


### Threading ###

**Decision**: Don't rely on Python's built-in data types' atomic operations,
there are corner cases where they aren't atomic (e.g. if `__hash__` or
`__eq__` are implemented as Python methods) and their  atomicity should not be
relied upon, neither should you rely on atomic variable assignment (since this
in turn depends on dictionaries).

Use the `queue.Queue` data type as the preferred way
to communicate data between threads, otherwise use the `threading` module and
its locking primitives.

Learn about the proper use of condition variables so you can use
`threading.Condition` instead of using lower-level locks.

**Pros**: code that runs over multiple threads will be safer and more robust.

**Cons**: additional dependencies, more complicated syntax.


### Power Features ###

**Decision**: Avoid using features such as metaclasses, access to bytecode,
on-the-fly compilation, dynamic inheritance, object reparenting, import hacks,
reflection, modification of system internals, etc. in your code.

**Pros**: These are powerful language features which can make your code more
compact.

**Cons**: It's very tempting to use these "cool" features when they're not
absolutely necessary, they can make code that's using unusual features
underneath harder to read, understand, and debug; It doesn't seem that way at
first, but when revisiting the code it tends to be more difficult than code
that is longer but is straightforward.


Style Guidelines
----------------

### Semicolons ###

**Decision**: Do not terminate your lines with semi-colons and do not use
semi-colons to put two commands on the same line.


### Line length ###

**Decision**: Maximum line length is 80 characters, with two exceptions:

- Long import statements
- URLs in comments

Also do not use backslash line continuation but rather make use of Python's
implicit line joining inside parentheses, brackets and braces; if necessary,
you can add an extra pair of parentheses around an expression.

When a literal string won't fit on a single line, use parentheses for implicit
line joining.

Within comments, put long URLs on their own line if necessary.

See the [indentation] section for explanation of how to handle indenting line
continuations.

```python
# Yes

foo_bar(self, width, height, color='black', design=None, x='foo',
        emphasis=None, highlight=0)

if (width == 0 and height == 0 and
    color == 'red' and emphasis == 'strong'):

x = ('This will build a very long long '
     'long long long long long long string')

# See details at
# http://www.example.com/...long long URL.../specification.html

# No

if width == 0 and height == 0 and \
    color == 'red' and emphasis == 'strong':

x = 'This will build a very long long ' \
    'long long long long long long string'

# See details at
# http://www.example.com/us/developer/documentation/api/content/\
# v2.0/csv_file_name_extension_full_specification.html
```


[indentation]: #indentation


### Parentheses ###

**Decision**: Do not use them in return statements or conditional statements
unless using parentheses for implied line continuation (see [line length]),
it is however fine to use parentheses around tuples.

```python
# Yes

if foo:
    bar()

while x:
    x = bar()

if x and y:
    bar()

if not x:
    bar()

return foo

for (x, y) in dict.items():
    pass

# No

if (x):
    bar()

if not(x):
    bar()

return (foo)
```

[Indentation]: #indentation
### Indentation ###
**Decision**: Use 4 spaces to indent your code blocks, never use tabs or mix
tabs and spaces.

In cases of implied line continuation, you should align wrapped elements either
vertically (see the examples in the [line length] section) or using a hanging
indent of 4 spaces, in which case there should be no argument on the first line.
```python
# Yes

# Aligned with opening delimiter
foo = long_function_name(var_one, var_two,
                         var_three, var_four)

# Aligned with opening delimiter in a dictionary
foo = {
    long_dictionary_key: value1 +
    value2,
    value3,
    value4
}

# 4-space hanging indent; nothing on first line
foo = long_function_name(
    var_one, var_two, var_three,
    var_four)

# 4-space hanging indent in a dictionary
foo = {
    long_dictionary_key:
        long_dictionary_value,
    long_dictionary_key2:
        long_dictionary_value2
}

# No

# Stuff on first line forbidden
foo = long_function_name(var_one, var_two,
    var_three, var_four)

# 2-space hanging indent forbidden
foo = long_function_name(
  var_one, var_two, var_three,
  var_four)

# No hanging indent in a dictionary
foo = {
    long_dictionary_key:
        long_dictionary_value,
    long_dictionary_key2:
        long_dictionary_value2}
```

[Blank Lines]: #blank-lines
### Blank Lines ###
* Two blank lines between top-level definitions (function or class)
* One blank line between method definitions and between the class line and the first method. Use single blank lines as you judge appropriate within functions or methods.

[Whilespace]: #whitespace
### Whitespace ###
* No whitespace inside parentheses, brackets or braces.
```python
#Yes
spam(ham[1], {eggs: 2}, [])

#No
spam( ham[ 1 ], { eggs: 2 }, [ ] )
```
* No whitespace before a comma, semicolon, or colon. Do use whitespace after a comma, semicolon, or colon except at the end of the line.
```python
#Yes:
if x == 4:
    print x, y
x, y = y, x
#No:
if x == 4 :
    print x , y
x , y = y , x
```
* No whitespace before the open paren/bracket that starts an argument list, indexing or slicing.
```python
#Yes:
spam(1)
#No:  
spam (1)
#Yes:
dict['key'] = list[index]
#No:  
dict ['key'] = list [index]
```
* Surround binary operators with a single space on either side for assignment (=), comparisons (==, <, >, !=, <>, <=, >=, in, not in, is, is not), and Booleans (and, or, not). Use your better judgment for the insertion of spaces around arithmetic operators but always be consistent about whitespace on either side of a binary operator.
```python
#Yes:
x == 1
#No:  
x<1
```
* Don't use spaces around the '=' sign when used to indicate a keyword argument or a default parameter value.
```python
#Yes:
def complex(real, imag=0.0): return magic(r=real, i=imag)
#No:  
def complex(real, imag = 0.0): return magic(r = real, i = imag)
```
* Don't use spaces to vertically align tokens on consecutive lines, since it becomes a maintenance burden (applies to :, #, =, etc.):
```python
#Yes:
  foo = 1000  # comment
  long_name = 2  # comment that should not be aligned

  dictionary = {
      'foo': 1,
      'long_name': 2,
  }
#No:
  foo       = 1000  # comment
  long_name = 2     # comment that should not be aligned

  dictionary = {
      'foo'      : 1,
      'long_name': 2,
  }
```

[Shebang Line]:#shebang-line
### Shebang Line ###
Most *.py* files do not need to start with a #! line. Start the main file of a program with *#!/usr/bin/env* python with an optional single digit 2 or 3 suffix. This line is used by the kernel to find the Python interpreter, but is ignored by Python when importing modules. It is only necessary on a file that will be executed directly.

[Comments]:#comments
### Comments ###
Be sure to use the right style for module, function, method and in-line comments.
#### Doc Strings ####
Python has a unique commenting style using doc strings. A doc string is a string that is the first statement in a package, module, class or function. These strings can be extracted automatically through the ```python __doc__``` member of the object and are used by *pydoc*. (Try running *pydoc* on your module to see how it looks.) We always use the three double-quote """ format for doc strings (per PEP 257). A doc string should be organized as a summary line (one physical line) terminated by a period, question mark, or exclamation point, followed by a blank line, followed by the rest of the doc string starting at the same cursor position as the first quote of the first line. There are more formatting guidelines for doc strings below.

#### Modules ####
Every file should contain license boilerplate. Choose the appropriate boilerplate for the license used by the project (for example, Apache 2.0, BSD, LGPL, GPL)

### Functions and Methods ###
As used in this section "function" applies to methods, function, and generators.

A function must have a *docstring*, unless it meets all of the following criteria:

* not externally visible
* very short
* obvious

A *docstring* should give enough information to write a call to the function without reading the function's code. A *docstring* should describe the function's calling syntax and its semantics, not its implementation. For tricky code, comments alongside the code are more appropriate than using *docstrings*.

Certain aspects of a function should be documented in special sections, listed below. Each section begins with a heading line, which ends with a colon. Sections should be indented two spaces, except for the heading.

Args:
* List each parameter by name. A description should follow the name, and be separated by a colon and a space. If the description is too long to fit on a single 80-character line, use a hanging indent of 2 or 4 spaces (be consistent with the rest of the file).
* The description should mention required type(s) and the meaning of the argument.
* If a function accepts ```python *foo*``` (variable length argument lists) and/or *bar* (arbitrary keyword arguments), they should be listed as ```python *foo* and **bar```.

Returns: (or Yields: for generators)
* Describe the type and semantics of the return value. If the function only returns None, this section is not required.
Raises:
* List all exceptions that are relevant to the interface.
```python
def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
    """Fetches rows from a Bigtable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by big_table.  Silly things may happen if
    other_silly_variable is not None.

    Args:
        big_table: An open Bigtable Table instance.
        keys: A sequence of strings representing the key of each table row
            to fetch.
        other_silly_variable: Another optional variable, that has a much
            longer name than the other args, and which does nothing.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {'Serak': ('Rigel VII', 'Preparer'),
         'Zim': ('Irk', 'Invader'),
         'Lrrr': ('Omicron Persei 8', 'Emperor')}

        If a key from the keys argument is missing from the dictionary,
        then that row was not found in the table.

    Raises:
        IOError: An error occurred accessing the bigtable.Table object.
    """
    pass
```
#### Classes ####

Classes should have a doc string below the class definition describing the class. If your class has public attributes, they should be documented here in an Attributes section and follow the same formatting as a function's Args section.
```python
class SampleClass(object):
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam=False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""
```
#### Block and Inline Comments ####
* The final place to have comments is in tricky parts of the code. If you're going to have to explain it at the next code review, you should comment it now. Complicated operations get a few lines of comments before the operations commence. Non-obvious ones get comments at the end of the line.
```python
# We use a weighted dictionary search to find out where i is in
# the array.  We extrapolate position based on the largest num
# in the array and the array size and then do binary search to
# get the exact number.

if i & (i-1) == 0:        # true iff i is a power of 2
```
* To improve legibility, these comments should be *at least 2 spaces* away from the code.
* On the other hand, never describe the code. Assume the person reading the code knows Python (though not what you're trying to do) better than you do.
```python
# BAD COMMENT: Now go through the b array and make sure whenever i occurs
# the next element is i+1
```
[Classes]:#classes
### Classes ###
If a class inherits from no other base classes, explicitly inherit from object. This also applies to nested classes.
```python
 #Yes:
    class SampleClass(object):
         pass


    class OuterClass(object):

         class InnerClass(object):
             pass


    class ChildClass(ParentClass):
         """Explicitly inherits from another class already."""

#No:
    class SampleClass:
        pass


    class OuterClass:

        class InnerClass:
            pass
```
Inheriting from object is needed to make properties work properly, and it will protect your code from one particular potential incompatibility with Python 3000. It also defines special methods that implement the default semantics of objects including ```__new__, __init__, __delattr__, __getattribute__, __setattr__, __hash__, __repr__, and __str__.```

[Strings]:#strings
### Strings ###
Use the format method or the ``` %``` operator for formatting strings, even when the parameters are all strings. Use your best judgement to decide between + and % (or format) though.
```python
 Yes: x = a + b
     x = '%s, %s!' % (imperative, expletive)
     x = '{}, {}!'.format(imperative, expletive)
     x = 'name: %s; score: %d' % (name, n)
     x = 'name: {}; score: {}'.format(name, n)
No: x = '%s%s' % (a, b)  # use + in this case
    x = '{}{}'.format(a, b)  # use + in this case
    x = imperative + ', ' + expletive + '!'
    x = 'name: ' + name + '; score: ' + str(n)
```
Avoid using the + and += operators to accumulate a string within a loop. Since strings are immutable, this creates unnecessary temporary objects and results in quadratic rather than linear running time. Instead, add each substring to a list and ''.join the list after the loop terminates (or, write each substring to a io.BytesIO buffer).
```python
Yes: items = ['<table>']
     for last_name, first_name in employee_list:
         items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
     items.append('</table>')
     employee_table = ''.join(items)
No: employee_table = '<table>'
    for last_name, first_name in employee_list:
        employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
    employee_table += '</table>'
```
Be consistent with your choice of string quote character within a file. Pick ' or " and stick with it. It is okay to use the other quote character on a string to avoid the need to \ escape within the string. GPyLint enforces this.
```python
Yes:
  Python('Why are you hiding your eyes?')
  Gollum("I'm scared of lint errors.")
  Narrator('"Good!" thought a happy Python reviewer.')
No:
  Python("Why are you hiding your eyes?")
  Gollum('The lint. It burns. It burns us.')
  Gollum("Always the great lint. Watching. Watching.")
```
Prefer """ for multi-line strings rather than '''. Projects may choose to use ''' for all non-docstring multi-line strings if and only if they also use ' for regular strings. Doc strings must use """ regardless. Note that it is often cleaner to use implicit line joining since multi-line strings do not flow with the indentation of the rest of the program:
```python
Yes:
  print ("This is much nicer.\n"
         "Do it this way.\n")
  No:
    print """This is pretty ugly.
Don't do this.
"""
```

[File and Sockets]:#files-and-sockets
### Files and Sockets ###
Explicitly close files and sockets when done with them.
Leaving files, sockets or other file-like objects open unnecessarily has many downsides, including:
* They may consume limited system resources, such as file descriptors. Code that deals with many such objects may exhaust those resources unnecessarily if they're not returned to the system promptly after use.
* Holding files open may prevent other actions being performed on them, such as moves or deletion.
* Files and sockets that are shared throughout a program may inadvertantly be read from or written to after logically being closed. If they are actually closed, attempts to read or write from them will throw exceptions, making the problem known sooner.
Furthermore, while files and sockets are automatically closed when the file object is destructed, tying the life-time of the file object to the state of the file is poor practice, for several reasons:

There are no guarantees as to when the runtime will actually run the file's destructor. Different Python implementations use different memory management techniques, such as delayed Garbage Collection, which may increase the object's lifetime arbitrarily and indefinitely.
Unexpected references to the file may keep it around longer than intended (e.g. in tracebacks of exceptions, inside globals, etc).
The preferred way to manage files is using the "with" statement:
```python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print line
```
For file-like objects that do not support the "with" statement, use contextlib.closing():
```python
import contextlib

with contextlib.closing(urllib.urlopen("https://www.python.org/")) as front_page:
    for line in front_page:
        print line
```
Legacy AppEngine code using Python 2.5 may enable the "with" statement using ```"from __future__ import with_statement".```

### TODO Comments ###
Use TODO comments for code that is temporary, a short-term solution, or good-enough but not perfect.

TODOs should include the string TODO in all caps, followed by the name, e-mail address, or other identifier of the person who can best provide context about the problem referenced by the TODO, in parentheses. A colon is optional. A comment explaining what there is to do is required. The main purpose is to have a consistent TODO format that can be searched to find the person who can provide more details upon request. A TODO is not a commitment that the person referenced will fix the problem. Thus when you create a TODO, it is almost always your name that is given.
```python
# TODO(kl@gmail.com): Use a "*" here for string repetition.
# TODO(Zeke) Change this to use relations.
```
If your TODO is of the form "At a future date do something" make sure that you either include a very specific date ("Fix by November 2009") or a very specific event ("Remove this code when all clients can handle XML responses.").

[Imports formatting]:#imports-formatting
### Imports formatting ###
Imports should be on separate lines.
E.g.:
```python
Yes: import os
     import sys
No:  import os, sys
```
Imports are always put at the top of the file, just after any module comments and doc strings and before module globals and constants. Imports should be grouped with the order being most generic to least generic:
* standard library imports
* third-party imports
* application-specific imports

Within each grouping, imports should be sorted lexicographically, ignoring case, according to each module's full package path.
```python
import foo
from foo import bar
from foo.bar import baz
from foo.bar import Quux
from Foob import ar
```
[Statements]:#statements
### Statements ###
Generally only one statement per line.
However, you may put the result of a test on the same line as the test only if the entire statement fits on one line. In particular, you can never do so with try/except since the try and except can't both fit on the same line, and you can only do so with an if if there is no else.
```python
Yes:

  if foo: bar(foo)
No:

  if foo: bar(foo)
  else:   baz(foo)

  try:               bar(foo)
  except ValueError: baz(foo)

  try:
      bar(foo)
  except ValueError: baz(foo)
```
[Access Control]:#access-control
### Access Control ###
If an accessor function would be trivial you should use public variables instead of accessor functions to avoid the extra cost of function calls in Python. When more functionality is added you can use property to keep the syntax consistent.

On the other hand, if access is more complex, or the cost of accessing the variable is significant, you should use function calls (following the Naming guidelines) such as ```get_foo() and set_foo().``` If the past behavior allowed access through a property, do not bind the new accessor functions to the property. Any code still attempting to access the variable by the old method should break visibly so they are made aware of the change in complexity.

[Naming]:#naming
### Naming ###
```module_name, package_name, ClassName, method_name, ExceptionName, function_name, GLOBAL_CONSTANT_NAME, global_var_name, instance_var_name, function_parameter_name, local_var_name```

#### Names to Avoid ####
* single character names except for counters or iterators
* dashes (-) in any package/module name
* ```__double_leading_and_trailing_underscore__ names (reserved by Python)```

#### Naming Convention ####
* "Internal" means internal to a module or protected or private within a class.
* Prepending a single underscore (_) has some support for protecting module variables and functions (not included with import * from). Prepending a double underscore (__) to an instance variable or method effectively serves to make the variable or method private to its class (using name mangling).
* Place related classes and top-level functions together in a module. Unlike Java, there is no need to limit yourself to one class per module.
* Use CapWords for class names, but lower_with_under.py for module names. Although there are many existing modules named CapWords.py, this is now discouraged because it's confusing when the module happens to be named after a class. ```("wait -- did I write import StringIO or from StringIO import StringIO?")```

#### Guidelines derived from Guido's Recommendations ####
| Type          | Public            | Internal                |
| ------------- |:-----------------:| -----------------------:|
| Packages	    | lower_with_under  |                         |
| Modules	      | lower_with_under	| _lower_with_under       |
| Classes     	| CapWords          | _CapWords               |
| Exceptions	  | CapWords          |                         |
| Functions	    | lower_with_under()|	_lower_with_under()     |
| Global/Class Constants|	CAPS_WITH_UNDER     |	_CAPS_WITH_UNDER  |
| Global/Class Variables|	lower_with_under    |	_lower_with_under |
| Instance Variables	  | lower_with_under    |	_lower_with_under (protected) or __lower_with_under (private)   |
| Method Names	        | lower_with_under()  |	_lower_with_under() (protected) or __lower_with_under() (private) |
| Function/Method Parameters|	lower_with_under|
| Local Variables	      |lower_with_under     |                   |

### Main ###
Even a file meant to be used as a script should be importable and a mere import should not have the side effect of executing the script's main functionality. The main functionality should be in a main() function.
In Python, pydoc as well as unit tests require modules to be importable. Your code should always check if ```__name__ == '__main__'``` before executing your main program so that the main program is not executed when the module is imported.
```python
def main():
      ...

if __name__ == '__main__':
    main()
```
All code at the top level will be executed when the module is imported. Be careful not to call functions, create objects, or perform other operations that should not be executed when the file is being pydoced.

### Parting Words ###
BE CONSISTENT.
If you're editing code, take a few minutes to look at the code around you and determine its style. If they use spaces around all their arithmetic operators, you should too. If their comments have little boxes of hash marks around them, make your comments have little boxes of hash marks around them too.

The point of having style guidelines is to have a common vocabulary of coding so people can concentrate on what you're saying rather than on how you're saying it. We present global style rules here so people know the vocabulary, but local style is also important. If code you add to a file looks drastically different from the existing code around it, it throws readers out of their rhythm when they go to read it. Avoid this.
