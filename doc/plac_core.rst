Plac: Parsing the Command Line the Easy Way
===========================================

:Author: Michele Simionato
:E-mail: michele.simionato@gmail.com
:Date: July 2019
:Download page: http://pypi.python.org/pypi/plac
:Project page: https://github.com/micheles/plac
:Requires: Python from 2.6 to 3.7
:Installation: ``pip install plac``
:License: BSD license

.. contents::

The importance of scaling down
------------------------------

There is no want of command-line arguments parsers in the Python
world. The standard library alone contains three different modules:
getopt_ (from the stone age),
optparse_ (from Python 2.3) and argparse_ (from Python 2.7).  All of
them are quite powerful and especially argparse_ is an industrial
strength solution; unfortunately, all of them feature a non-negligible learning
curve and a certain verbosity. They do not scale down well enough, at
least in my opinion.

It should not be necessary to stress the importance of `scaling down`_;
nevertheless, a lot of people are obsessed with features and concerned with
the possibility of scaling up, forgetting the equally important
issue of scaling down. This is an old meme in
the computing world: programs should address the common cases simply and
simple things should be kept simple, while at the same time keeping
difficult things possible. plac_ adhere as much as possible to this
philosophy and it is designed to handle well the simple cases, while
retaining the ability to handle complex cases by relying on the
underlying power of argparse_.

Technically plac_ is just a simple wrapper over argparse_ which hides
most of its complexity by using a declarative interface: the argument
parser is inferred rather than written down by imperatively.  Still, plac_ is
surprisingly scalable upwards, even without using the underlying
argparse_. I have been using Python for 9 years and in my experience
it is extremely unlikely that you will ever need to go beyond the
features provided by the declarative interface of plac_: they should
be more than enough for 99.9% of the use cases.

plac_ is targetting especially unsophisticated users,
programmers, sys-admins, scientists and in general people writing
throw-away scripts for themselves, choosing the command-line
interface because it is quick and simple. Such users are not
interested in features, they are interested in a small learning curve:
they just want to be able to write a simple command line tool from a
simple specification, not to build a command-line parser by
hand. Unfortunately, the modules in the standard library forces them
to go the hard way. They are designed to implement power user tools
and they have a non-trivial learning curve. On the contrary, plac_
is designed to be simple to use and extremely concise, as the examples
below will show.

Scripts with required arguments
-------------------------------

Let me start with the simplest possible thing: a script that takes a
single argument and does something to it.  It cannot get simpler
than that, unless you consider the case of a script without command-line
arguments, where there is nothing to parse. Still, it is a use
case *extremely common*: I need to write scripts like that nearly
every day, I wrote hundreds of them in the last few years and I have
never been happy. Here is a typical example of code I have been
writing by hand for years:

.. include:: example1.py
   :literal:

As you see the whole ``if __name__ == '__main__'`` block (nine lines)
is essentially boilerplate that should not exist.  Actually I think
the language should recognize the main function and pass the
command-line arguments automatically; unfortunaly this is unlikely to
happen. I have been writing boilerplate like this in hundreds of
scripts for years, and every time I *hate* it. The purpose of using a
scripting language is convenience and trivial things should be
trivial. Unfortunately the standard library does not help for this
incredibly common use case. Using getopt_ and optparse_ does not help,
since they are intended to manage options and not positional
arguments; the argparse_ module helps a bit and it is able to reduce
the boilerplate from nine lines to six lines:

.. include:: example2.py
   :literal:

However, it just feels too complex to instantiate a class and to
define a parser by hand for such a trivial task.

The plac_ module is designed to manage well such use cases, and it is able
to reduce the original nine lines of boiler plate to two lines. With the
plac_ module all you need to write is

.. include:: example3.py
   :literal:

The plac_ module provides for free (actually the work is done by the
underlying argparse_ module) a nice usage message::

 $ python example3.py -h

.. include:: example3.help
   :literal:

