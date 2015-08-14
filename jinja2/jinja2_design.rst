.. _jinja2_design:

===============
Jinja2 Design
===============

.. contents:: Topics

::

  <!DOCTYPE html>
  <html lang="en">
  <head>
      <title>My Webpage</title>
  </head>
  <body>
      <ul id="navigation">
      {% for item in navigation %}
          <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
      {% endfor %}
      </ul>
  
      <h1>My Webpage</h1>
      {{ a_variable }}
  
      {# a comment #}
  </body>
  </html>

A template contains variables and/or expressions, which get replaced with values when a template is rendered; and tags, which control the logic of the template.  

The example shows the default configuration settings. An application developer can change the syntax configuration from {% foo %} to <% foo %>, or something similar.

The default Jinja delimiters are configured as follows::

   * {%...%} for Statements
   * {{...}} for Expressions to print to the template output
   * {#...#} for Comments not included in the template output
   * #...## for Line Statements

Variables
===========

Template variables are defined by the context dictionary passed to the template.

You can use a dot (.) to access attributes of a variable in addition to the standard Python __getitem__ “subscript” syntax ([]).

The following lines do the same thing::

  {{ foo.bar }}
  {{ foo['bar'] }}

Filters 
=========

Variables can be modified by filters. Filters are separated from the variable by a pipe symbol (|) and may have optional arguments in parentheses. Multiple filters can be chained. The output of one filter is applied to the next.

See :ref:`list_of_builtin_filters` below.

Tests
=======

Tests can be used to test a variable against a common expression. To test a variable or expression, you add is plus the name of the test after the variable::

  {% if loop.index is divisibleby 3 %}
  {% if loop.index is divisibleby(3) %}
  {% name is defined %}

See :ref:`list_of_builtin_tests` below.

Comments
==========

To comment-out part of a line in a template, use the comment syntax which is by default set to {# ... #}. This is useful to comment out parts of the template for debugging or to add information for other template designers or yourself::

  {# note: commented-out template because we no longer use this
    {% for user in users %}
      ...
    {% endfor %}
  #}

Whitespace Control
====================

In the default configuration::

  * a single trailing newline is stripped if present
  * other whitespace (spaces, tabs, newlines etc.) is returned unchanged 

If an application configures Jinja to *trim_blocks* , the first newline after a template tag is removed automatically (like in PHP). The *lstrip_blocks* option can also be set to strip tabs and spaces from the beginning of a line to the start of a block. (Nothing will be stripped if there are other characters before the start of the block.)

With both *trim_blocks* and *lstrip_blocks* enabled, you can put block tags on their own lines, and the entire block line will be removed when rendered, preserving the whitespace of the contents.

You can manually disable the *lstrip_blocks* behavior by putting a plus sign (+) at the start of a block::

  <div>
       {%+ if name %}QWQ{% endif %}
  </div>

You can also strip whitespace in templates by hand. If you add a minus sign (-) to the start or end of a block (e.g. a For tag), a comment, or a variable expression, the whitespaces before or after that block will be removed::

  {% for item in seq -%}
      {{ item }}
  {%- endfor %}

This will yield all elements without whitespace between them. If seq was a list of numbers from 1 to 9, the output would be 123456789.

Escaping
==========

The easiest way to output a literal variable delimiter ({{) is by using a variable expression::

  {{ '{{' }}

For bigger sections, it makes sense to mark a block raw. For example, to include example Jinja syntax in a template, you can use this snippet::

  {% raw %}
    <ul>
    {% for item in seq %}
      <li>{{ item }}</li>
    {% endfor %}
    </ul>
  {% endraw %}

.. _line_statements:

Line Statements
=================

If line statements are enabled by the application, it's possible to mark a line as a statement. For example, if the line statement prefix is configured to # , the following two examples are equivalent::

  <ul>
  # for item in seq
    <li>{{ item }}</li>
  # endfor
  </ul>

  <ul>
  {% for item in seq %}
    <li>{{ item }}</li>
  {% endfor %}
  </ul>

The line statement prefix can appear anywhere on the lines as long as no text precedes it.

.. note::

  Line Statements can span multiple lines if there are open parentheses, braces or brackets::

    <ul>
    # for href, caption in [('index.html', 'Index'),
                            ('about.html', 'About')]:
        <li><a href="{{ href }}">{{ caption }}</a></li>
    # endfor
    </ul>

.. _list_of_control_structures:

List of Control Structures
==============================

For
---

Loop over each item in a sequence::

  <h1>Members</h1>
  <ul>
  {% for user in users %}
    <li>{{ user.username|e }}</li>
  {% endfor %}
  </ul>

As varialbes in templates retain their object properties, it is possible to iterate over containers like dict::

  <dl>
  {% for key, value in my_dict.iteritems() %}
    <dt>{{ key|e }}</dt>
    <dd>{{ value|e }}</dd>
  {% endfor %}
  </dl>

Python dicts are not ordered, so a sorted *list* of tuples or a *collections.OrderedDict* or *dictsort* filter can be used to order dict.

Inside of a for-loop block, you can access some special variables:

+--------------+----------------------------------------------------------------------------------------------+
|Variable      |Description                                                                                   |
+==============+==============================================================================================+
|loop.index    |The current iteration of the loop.(1 indexed)                                                 |
+--------------+----------------------------------------------------------------------------------------------+
|loop.index0   |The current iteration of the loop.(0 indexed)                                                 |
+--------------+----------------------------------------------------------------------------------------------+
|loop.revindex |The number of iterations from the end of the loop(1 indexed)                                  |
+--------------+----------------------------------------------------------------------------------------------+
|loop.revindex0|The number of iterations from the end of the loop(0 indexed)                                  |
+--------------+----------------------------------------------------------------------------------------------+
|loop.first    |True if first iteration.                                                                      |
+--------------+----------------------------------------------------------------------------------------------+
|loop.last     |True if last iteration.                                                                       |
+--------------+----------------------------------------------------------------------------------------------+
|loop.length   |The number of items in the sequence.                                                          |
+--------------+----------------------------------------------------------------------------------------------+
|loop.cycle    |A helper function to cycle between a list of sequences.                                       |
+--------------+----------------------------------------------------------------------------------------------+
|loop.depth    |Indicates how deep in deep in a recursive loop the rendering currently is. Starts at level 1. |
+--------------+----------------------------------------------------------------------------------------------+
|loop.depth0   |Indicates how deep in deep in a recursive loop the rendering currently is. Starts at level 0. |
+--------------+----------------------------------------------------------------------------------------------+

||

If
---

The if statement in Jinja is comparable with the Python if statement. You can use it to test if a variable is defined, not empty, or not false::

  {% if users %}
  <ul>
    {% for user in users %}
      <li> {{ user.username|e }} </li>
    {% endfor %}
  </ul>
  {% endif %}

For multiple branches, elif and else can be used like in Python. 

Macros
-------

Macros are comparable with functions in regular programming languages. They are useful to put often used idioms into reusable functions to not repeat yourself (“DRY”).

::

  {% macro input(name, value='', type='text', size=20) -%}
    <input type="{{type}}" name="{{name}}" value="{{value|e}}" size="{{size}}" />
  {%- endmacro %}

Then, macro 'input' can be called like a function in the namespace::

  <p>{{ input('username') }}</p>

Call
------

Filters
--------

Assignments
------------

.. _list_of_builtin_tests:

List of Builtin Tests
===========================

callable(obj)
  return whether the object is callable.

defined(value)
  return true if variable is defined

divisibleby(value, num)
  Check if a variable is divisible by a number.

equalto(value, other)
  Check if an object has the same value as another object.

even(value)
  return true if the variable is even

iterable(value)
  check if it's possible to iterate over an object.

lower(value)
  return true if the variable is lowercased.

maping(value)
  return true if the object is a mapping(dict etc.)

none(value)
  return true if the variable is none

number(value)
  return true if the variable is a number

odd(value)
  return true if the variable is odd

sequence(value)
  return true if the variable is a sequence.

string(value)
  return true if the object is a string.

undefined(value)
  return true if variable is undefined
  
upper(value)  
  return true if the variable is uppercased

.. _list_of_builtin_filters:

List of Builtin Filters
===========================

abs(number)
  absolute value of the argument

attr(obj,name)
  attribute of object. foo|attr("bar") works like foo.bar

capitalize(s)
  capitalize a value

center(value, width=80)
  centers the value in a field of a given width

default(value,default_value=u'', boolean=False)
  If the value is undefined it will return the passed default value, otherwise the value of the variable.

dictsort(value, case_sensitive=False, by='key')
  sort a dict and yield(key,value) pairs.o

escape(s)
  Convert the characters &, <, >, ‘, and ” in string s to HTML-safe sequences. 

filesizeformat(value, binary=False)
  Format the value like a ‘human-readable’ file size (i.e. 13 kB, 4.1 MB, 102 Bytes, etc). Per default decimal prefixes are used (Mega, Giga, etc.), if the second parameter is set to True the binary prefixes are used (Mebi, Gibi).

first(seq)
  first item of a sequence

float(value, default=0.0)
  convert the value into a floating point number.

forceescape(value)
  Enforce HTML escaping

format(value,*args,**kwargs)
  apply python string formatting on an object

groupby(value, attribute)
  Group a sequence of objects by a common attribute.

indent(s, width=4, indentfirst=False)
  return a copy of the passed string, each line indented by 4 spaces

int(value, default=0)
  convert the value into integer

join(value, d=u'', attribute=None)
  Return a string which is the concatenation of the strings in the sequence. The separator between elements is an empty string per default, you can define it with the optional parameter.

last(seq)
  last item of a sequence

length(object)
  return the number of items of a sequence or mapping.

list(value)
  convert the value into a list.

lower(s)
  convert a value to lowercase.

map()
  Applies a filter on a sequence of objects or looks up an attribute. 
  Basic uasge is mapping on an attribute: ``{{ users|map(attribute='username')|join(', ') }}``
  Applies a filter on a sequence: ``{{ titles|map('lower')|join(', ') }}``

pprint(value, verbose=False)
  pretty print a variable, useful for debugging.

random(seq)
  return a random item from the sequence.

replace(s, old, new, count=None)
  return a copy of the value with all occurrences of a substring replaced with a new one.

reverse(value)
  reverse the object or return an iterator the iterates over it the other way round.

round(value, precision=0, method='common')
  round the number to a given precision, 'commom' rounds either up or down, 'ceil' always rounds up, 'floor' always rounds down.

select()
  Filters a sequence of objects by applying a test to the object and only selecting the ones with the test succeeding.

selectattr()
  Filters a sequence of objects by applying a test to an attribute of an object and only selecting the ones with the test succeeding.

slice(value,slices,fill_with=None)
  slice an iterator and return a list of lists containing those items.

sort(value, reverse=False, case_sensitive=False, attribute=None)
  Sort an iterable. Per default it sorts ascending, if you pass it true as first argument it will reverse the sorting.

string(obj)
   make a string unicode if it isn't already

sum(iterable, attribute=None, start=0)
  Returns the sum of a sequence of numbers plus the value of parameter ‘start’ (which defaults to 0). When the sequence is empty it returns start.

trim(value)
  strip leading and trailing whitespace

truncate(s, length=255, killwords=False, end='...')
  Return a truncated copy of the string. The length is specified with the first parameter which defaults to 255.
  ``{{"foo bar baz"|truncate(9,True)}}``  --> "foo ba..."

upper(s)
  convert a value to uppercase

urlencode(value)
  escape strings for use in URLs

wordcount(s)  
  count the words in that string

wordwrap(s,width=79,break_long_words=True,wrapstring=None)
  Return a copy of the string passed to the filter wrapped after 79 characters.

xmlattr9d,autospace=True)
  Create an SGML/XML attribute string based on the items in a dict.

Template inheritance
======================

The most powerful part of Jinja is template inheritance. Template inheritance allows you to build a base “skeleton” template that contains all the common elements of your site and defines blocks that child templates can override.

Base Template
-----------------

Base.html, defines a simple HTML skeleton document that you might use for a simple two-column page. It’s the job of “child” templates to fill the empty blocks with content::
  
  <!DOCTYPE html>
  <html lang="en">
  <head>
      {% block head %}
      <link rel="stylesheet" href="style.css" />
      <title>{% block title %}{% endblock %} - My Webpage</title>
      {% endblock %}
  </head>
  <body>
      <div id="content">{% block content %}{% endblock %}</div>
      <div id="footer">
          {% block footer %}
          &copy; Copyright 2008 by <a href="http://domain.invalid/">you</a>.
          {% endblock %}
      </div>
  </body>
  </html>
  
The {% block %} tags define four blocks that child templates can fill in. All the block tag does is tell the template engine that a child template may override those placeholders in the template.

Child Template
-----------------

A child  template might look like this::
  
  {% extends "base.html" %}
  {% block title %}Index{% endblock %}
  {% block head %}
  {{ super() }}
    <style type="text/css">
    .important { color: #336699; }
    </style>
  {% endblock %}
  {% block content %}
    <h1>Index</h1>
    <p class="important">
    Welcome here
    </p>
  {% endblock %}

{% extends %} tells the template engine that this template "extends" another template.When the template system evaluates this template, it first locates the parent. The extends tag should be the first tag in the template. Everything before it is printed out normally and may cause confusion. 

You can’t define multiple {% block %} tags with the same name in the same template. This limitation exists because a block tag works in “both” directions. That is, a block tag doesn’t just provide a placeholder to fill - it also defines the content that fills the placeholder in the parent. 

If you want to print a block multiple times, you can, however, use the special self variable and call the block with that name::

  <title>{% block title %}{% endblock %}</title>
  <h1>{{ self.title() }}</h1>
  {% block body %}{% endblock %}

.. seealso::

  #. `List of Builtin Filters`_

  #. `List of Builtin Tests`_
       
.. _List of Builtin Filters: http://jinja.pocoo.org/docs/dev/templates/#list-of-builtin-filters
.. _List of Builtin Tests: http://jinja.pocoo.org/docs/dev/templates/#list-of-builtin-tests
