# Giorgio's guide to Python

A futile attempt to stop the takeover of the snakes by illuminating people on the dangers of them.

You will understand this article a bit more if you're a programmer.

## Managing `python` versions

Multiple `python` versions cannot live in the same system. To manage them it's highly suggested to use a version manager. `pyenv` is the weapon of choice for this work.

### `pyenv` basic guide

`pyenv versions` shows the installed python versions:

```
❯ pyenv versions
* system (set by /Users/giorgio/.pyenv/version)
3.8.13
3.9.13
3.10.8
* 3.11.3 (set by /Users/giorgio/.pyenv/version)
```

`pyenv install --list` shows the installable versions (use grep to filter)

`pyenv install/uninstall major.minor.patch` does what it says on the tin

the really important commands are `pyenv local/global pythonVersion` , e.g. `python local 3.11.3` those sets the global (system) python version or the local (directory + subdirs) version. I never touch the global one, i just set a local in every directory i want to use python with.

Using `python local x.y.z` will generate a `.python-version` file (it's not a bad idea to put it in gitignore, but it's also quite fine to have it in the repo to esplicitly declare which version should be the "blessed" one for the project. You can omit the patch number in the file.) that will enable `pyenv` to understand what python version to invoke. It's a simple text file with a version number:

```
❯ cat .python-version
3.11.3
```

### Which version should I use?

Usually `latest` is fine, specially after Python 3.9. Before 3.9, there has been some breakages/feature changes in between 3.6->3.9 like new operators, dictionaries (basically hashmaps) became insertion-ordered and so on. Try to stay at least on 3.9, almost forced if you want to use type hints.

### virtualenvs

We installed pyenv, we can now go try it:

```sh
pyenv install 3.11.3
cd ~
mkdir python_test
cd python_test
pyenv local 3.11.3
python --version
```

And we should see the correct version. Keep in mind it's still a GLOBAL install; even if you control the version per directory, if I have another directory with a 3.11.3 version they will share installed packages, and so on.

To have a real sandbox, we need another step: the virtualenv. Long story short a virtualenv is a copy of the standard library and a series of shell script that redirects installation of packages into the sandbox environment.