Moreover plac_ manages the case of missing arguments and of too many arguments.
This is only the tip of the iceberg: plac_ is able to do much more than that.

Scripts with default arguments
------------------------------

The need to have suitable defaults for command-line scripts is quite
common. For instance I have encountered this use case at work hundreds
of times:

.. include:: example4.py
   :literal:

Here I want to perform a query on a database table, by extracting the
most recent data: it makes sense for ``today`` to be a default argument.
If there is a most used table (in this example a table called ``'product'``)
it also makes sense to make it a default argument. Performing the parsing
of the command-line arguments by hand takes 8 ugly lines of boilerplate
(using argparse_ would require about the same number of lines).
With plac_ the entire ``__main__`` block reduces to the usual two lines::

  if __name__ == '__main__':
      import plac; plac.call(main)

In other words, six lines of boilerplate have been removed, and we get
the usage message for free:

.. include:: example5.help
   :literal:

Notice that by default plac_ prints the string representation
of the default values (with square brackets) in the usage message.
plac_ manages transparently even the case when you want to pass a
variable number of arguments. Here is an example, a script running
on a database a series of SQL scripts:

.. include:: example7.py
   :literal:

Here is the usage message:

.. include:: example7.help
   :literal:

The examples here should have made clear that *plac is able to figure out
the command-line arguments parser to use from the signature of the main
function*. This is the whole idea behind plac_: if the intent is clear,
let's the machine take care of the details.

plac_ is inspired to an old Python Cookbook recipe of mine (optionparse_), in
the sense that it delivers the programmer from the burden of writing
the parser, but is less of a hack: instead of extracting the parser
from the docstring of the module, it extracts it from the signature of
the ``main`` function.

The idea comes from the `function annotations` concept, a new
feature of Python 3. An example is worth a thousand words, so here
it is:

.. include:: example7_.py
   :literal:

Here the arguments of the ``main`` function have been annotated with
strings which are intented to be used in the help message:

.. include:: example7_.help
   :literal:

plac_ is able to recognize much more complex annotations, as
I will show in the next paragraphs.


Scripts with options (and smart options)
----------------------------------------

It is surprising how few command-line scripts with options I have
written over the years (probably less than a hundred), compared to the
number of scripts with positional arguments I wrote (certainly more
than a thousand of them).  Still, this use case cannot be neglected.
The standard library modules (all of them) are quite verbose when it
comes to specifying the options and frankly I have never used them
directly. Instead, I have always relied on the
optionparse_ recipe, which provides a convenient wrapper over
argparse_. Alternatively, in the simplest cases, I have just
performed the parsing by hand. In plac_ the parser is inferred by the
function annotations. Here is an example:

.. include:: example8.py
   :literal:

Here the argument ``command`` has been annotated with the tuple
``("SQL query", 'option', 'c')``: the first string is the help string
which will appear in the usage message, the second string tells plac_
that ``command`` is an option and the third string that there is also
a short form of the option ``-c``, the long form being ``--command``.
The usage message is the following:

.. include:: example8.help
   :literal:

Here are two examples of usage::

 $ python3 example8.py -c "select * from table" dsn
 executing select * from table on dsn

 $ python3 example8.py --command="select * from table" dsn
 executing select * from table on dsn

The third argument in the function annotation can be omitted: in such
case it will be assumed to be ``None``. The consequence is that
the usual dichotomy between long and short options (GNU-style options)
disappears: we get *smart options*, which have the single character prefix
of short options and behave like both long and short options, since
they can be abbreviated. Here is an example featuring smart options:

.. include:: example6.py
   :literal:

.. include:: example6.help
   :literal:
 
The following are all valid invocations ot the script::

  $ python3 example6.py -c "select" dsn
  executing 'select' on dsn
  $ python3 example6.py -com "select" dsn
  executing 'select' on dsn
  $ python3 example6.py -command="select" dsn
  executing 'select' on dsn

