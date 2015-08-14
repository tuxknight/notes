.. _jinja2_intro:

=============
Jinja2 Intro
=============

.. contents:: Topics

Installation
===============

As a Python egg::

  easy_install Jinja2
  pip install Jinja2

From the tarball release::

  wget https://pypi.python.org/packages/source/J/Jinja2/Jinja2-2.7.3.tar.gz
  tar xzvf Jinja2-2.7.3.tar.gz
  cd Jinja2-2.7.3 && python setup.py install 
  # root privilege required

Installing the development version::

  git clone git://github.com/mitsuhiko/jinja2.git
  cd jinja2
  ln -s jinja2 /usr/lib/python2.X/site-packages
  # package git required

As of version 2.7 Jinja2 depends on the *MarkupSafe* module.

Basic API Usage
==================

The most basic way to create a template and render it is through Template. This however is not the recommended way to work with it if your templates are not loaded from strings but the file system or another data source::

  >>> from jinja2 import Template
  >>> template = Template('Hello {{ name }}!')
  >>> template.render(name='World')
  u'Hello World!'