Many tools promise to manage venvs for you, but dont let them do that: generate them manually, always in the project directory, always in the `.venv` dir. This is a commonly accepted best practice (i.e. pycharm automatically resolved the virtualenv if it's there). How to generate a virtualenv?

```sh
python -m venv .venv
```

This invokes python's stdlib `venv` module, which when invoked like this produces a new virtualenv in the directory specified with the first argument, creating the dir if not existing.

Once the virtualenv has been generated, you need to *activate* it with this command:

```sh
source .venv/bin/activate
```

Your prompt will change to signal you're now "inside" the venv. You can now install packages locally, just for the current virtualenv:

```sh
pip install ipython requests
```

To get out of the venv, just use the `deactivate` command (don't do that right now!). In general I just close the shell rather then deactivate, less room for error.

### Package managers

This is Python weakest point. The community is hell-bent on producing package managers and web frameworks in search of the next seat at the rockstar dev table, but almost all of them are of terrible quality, with unclear governance and useless docs.

The only blessed way to install packages is `pip`, which does not produce a lockfile and other nasty things.

`pip` basic guide:

* `pip install/unistall foo` - does what it says on the tin
* `pip freeze` - shows all installed packages. transitive dependencies are listed too (i.e. you install foo depending on bar, pip freeze will show foo and bar)
* `pip freeze > requirements.txt` produces a requirements file, the "old" way of telling people what was needed for the project
* `pip install -r requirements.txt` install a requirements file.

In our org, `poetry` is usually used, but I'm not an expert: I installed it, then I get into my manually generated virtualenv and run `poetry install` or `poetry add foo` to add a dependency. Let's not go deeper than that for now, it's really the worst part of the ecosystem. Be careful as `poetry` is misbehaved and they don't use SemVer. Many breaking changes appear in between minors.

If I could have had my choice, `pip-tools` is the best package manager as it's basically a wrapper over `pip` that automates the production and syncronization of requirement files. But I didn't have my choice and now you have to suffer.

### `pyproject.toml`

How to package python and to specify project metadata has been changed like a gazillion times in between PEPs (PEP = Python Enhancement Proposal, basically RFCs that are voted and implemented to change the lang).

The latest fad is the `pyproject.toml` file, which is used to specify dependencies, metadata, configure some tools. It's best to leave it managed by `poetry` for non library projects, as metadata is useful only for publication on PyPI, the package repo.

https://github.com/casavo/casavo-log-formatter/blob/main/pyproject.toml take a look here for a pypi-enabled pyproject toml, or our projects for some more examples.

### The REPL

Python has a moderately good REPL; it's not on Clojure's level, but it's workable. I made you install `ipython`, which is way better as it has autocomplete, syntax highlight, and so on.

Run it with the `ipython` command:

```❯ ipython
Python 3.11.3 (main, May 29 2023, 10:23:41) [Clang 14.0.3 (clang-1403.0.22.14.1)]
Type 'copyright', 'credits' or 'license' for more information
IPython 8.15.0 -- An enhanced Interactive Python. Type '?' for help.

In [ ]:
```

you can now play with the language.

```IPython
In [ ]: 2 + 2
Out[ ]: 4

In [ ]: name = "Giorgio"

In [ ]: print(f"Hello, {name}!")
Hello, Giorgio!

In [ ]: import requests

In [ ]: response = requests.get("https://httpbin.org/get")

In [ ]: response.json()
Out[ ]:
{'args': {},
'headers': {'Accept': '*/*',
'Accept-Encoding': 'gzip, deflate',
'Host': 'httpbin.org',
'User-Agent': 'python-requests/2.31.0',
'X-Amzn-Trace-Id': 'Root=1-6501c713-0d5b7a74419632b9319462cc'},
'origin': '93.147.246.47',
'url': 'https://httpbin.org/get'}
```

Some anatomy lessons: returns are annotated on `Out` lines, string interpolation is done by prepending `f` on a string literal, assignation is `=`, packages are imported by doing `import foo` or `from foo import specific_thing`. String can be specified with `""` or `''` quoting. `""` is more idiomatic. `requests` is just a http client library, and we used a function from the package with the `.get` syntax.

You can read docstring in REPL with the `help()` command, try `help(requests.get)`.

In python everything is an object, so you can also use `dir(foo)` to see all the properties, methods, fields and so on of an instance.

```
In [ ]: help(requests.get)

In [ ]: dir(response)
Out[ ]:
['__attrs__',
'__bool__',
'__class__',
'__delattr__',

... OMITTED FOR BREVITY

'ok',
'raise_for_status',
'raw',
'reason',
'request',
'status_code',
'text',
'url']
```

Use `quit()` to quit the REPL. Have fun!

## Language, quirks, best practices and conventions

### Basic characteristics

Python is a strongly typed dynamic language. It has structural subtyping aka duck typing. If I want something with a `quack()` method, it could be a Duck, Train, Integer instance and it would work nonetheless if it has the correct method.

There is no implicit type coercion, so `1 + "ah"` will fail with a `TypeError`.

#### Indentation

Yes python has significant whitespace and no braces. Use 4 spaces. Never use tabs. Never mix them up. It used to be a matter of style but industry standardized on 4 spaces. Use an `.editorconfig` aware editor + a formatter and be done with it.

Scopes are delimited by the indentation level.

#### Visibility

There is no concept of visibility in python. Everything is public. A convention (which some IDE helps enforce) is to prepend functions/classes/methods names with an `_` if they must be considered package private (`internal` in Java i think?).

TODO: module or package private?

There is also the double underscore prefix `__` that does something called name mangling. Don't use it, it's pointless, but also consider double underscore prefixed things as package private.

#### Dunders/magic methods

By implementing some magic methods on your classes, you can make them behave as primitive types and/or interact with keywords of the language. The magic methods are called dunders (double-underscore, since they are all in the form `__methodname__()`)

Some easy examples are `__str__()` + `__repr__()` to have `.toString` (`str()` in py) + cool representation in REPL, `__eq__()` to implement the `==` operator, `__len__()` to implement the possibility of using `len()` on your class and so on.

#### Iterables

Many things in python are iterators so they can be iterated on. It's a central python concept and it's implemented in most primitives. you can create your iterators on any class by implementing the dunders `__iter__()` and `__next__()`.

#### Decorators

Python makes massive use of the decorator pattern which are encoded in the language with the special syntax `@decoratorname` before the thing it decorates.
Some common decorators are `@property, @classmethod, @staticmethod` and the special things defined by libraries, usually web frameworks like the classic flask `@get`.
You can write your custom decorators obviously.

#### Packages and modules

A python *module* is a python file containing code. A python *package* is a directory with 1+ python modules. Can have other packages inside.

Projects are usually structured with a `pyproject.toml` at the root level, then a directory with the name of the main package.

Imports are absolute in the package structure if you did everything correctly, so an `app/handlers` package, that has a `user.py` module, and inside the module a `add_user()` function, should be importable like this:

`from app.handlers.user import add_user`.

#### `__init.py__`

Every python package (a directory basically) **must** have an `__init.py__` file inside in order to be recognized as a package. This is a rule. Just do it. Quirks. Don't write anything inside it. It's usually used to fake visibility so people should just import stuff declared there in other packages, but no one does it anymore. Curse of popularity.

#### Naming/stylistic conventions

Everything should be in `lowercase_kebab_case`. Classes should be named in `PascalCase`.

#### Comments

Use `#` for comments. No special syntax for multiline comments and can comment inline.

### Strings

Literals declared with `""` or `''`. Can interpolate if prepended by an `f""` (called f-strings). Are basically bytes, UTF-8.

They are of type `str` which has a corresponding `str()` constructor.

Useful methods:

```py
"Gio" + "rgio"
# 'Giorgio'

"Gio" * 3
# 'GioGioGio'

"Giorgio".startswith("G")
# True

"Giorgio".endswith("i")
# False

"9".isdigit()
# True

", ".join(["Foo", "Bar", "Qux"])
# 'Foo, Bar, Qux'

"Foo, Bar, Qux".split(", ")
# ['Foo', 'Bar', 'Qux']

"""
This is a multiline
string for your
editing pleasure
"""
# '\nThis is a multiline\nstring for your\nediting pleasure\n'
# notice the starting and ending \n ;)
```

#### Bytes

Some libraries wants `bytes()` instead of strings. Bytes are string literals prepended by a `b`, i.e. `b"Giorgio"` and are usually a sign of python 2 legacy. Your best bet is to immediately `.decode("utf-8")` them to strings. `.encode()` on a string transform it to bytes.

```py
"Giorgio".encode()
# b'Giorgio'

b"Giorgio".decode()
# 'Giorgio'
```

### Numbers

Type `int()` for integers, arbitrarily big. I think the constructor also accepts strings.

```py
2349324234902349023 + 8923448923
# 2349324243825797946

# usual suspects here, most important division and integer division
30 / 2
# 15.0 - a float

30 // 2
# 15 - an int
```

Type `float()` for ... floats. If a number has a dot, it's a float literal. IEEE classic float implementation, bad precision. Use `Decimal` in the standard library if you want arbitrary precision. `float()` with "NaN" or "inf"/"-inf" values makes NaN and the signed infinites, useful if you need to accumulate a counter over a loop and there are signed numbers as the infinites are greather than/lesser than every other number.

```py
float("-inf")
# -inf
  
float("NaN")
# nan

0.2+0.1
# 0.30000000000000004
```

### Boolean

Not much to say, `True` and `False`. Use the `bool()` constructor to check if something is truthy or falsy:

```py
bool(34)
# True

bool(False)
# False

bool("")
# False
```

#### Truthiness and falsiness

I need to introduce some datatypes beforehand but bear with me: the following are falsy values in Python:

- The number zero (`0`)
- An empty string `''`
- `False`
- `None`
- An empty list `[]`
- An empty tuple `()`
- An empty dictionary `{}`

everything else is truthy. This is important because py relies a lot on truthy-falsy values for constructs like `if`.

### None

Just null, but it's written `None`.

### Operators

Usually enabled by implementing dunders.

- `=` assigns
- `+ - * / < > ==`  do what they say on the tin barring some strange cases (like string multiplication)
- `**` pow
- `in` checks if left hand is inside right hand collection
- `is` is identity equality (are they the same object, occupying the same memory space). `False, True, None` are singletons so usually those checks are done with `if foo is None` and not with normal equality `==` 
- `and or ^` boolean operators, i think XOR is `^` if left hand side and right hand side are both bools. I never used a xor in 8 years of python. They short circuit.

#### Less common operators

- `|` set/dict union, also used in type annotations to create something similar to a tagged union from functional languages
- `*name` the operator is `*`, "absorbs" stuff into a list that will be bound to `name`, used in destructuring and stuff like that
- `**dict` splats the dict, where possible

### Builtins

Before going with other primitives, be aware that python has a lot of builtins functions that can be invoked from everywhere (yay, PHP!). You can list them by doing

```py
import builtins
dir(builtins)
# ...omitted
```

The most useful are `isinstance()` to check type membership, `type()` to know a type of something (don't use it to compare, use isinstance for that), `len()` to see the length of things, `any()` and `all()` on iterables to know if at least one or all the values are truthy, `sum()` and `sorted()` are pretty intuitive (sorted returns a copy, it's not in place). I will be using some datatypes I'll explain later.

```py
isinstance("", str)
# True

type(9)
# int

len("ciaociao")
# 8

any(item > 1 for item in [-1,2,3])
# True

all(item > 1 for item in [-1,2,3])
# False

sum([1,2,3])
# 6

sorted(["cap2_1", "cap1", "cap2"])
# ['cap1', 'cap2', 'cap2_1']
```

### Conditionals

Your basic conditional keyword is `if..elif..else`. Really not that much to say other than it's idiomatic to rely on falsy values as conditions rather than explicit checks (i.e. a common idiom is `if not list` to check if a list is empty, not `if len(list) == 0`)

```py
stock_list = []
if not stock_list:
     print("you're poor")
# you're poor

debt_list = [100_000]  # you can use _ as a separator for integers
if debt_list:
     print("you're in debt")
# you're in debt

bank_account = 0
if bank_account > 100_000:
     print("ok")
 elif bank_account > 50_000:
     print("meh")
 else:
     print(":(")
# :(
```

#### Ternaries

Ternaries expressions have the form `result_true if condition else result_false`. They are very expressive but beware as coverage tools won't spot if you're not walking both branches.

```py
weather = "sunny"
return "Go for a walk" if weather == "sunny" else "Stay inside"
```

### Loops

Two basic constructs: `for` and `while`. `for` walks over iterables (no counter management in the loop declaration), `while` needs counter management.

```py
for number in range(0, 4):  # range produces an iterable that goes from first argument inclusive to last argument exclusive
     print(number)
# 0
# 1
# 2
# 3

while True:
     print("I would be an infinite loop but...")
     break  # break interrupts the loop
# I would be an infinite loop but...

counter = 0
while counter < 3:
    print(counter)
    counter += 1
# 0
# 1
# 2

for item in [1, 2, 3, 4, 5]:
    print(item ** 2)
# 1
# 4
# 9
# 16
# 25

for item in [1, 2, 3, 4, 5]:
    if item % 2 == 0:
        continue  # stop and go to the next iteration
    print(item ** 2)
# 1
# 9
# 25
```

Try to use comprehensions (we will see them later) instead of `for`s and `while`s.

### Lists

List literals are declared with `[]` and can contain heterogenous types. Some example of classical operations on lists:

```
In [ ]: names = ["Giorgio", "Egle", "Giovanna"]

In [ ]: names.append(3)

In [ ]: names
Out[ ]: ['Giorgio', 'Egle', 'Giovanna', 3]

In [ ]: names.pop()
Out[ ]: 3

In [ ]: names[0]  # index access
Out[ ]: 'Giorgio'

In [ ]: names[0:1]  # slice syntax, from:to. to is exclusive.
Out[ ]: ['Giorgio']

In [113]: names[1:]  # From 1 until the end
Out[113]: ['Egle', 'Giovanna']

In [115]: names[:2]  # From 0 until 1
Out[115]: ['Giorgio', 'Egle']

In [116]: names[::-1]  # No indexes, but specifies the step, so this actually reverses a list
Out[116]: ['Giovanna', 'Egle', 'Giorgio']

In [117]: names[-1]  # Can access from the back with negative indexes
Out[117]: 'Giovanna'

In [118]: for name in names:  # iterates over a list
     ...:     print(name)
     ...:
Giorgio
Egle
Giovanna
# Use `for index, item in enumerate(items):` if you need the index and the item

In [ ]: first_name, *rest = names  # List destructuring

In [ ]: print(f"{first_name=}\n{rest=}")
first_name='Giorgio'
rest=['Egle', 'Giovanna']

In [ ]: unsorted_nums = [7, 1, 33, 55, -8]

In [ ]: sorted(unsorted_nums)  # returns a copy
Out[ ]: [-8, 1, 7, 33, 55]

In [ ]: unsorted_nums.sort()  # in place, no return

In [ ]: unsorted_nums
Out[ ]: [-8, 1, 7, 33, 55]
```

#### List comprehensions

Very important. Python don't use map/filter/reduce and LINQ-like functional methods, we just use listcomps.

```
In [ ]: nums = [1, 2, 3, 4]

In [ ]: [n * 2 for n in nums]
Out[ ]: [2, 4, 6, 8]

In [ ]: [n * 2 for n in nums if n % 2 == 0]
Out[ ]: [4, 8]

In [ ]: ["FizzBuzz" if n % 15 == 0 else "Buzz" if n % 5 == 0 else "Fizz" if n % 3 == 0 else n for n in range(1, 31)]
# range(start, end_exclusive, step) generates a list of numbers
Out[ ]:
[1,
 2,
 'Fizz',
 4,
 'Buzz',
 'Fizz',
 7,
 8,
 'Fizz',
 'Buzz',
 11,
 'Fizz',
 13,
 14,
 'FizzBuzz',
 16,
 17,
 'Fizz',
 19,
 'Buzz',
 'Fizz',
 22,
 23,
 'Fizz',
 'Buzz',
 26,
 'Fizz',
 28,
 29,
 'FizzBuzz']
```

You can use `()` instead of `[]` so they become generators, basically lazy lists. It is possible to chain `for` expressions but it's super unreadable, don't do that. The comprehension syntax is used for other things so internalize it well.

#### Tuples

Basically immutable lists. Use the `()` literal syntax, or the `tuple()` constructor. Not really used that much in modern python.

### Sets

Use the `set()` constructor or the `{}` literal syntax. Unordered. 

```
In [ ]: names_set = {"Giorgio", "Giorgio", "Egle", "Giovanna"}

In [ ]: names_set
Out[ ]: {'Egle', 'Giorgio', 'Giovanna'}

In [ ]: names_set.add("Marco")

In [ ]: names_set
Out[ ]: {'Egle', 'Giorgio', 'Giovanna', 'Marco'}

In [ ]: names_set.isdisjoint({"Giuseppe"})
Out[ ]: True

In [ ]: names_set.issuperset({"Egle"})
Out[ ]: True

In [ ]: names_set.issubset({"Egle"})
Out[ ]: False

In [ ]: {"Egle"} | {"Giorgio"}  # union
Out[ ]: {'Egle', 'Giorgio'}

In [ ]: {"Egle"} == {"Egle"}  # equality
Out[ ]: True

In [ ]: {num for num in [1, 1, 1, 2, 2, 3]}  # set comprehension
Out[ ]: {1, 2, 3}
```

### Dicts

Use the `dict()` constructor or the `{k: v}` literal syntax. Insertion ordered.
```
In [ ]: ages = {"Giorgio": 39, "Egle": 2, "Giovanna": 41}

In [ ]: [key for key in ages.keys()]
Out[ ]: ['Giorgio', 'Egle', 'Giovanna']

In [ ]: [value for value in ages.values()]
Out[ ]: [39, 2, 41]

In [ ]: [mmm for mmm in ages]  # common error: iteration over a dict yield keys, not k:v pairs
Out[ ]: ['Giorgio', 'Egle', 'Giovanna']

In [ ]: [f"{name} has {age} years" for name, age in ages.items()]  # .items() return tuples, which gets destructured into the variables of the loop
Out[ ]: ['Giorgio has 39 years', 'Egle has 2 years', 'Giovanna has 41 years']

In [ ]: ages["Giorgio"]
Out[ ]: 39

In [ ]: ages.get("Giorgio")
Out[ ]: 39

In [ ]: ages["Carlo"]
KeyError: 'Carlo'

In [ ]: print(ages.get("Carlo"))  # print to see the None
None

In [ ]: ages.get("Carlo", 33)
Out[ ]: 33

In [ ]: ages["Carlo"] = 33

In [ ]: ages
Out[ ]: {'Giorgio': 39, 'Egle': 2, 'Giovanna': 41, 'Carlo': 33}

In [ ]: ages.update({"Marco": 22})  # update in place

In [ ]: ages
Out[ ]: {'Giorgio': 39, 'Egle': 2, 'Giovanna': 41, 'Carlo': 33, 'Marco': 22}

In [ ]: {"Roberto": 70} | ages  # update and return copy
Out[ ]:
{'Roberto': 70,
 'Giorgio': 39,
 'Egle': 2,
 'Giovanna': 41,
 'Carlo': 33,
 'Marco': 22}

In [ ]: {"Roberto": 70, **ages}  # yet another syntax
Out[ ]:
{'Roberto': 70,
 'Giorgio': 39,
 'Egle': 2,
 'Giovanna': 41,
 'Carlo': 33,
 'Marco': 22}
 
In [ ]: {name: age for name, age in zip(["Giorgio", "Egle"], [39, 2])}  # dict comprehension. also zip(), that mixes lists into tuples. works with other iterable also.
Out[ ]: {'Giorgio': 39, 'Egle': 2}
```
### Functions

Functions are declared with the `def` keyword:

```
def greet(name):
    return f"Hello, {name}"

In [ ]: greet("Giorgio")
Out[ ]: 'Hello, Giorgio'
```

Arguments are usually positional, but can also be called via keyword, out of order:

```
def greet_with_age(name, age):
    return f"Hello, {name}, you are {age} years old"

In [ ]: greet_with_age(age=22, name="Giorgio")
Out[ ]: 'Hello, Giorgio, you are 22 years old'
```
  
Arguments can have defaults:

```
def better_greet(name="person"):
    return f"Hello, {name}"

In [ ]: better_greet()
Out[ ]: 'Hello, person'
```

You can have variadic functions by adding `*args` to the signature, and those arguments will be packed in a list. `*` is a special syntax that says "absorb every positional arguments after the last positional argument in the signature"

```
def multi_greeter(*args):
     return f"Hello, {', '.join(args)}"

In [ ]: multi_greeter("Giorgio")
Out[ ]: 'Hello, Giorgio'

In [ ]: multi_greeter("Giorgio", "Egle", "Giovanna")
Out[ ]: 'Hello, Giorgio, Egle, Giovanna'
```

A best practice, not really super common as of now, is to have keyword only arguments in functions. Imagine the `*` absorbing every positional argument and binding them to nothing:

```
def kw_greeter(*, name, age):
     return f"Hello, {name}, you are {age} years old"

In [ ]: kw_greeter("Giorgio", 22)
...omitted stack trace
TypeError: kw_greeter() takes 0 positional arguments but 2 were given

In [ ]: kw_greeter(name="Giorgio", age=22)
Out[ ]: 'Hello, Giorgio, you are 22 years old'
```

You can also pass variadic keyword arguments with the `**kwargs` special syntax at the end of a function. The variadic arguments will be unpacked inside a dictionary. This is a super useful technique for libraries, specially ORMs (see django orm):

```
def kwargs_greeter(**kwargs):
     print(kwargs)
     name = kwargs["name"]
     age = kwargs["age"]
     return f"Hello, {name}, you are {age} years old"


In [ ]: kwargs_greeter(name="Giorgio", age=22, fav_food="Pasta", profile_pic=None)

{'name': 'Giorgio', 'age': 22, 'fav_food': 'Pasta', 'profile_pic': None}

Out[ ]: 'Hello, Giorgio, you are 22 years old'
```

#### Anonymous functions

Don't use them if not in very very specific situations, like passing a sorting function to `.sort()` or to the `map()` builtin. Lambdas in py are restricted to single statement and the functional style is, in general, frowned upon.

```
sum2 = lambda a, b: a + b

In [ ]: sum2(3,4)
Out[ ]: 7

In [ ]: sorted(["11111", "22", "3"], key=lambda x: len(x))
Out[ ]: ['3', '22', '11111']
```

#### Footguns

Never never never use mutable things (lists, dicts, etc.) as default fn arguments  or nasty things will happen:

```
def bugged(a=[]):
     a.append(1)
     return a

In [ ]: bugged()
Out[ ]: [1]

In [ ]: bugged()
Out[ ]: [1, 1]

In [ ]: bugged()
Out[ ]: [1, 1, 1]

In [ ]: bugged()
Out[ ]: [1, 1, 1, 1]
```

### Classes
Classes are defined by the `class` keyword. Python has multiple inheritance: don't use it unless you have a good reason. It's strange and involves things with bad names like `MRO - Module Resolution Order`.

```
class Person:
    originating_planet = "earth"  # this is a class variable...they're not used that much, so don't mind them, just don't be surprised when you see some frameworks (django specially) inheriting and setting those outside the constructor. They exist. Mainly used to avoid having big constructors with default parameters.
    
    def __init__(self, name, age):
        # __init__ is the special name for the constructor.
        # All methods must have self as the first parameter,
        # and it will be automatically passed when used.
        self.name = name
        self.age = age

    def __str__(self):
        # basically the .toString()
        return f"<Person {self.name=}>"

    def __repr__(self):
        # cool REPL repr-esentation
        return self.__str__()

    def is_adult(self):
        return self.age >= 18

    @classmethod  # takes cls as first arg, basically "things that in KT go in the companion object"
    def from_string(cls, person_string):
        """
        Constructs a person from a "name, age" string.
        
        By the way, this special syntax of having a
        multiline string after a definition is called
        the docstring, and it's the thing that gets
        pulled out when help()-ing something or from the
        ide.
        """
        name, age = person_string.split(",")
        return cls(name, int(age))


person = Person("Giorgio", 39)  # no need for new

# getters and setters are not in python style.
In [ ]: person.name
Out[ ]: 'Giorgio'

In [ ]: person.name = "Girgio"

In [ ]: person.name
Out[ ]: 'Girgio'

In [ ]: person
Out[ ]: <Person self.name='Giorgio'>

In [ ]: person_2 = Person.from_string("Marco, 33")

In [ ]: person_2
Out[ ]: <Person self.name='Marco'>

In [ ]: person_2.is_adult()
Out[ ]: True
```

### Error handling

Python is exception based, not checked, and the basic construct to handle errors is by wrapping potential excepting functions with `try...except...finally`. All python exceptions inherit from `Exception` and there is a wealth of builtin exceptions. `except` can catch specific exceptions and it's good practice to do so rather than be naked.

```
ages = {"Giorgio": 39, "Egle": 2, "Giovanna": 41}

def fetch_age(name):  
    try:  
        return ages[name]  
    except KeyError:  # Key error is emitted when trying to access a non-present key  
        return "Unknown name"  
    finally:  
        print("I will be executed anyway")

In [ ]: fetch_age("Marco")
I will be executed anyway
Out[ ]: 'Unknown name'
```

You can throw via the `raise` keyword, and can also create custom exceptions:

```
class CustomException(Exception):
    def __init__(self, msg):
        self.msg = msg

def raiser():
    raise CustomException("I'm broken!")

try:
    raiser()
except CustomException as e:
    print(e)

I'm broken!
```

Handled exception can be put in scope with the `as` keyword as shown.


### "advanced" stuff

#### Context managers

#### dunders

#### ABCs, protocols, multiple inheritance

#### debugging

  

## Ecosystem

### Standard library
### Third party libraries
### Tools