Notice that the form ``-command=SQL`` is recognized only for the full
option, not for its abbreviations::

  $ python3 example6.py -com="select" dsn
  usage: example6.py [-h] [-command COMMAND] dsn
  example6.py: error: unrecognized arguments: -com=select

If the option is not passed, the variable ``command``
will get the value ``None``. However, it is possible to specify a non-trivial
default. Here is an example:

.. include:: example8_.py
   :literal:

Notice that the default value appears in the help message:

.. include:: example8_.help
   :literal:

When you run the script and you do not pass the ``-command`` option, the
default query will be executed::

 $ python3 example8_.py dsn
 executing 'select * from table' on dsn

Scripts with flags
------------------

plac_ is able to recognize flags, i.e. boolean options which are
``True`` if they are passed to the command line and ``False`` 
if they are absent. Here is an example:

.. include:: example9.py
   :literal:

.. include:: example9.help
   :literal:

::

 $ python3 example9.py -v dsn
 connecting to dsn

Notice that it is an error trying to specify a default for flags: the
default value for a flag is always ``False``. If you feel the need to
implement non-boolean flags, you should use an option with two
choices, as explained in the "more features" section.

For consistency with the way the usage message is printed, I suggest
you to follow the Flag-Option-Required-Default (FORD) convention: in
the ``main`` function write first the flag arguments, then the option
arguments, then the required arguments and finally the default
arguments. This is just a convention and you are not forced to use it,
except for the default arguments (including the varargs) which must
stay at the end as it is required by the Python syntax.

I also suggests to specify a one-character abbreviation for flags: in
this way you can use the GNU-style composition of flags (i.e. ``-zxvf``
is an abbreviation of ``-z -x -v -f``). I usually do not provide
the one-character abbreviation for options, since it does not make sense
to compose them.

Starting from plac_ 0.9.1 underscores in options and flags are
automatically turned into dashes. This feature was implemented at user
request, to make it possible to use a more traditional naming. For
instance now you can have a ``--dry-run`` flag, whereas before you had
to use ``--dry_run``.

.. include:: dry_run.py
   :literal:

Here is an example of usage::

 $ python3.2 dry_run.py -h
 usage: dry_run.py [-h] [-d]
 
 optional arguments:
   -h, --help     show this help message and exit
   -d, --dry-run  Dry run
 
plac for Python 2.X users
-------------------------

While plac runs great on Python 3, I do not personally use it.
At work we migrated to Python 2.7 in 2011. It will take a few more
years before we consider migrating to Python 3. I am pretty much
sure many Pythonistas are in the same situation. Therefore plac_
provides a way to work with function annotations even in Python 2.X
(including Python 2.3).  There is no magic involved; you just need to
add the annotations by hand. For instance the annotated function
declaration

::

  def main(dsn: "Database dsn", *scripts: "SQL scripts"):
      ...

is equivalent to the following code::

  def main(dsn, *scripts):
      ...
  main.__annotations__ = dict(
      dsn="Database dsn",
      scripts="SQL scripts")

One should be careful to match the keys of the annotation dictionary
with the names of the arguments in the annotated function; for lazy
people with Python 2.4 available the simplest way is to use the
``plac.annotations`` decorator that performs the check for you::

  @plac.annotations(
      dsn="Database dsn",
      scripts="SQL scripts")
  def main(dsn, *scripts):
      ...

In the rest of this article I will assume that you are using Python 2.X with
X >= 4 and I will use the ``plac.annotations`` decorator. Notice however
that the core features of plac_ run even on Python 2.3.

More features
-------------

One of the goals of plac_ is to have a learning curve of *minutes* for
its core features, compared to the learning curve of *hours* of
argparse_. In order to reach this goal, I have *not* sacrificed all
the features of argparse_. Actually a lot of the argparse_ power persists
in plac_.  Until now, I have only showed simple annotations, but in
general an annotation is a 6-tuple of the form

  ``(help, kind, abbrev, type, choices, metavar)``

