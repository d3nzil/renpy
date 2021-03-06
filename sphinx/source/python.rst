.. _python:

Python Statements
=================

Ren'Py is written in the Python programming language, and includes
support for including python code inside Ren'Py scripts. Python
support can be used for many things, from setting a flag to creating
new displayables. This chapter covers ways in which Ren'Py scripts can
directly invoke Python code, through the various python statements.


.. _python-statement:

Python
------

The python statement takes a block of python code, and runs that code
when control reaches the statement. A basic python statement can be
very simple::

    python:
        flag = True

Python statements can get more complex, when necessary::

    python:
        player_health = max(player_health - damage, 0)
        if enemy_vampire:
            enemy_health = min(enemy_health + damage, enemy_max_health)

There are two modifiers to the python statement that change its
behavior:

``hide``

    If given the hide modifier, the python statement will run the
    code in an anonymous scope. The scope will be lost when the python
    block terminates.

    This allows python code to use temporary variables that can't be
    saved - but it means that the store needs to be accessed as fields
    on the store object, rather than directly.

``in``

   The ``in`` modifier takes a name. Instead of executing in the
   default store, the python code will execute in the store that
   name.


One-line Python Statement
-------------------------

A common case is to have a single line of python that runs in the
default store. For example, a python one-liner can be used to
initialize or update a flag. To make writing python one-liners
more convenient, there is the one-line python statement.

The one-line python statement begins with the dollar-sign ($)
character, and contains all of the code on that line. Here
are some example of python one-liners::

    # Set a flag.
    $ flag = True

    # Initialize a variable.
    $ romance_points = 0

    # Increment a variable.
    $ romance_points += 1

    # Call a function that exposes Ren'Py functionality.
    $ renpy.movie_cutscene("opening.ogv")

Python one-liners always run in the default store.


.. _init-python-statement:

Init Python Statement
---------------------

The ``init python`` statement runs python code at initialization time,
before the game loads. Among other things, this code can be used to define
classes and functions, or to initialize styles, config variables, or
persistent data. ::

    init python:

        def auto_voice_function(ident):
            return "voice/" + ident + ".ogg"

        config.auto_voice = auto_voice_function

        if persistent.endings is None:
            persistent.endings = set()

    init 1 python:

        # The bad ending is always unlocked.
        persistent.endings.add("bad_ending")

A priority number can be placed between ``init`` and ``python``. When
a priority is not given, 0 is used. Init  statements are run in priority
order, from lowest to highest. Init statements of the same priority are run in
unicode order by filename, and then from top to bottom within a file.

To avoid conflict with Ren'Py, creators should use priorities in the
range -999 to 999. Priorities of less than 0 are generally used for
libraries and to set up themes. Normal init code should have a priority
of 0 or higher.

Init python statements also take the ``hide`` or ``in`` clauses.

Variables that have their value set in an init python block are not
saved, loaded, and do not participate in rollback, unless the object
the variable refers to is changed.


.. _define-statement:

Define Statement
----------------

The define statement sets a single variable in the default store
to a value at init time. For example::

    define e = Character("Eileen")

is equivalent to::

    init python:
        e = Character("Eileen")

The define statement can take an optional named store (see below), by
prepending it to the variable name with a dot. For example::

    define character.e = Character("Eileen")

One advantage of using the define statement is that it records the
filename and line number at which the assignment occurred, and
makes that available to the navigation feature of the launcher.

Names in the Store
------------------

The default place that Ren'Py stores Python variables is called the
store. It's important to make sure that the names you use in the
store do not conflict.

The define statement assigns a value to a variable, even when it's
used to define a character. This means that it's not possible to
use the same name for a character and a flag.

The following faulty code::

    define e = Character("Eileen")

    label start:

        $ e = 0

        e "Hello, world."

        $ e += 1
        e "You scored a point!"

will not work, because the variable `e` is being used as both a
character and a flag. Other things that are usually placed into
the store are transitions and transforms.

Names beginning with underscore (\_) are reserved for Ren'Py's
internal use. In addition, there is an :ref:`Index of Reserved Names <reserved-names>`.


Other Named Stores
------------------

Named stores provide a way of organizing python code into modules. By
placing code in modules, you can minimize the chance of name
conflicts.

Named stores can be accessed by supplying the ``in`` clause to
``python`` or ``init python``, code can run accessed in a named
store. Each store corresponds to a python module. The default store is
``store``, while a named store is accessed as ``store``.`name`. These
python modules can be imported using the python import statement,
while names in the modules can be imported using the python from
statement.

For example::

    init python in mystore:

        serial_number = 0

        def serial():

            global serial_number
            serial_number += 1
            return serial_number

    init python:
        import store.mystore as mystore

    label start:
        $ serial = mystore.serial()


Named stores participate in save, load, and rollback in the same way
that the default store does. The defined statement can be used to
define names in a named store.