where ``help`` is the help message, ``kind`` is a string in the set {
``"flag"``, ``"option"``, ``"positional"``}, ``abbrev`` is a
one-character string or ``None``, ``type`` is a callable taking a 
string in input,
``choices`` is a discrete sequence of values and ``metavar`` is a string.

``type`` is used to automagically convert the command line arguments
from the string type to any Python type; by default there is no
conversion and ``type=None``.

``choices`` is used to restrict the number of the valid
options; by default there is no restriction i.e. ``choices=None``.

``metavar`` has two meanings. For a positional argument it is used to
change the argument name in the usage message (and only there). By
default the metavar is ``None`` and the name in the usage message is
the same as the argument name.  For an option
the ``metavar`` is used differently in the usage message, which has
now the form ``[--option-name METAVAR]``. If the ``metavar`` is ``None``,
then it is equal to the uppercased name of the argument, unless the
argument has a default: then it is equal to the stringified
form of the default.

Here is an example showing many of the features (copied from the
argparse_ documentation):

.. include:: example10.py
   :literal:

Here is the usage:

.. include:: example10.help
   :literal:

Notice that the docstring of the ``main`` function has been automatically added
to the usage message. Here are a couple of examples of usage::

 $ python example10.py add 1 2 3 4
 10.0
 $ python example10.py mul 1 2 3 4
 24.0
 $ python example10.py ad 1 2 3 4 # a mispelling error
 usage: example10.py [-h] {add,mul} [n [n ...]]
 example10.py: error: argument operator: invalid choice: 'ad' (choose from 'add', 'mul')

``plac.call`` can also be used in doctests like this:

>>> import plac, example10
>>> plac.call(example10.main, ['add', '1', '2'])
3.0

``plac.call`` works for generators too:

>>> def main(n):
...     for i in range(int(n)):
...         yield i
>>> plac.call(main, ['3'])
[0, 1, 2]

Internally ``plac.call`` tries to convert the output of the main function
into a list, if possible. If the output is not iterable or it is a
string, it is left unchanged, but if it is iterable it is converted. 
In particular, generator objects are exhausted by ``plac.call``.

This behavior avoids mistakes like forgetting of applying
``list(result)`` to the result of ``plac.call``; moreover it makes
errors visible early, and avoids mistakes in code like the following::

 try:
     result = plac.call(main, args)
 except:
    # do something

Without eagerness, a main function returning a generator object would
not raise any exception until the generator is iterated over.
If you are a fan of lazyness, you can still have it by setting the ``eager``
flag to ``False``, as in the following example::

   for line in plac.call(main, args, eager=False):
       print(line)

If ``main`` returns a generator object this example will print each
line as soon as available, whereas the default behaviour is to print
all the lines together and the end of the computation.

A realistic example
-------------------

Here is a more realistic script using most of the features of plac_ to
run SQL queries on a database by relying on SQLAlchemy_. Notice the usage
of the ``type`` feature to automagically convert a SQLAlchemy connection
string into a SqlSoup_ object:

.. include:: dbcli.py
   :literal:

You can see the *yield-is-print* pattern here: instead of using
``print`` in the main function, I use ``yield``, and I perform the
print in the ``__main__`` block. The advantage of the pattern is that
tests invoking ``plac.call`` and checking the result become trivial:
had I performed the printing in the main function, the test would have
involved an ugly hack like redirecting ``sys.stdout`` to a
``StringIO`` object.

Here is the usage message:

.. include:: dbcli.help
   :literal:

You can check for yourself that the script works.

Keyword arguments
-----------------

Starting from release 0.4, plac_ supports keyword arguments.  In
practice that means that if your main function has keyword arguments,
plac_ treats specially arguments of the form ``"name=value"`` in the
command line.  Here is an example:

.. include:: example12.py
   :literal:

Here is the generated usage message:

.. include:: example12.help
   :literal:

Here is how you call the script::

  $ python example12.py -o X a1 a2 name=value
  opt=X
  args=('a1', 'a2')
  kw={'name': 'value'}

When using keyword arguments, one must be careful to use names which
are not alreay taken; for instance in this examples the name ``opt``
is taken::
 
 $ python example12.py 1 2 kw1=1 kw2=2 opt=0
 usage: example12.py [-h] [-o OPT] [args [args ...]] [kw [kw ...]]
 example12.py: error: colliding keyword arguments: opt

The names taken are the names of the flags, of the options, and of the
positional arguments, excepted varargs and keywords. This limitation
is a consequence of the way the argument names are managed in function calls
by the Python language.

plac vs argparse
----------------

plac_ is opinionated and by design it does not try to make available
all of the features of argparse_ in an easy way.  In particular you
should be aware of the following limitations/differences (the
following assumes knowledge of argparse_):

- plac does not support the destination concept: the destination
  coincides with the name of the argument, always. This restriction
  has some drawbacks. For instance, suppose you want to define a long
  option called ``--yield``. In this case the destination would be ``yield``,
  which is a Python keyword, and since you cannot introduce an
  argument with that name in a function definition, it is impossible
  to implement it. Your choices are to change the name of the long
  option, or to use argparse_ with a suitable destination.

- plac_ does not support "required options". As the argparse_
  documentation puts it: *Required options are generally considered bad
  form - normal users expect options to be optional. You should avoid
  the use of required options whenever possible.* Notice that since
  argparse_ supports them, plac_ can manage them too, but not directly.

- plac_ supports only regular boolean flags. argparse_ has the ability to 
  define generalized two-value flags with values different from ``True`` 
  and ``False``. An earlier version of plac_ had this feature too, but 
  since you can use options with two choices instead, and in any case
  the conversion from ``{True, False}`` to any couple of values
  can be trivially implemented with a ternary operator
  (``value1 if flag else value2``), I have removed it (KISS rules!).

- plac_ does not support ``nargs`` options directly (it uses them internally,
  though, to implement flag recognition). The reason it that all the use
  cases of interest to me are covered by plac_ and I did not feel the need
  to increase the learning curve by adding direct support for ``nargs``.

- plac_ does support subparsers, but you must read the `advanced usage
  document`_ to see how it works.

- plac_ does not support actions directly. This also
  looks like a feature too advanced for the goals of plac_. Notice however
  that the ability to define your own annotation objects (again, see
  the  `advanced usage document`_) may mitigate the need for custom actions.

On the plus side, plac_ can leverage directly on a number of argparse_ features.

For instance, you can use argparse.FileType_ directly. Moreover,
it is possible to pass options to the underlying
``argparse.ArgumentParser`` object (currently it accepts the default
arguments ``description``, ``epilog``, ``prog``, ``usage``,
``add_help``, ``argument_default``, ``parents``, ``prefix_chars``,
``fromfile_prefix_chars``, ``conflict_handler``, ``formatter_class``).
It is enough to set such attributes on the ``main`` function.  For
instance writing

::

  def main(...):
      pass

  main.add_help = False

disables the recognition of the help flag ``-h, --help``. This
mechanism does not look particularly elegant, but it works well
enough.  I assume that the typical user of plac_ will be happy with
the defaults and would not want to change them; still it is possible
if she wants to. 

For instance, by setting the ``description`` attribute, it is possible
to add a comment to the usage message (by default the docstring of the
``main`` function is used as description).

It is also possible to change the option prefix; for
instance if your script must run under Windows and you want to use "/"
as option prefix you can add the line::

  main.prefix_chars='/-'

The first prefix char (``/``) is used
as the default for the recognition of options and flags;
the second prefix char (``-``) is kept to keep the ``-h/--help`` option
working: however you can disable it and reimplement it, if you like.

It is possible to access directly the underlying ArgumentParser_ object, by
invoking the ``plac.parser_from`` utility function:

>>> import plac
>>> def main(arg):
...     pass
... 
>>> print(plac.parser_from(main)) #doctest: +ELLIPSIS
ArgumentParser(prog=...)

Internally ``plac.call`` uses ``plac.parser_from``. Notice that when
``plac.call(func)`` is invoked multiple time, the parser is re-used
and not rebuilt from scratch again.

I use ``plac.parser_from`` in the unit tests of the module, but regular
users should not need to use it, unless they want to access *all*
of the features of argparse_ directly without calling the main function.

Interested readers should read the documentation of argparse_ to
understand the meaning of the other options. If there is a set of
options that you use very often, you may consider writing a decorator
adding such options to the ``main`` function for you. For simplicity,
plac_ does not perform any magic.

Final example: a shelve interface
---------------------------------

Here is a nontrivial example showing off many plac_ feature, including
keyword arguments recognition.
The use case is the following: suppose we have stored the
configuration parameters of a given application into a Python shelve
and we need a command-line tool to edit the shelve.
A possible implementation using plac_ could be the following:

.. include:: ishelve.py
   :literal:

A few notes are in order:

1. I have disabled the ordinary help provided by argparse_ and I have 
   implemented a custom help command.
2. I have changed the prefix character used to recognize the options
   to a dot.
3. Keyword arguments recognition (in the ``**setters``) is used to make it
   possible to store a value in the shelve with the syntax 
   ``param_name=param_value``.
4. ``*params`` are used to retrieve parameters from the shelve and some
   error checking is performed in the case of missing parameters
5. A command to clear the shelve is implemented as a flag (``.clear``).
6. A command to delete a given parameter is implemented as an option
   (``.delete``).
7. There is an option with default (``.filename=conf.shelve``) to set
   the filename of the shelve.
8. All things considered, the code looks like a poor man's object oriented
   interface implemented with a chain of elifs instead of methods. Of course,
   plac_ can do better than that, but let me start from a low-level approach
   first.

If you run ``ishelve.py`` without arguments you get the following
message::

 $ python ishelve.py
 no arguments passed, use .help to see the available commands

If you run ``ishelve.py`` with the option ``.h`` (or any abbreviation
of ``.help``) you get::

  $ python ishelve.py .h
  Commands: .help, .showall, .clear, .delete
  <param> ...
  <param=value> ...

You can check by hand that the tool works::

 $ python ishelve.py .clear # start from an empty shelve
 cleared the shelve
 $ python ishelve.py a=1 b=2
 setting a=1
 setting b=2
 $ python ishelve.py .showall
 b=2
 a=1
 $ python ishelve.py .del b # abbreviation for .delete
 deleted b
 $ python ishelve.py a
 1
 $ python ishelve.py b
 b: not found
 $ python ishelve.py .cler # mispelled command
 usage: ishelve.py [.help] [.showall] [.clear] [.delete DELETE]
                   [.filename /home/micheles/conf.shelve]
                   [params [params ...]] [setters [setters ...]]
 ishelve.py: error: unrecognized arguments: .cler

plac vs the rest of the world
-----------------------------

Originally plac_ boasted about being "the easiest command-line
arguments parser in the world". Since then, people started pointing
out to me various projects which are based on the same idea
(extracting the parser from the main function signature) and are
arguably even easier than plac_:

- opterator_ by Dusty Phillips
- CLIArgs_ by Pavel Panchekha
- commandline_ by David Laban

Luckily for me none of such projects had the idea of using 
function annotations and argparse_; as a consequence, they are
no match for the capabilities of plac_.

Of course, there are tons of other libraries to parse the command
line. For instance Clap_ by Matthew Frazier which appeared on PyPI
just the day before plac_; Clap_ is fine but it is certainly not
easier than plac_.

plac_ can also be used as a replacement of the cmd_ module in the standard
library and as such it shares many features with the module cmd2_ by
Catherine Devlin. However, this is completely coincidental, since I became
aware of the cmd2_ module only after writing plac_.

Command-line argument parsers keep coming out; between the newcomers I
will notice `marrow.script`_ by Alice Bevan-McGregor, which is quite
similar to plac_ in spirit, but does not rely on argparse_ at all.
Argh_ by Andrey Mikhaylenko is also worth mentioning: it is based
on argparse_, it came after plac_ and I must give credit to the author
for the choice of the name, much funnier than plac!

The future
----------

Currently the core of plac_ is around 200 lines of code, not counting blanks,
comments and docstrings. I do not plan to extend the core much in the
future. The idea is to keep the module short: it is and it should
remain a little wrapper over argparse_. Actually I have thought about
contributing the core back to argparse_ if plac_ becomes successful
and gains a reasonable number of users. For the moment it should be
considered in a frozen status.

Notice that even if plac_ has been designed to be simple to use for
simple stuff, its power should not be underestimated; it is actually a
quite advanced tool with a domain of applicability which far exceeds
the realm of command-line arguments parsers.

Version 0.5 of plac_ doubled the code base and the documentation: it is
based on the idea of using plac_ to implement command-line interpreters,
i.e. something akin to the ``cmd`` module in the standard library, only better.
The new features of plac_ are described in the `advanced usage document`_ .
They are implemented in a separated module (``plac_ext.py``), since
they require Python 2.5 to work, whereas ``plac_core.py`` only requires
Python 2.3.

Trivia: the story behind the name
---------------------------------

The plac_ project started very humbly: I just wanted to make 
my old optionparse_ recipe easy_installable, and to publish it on PyPI.
The original name of plac_ was optionparser and the idea behind it was
to build an OptionParser_ object from the docstring of the module.
However, before doing that, I decided to check out the argparse_ module,
since I knew it was going into Python 2.7 and Python 2.7 was coming out.
Soon enough I realized two things:

1. the single greatest idea of argparse_ was unifying the positional arguments
   and the options in a single namespace object;
2. parsing the docstring was so old-fashioned, considering the existence
   of functions annotations in Python 3.

Putting together these two observations with the original idea of inferring the
parser I decided to build an ArgumentParser_ object from function
annotations. The ``optionparser`` name was ruled out, since I was
now using argparse_; a name like ``argparse_plus`` was also ruled out,
since the typical usage was completely different from the argparse_ usage.

I made a research on PyPI and the name *clap* (Command Line Arguments Parser)
was not taken, so I renamed everything to clap. After two days 
a Clap_ module appeared on PyPI <expletives deleted>!

Having little imagination, I decided to rename everything again to plac,
an anagram of clap: since it is a non-existing English name, I hope nobody
will steal it from me!

That concludes the section about the basic usage of plac_. You are now ready to
read about the advanced usage.

.. _argparse: http://argparse.googlecode.com
.. _optparse: http://docs.python.org/library/optparse.html
.. _getopt: http://docs.python.org/library/getopt.html
.. _optionparse: http://code.activestate.com/recipes/278844-parsing-the-command-line/
.. _plac: http://pypi.python.org/pypi/plac
.. _scaling down: http://www.welton.it/articles/scalable_systems
.. _ArgumentParser: http://argparse.googlecode.com/svn/tags/r11/doc/ArgumentParser.html
.. _argparse.FileType: http://argparse.googlecode.com/svn/tags/r11/doc/other-utilities.html?highlight=filetype#FileType
.. _Clap: http://pypi.python.org/pypi/Clap
.. _OptionParser: http://docs.python.org/library/optparse.html?highlight=optionparser#optparse.OptionParser
.. _SQLAlchemy: http://www.sqlalchemy.org/
.. _SqlSoup: http://www.sqlalchemy.org/docs/reference/ext/sqlsoup.html
.. _CLIArgs: http://pypi.python.org/pypi/CLIArgs
.. _opterator: http://pypi.python.org/pypi/opterator
.. _advanced usage document: in-writing
.. _cmd2: http://packages.python.org/cmd2/
.. _cmd: http://docs.python.org/library/cmd.html
.. _marrow.script: https://github.com/pulp/marrow.script 
.. _commandline: http://pypi.python.org/pypi/commandline
.. _argh: http://packages.python.org/argh
