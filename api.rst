CDMS Python Application Programming Interface
=============================================

This chapter describes the CDMS Python application programming interface (API). Python is a popular public-domain, object-oriented language. Its features include support for object-oriented development, a rich set of programming constructs, and an extensible architecture. CDMS itself is implemented in a mixture of C and Python. In this chapter the assumption is made that the reader is familiar with the basic features of the Python language.

Python supports the notion of a module, which groups together associated classes and methods. The ``import`` command makes the module accessible to an application. This chapter documents the cdms2_, cdtime_, and regrid2_ modules.

The chapter sections correspond to the CDMS classes. Each section contains tables describing the class internal (non-persistent) attributes, constructors (functions for creating an object), and class methods (functions). A method can return an instance of a CDMS class, or one of the Python types:

.. table:: Table 2.1 Basic Python Types

   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   |Type                        | Description                                                                                                                                                                                         |
   +============================+=====================================================================================================================================================================================================+
   |``ndarray``, ``MaskedArray``| NumPy or masked multidimensional data array. All elements of the array are of the same type. Defined in the numpy and numpy.core.ma modules.                                                        |
   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | ``Comptime``               | Absolute time value, a time with representation (year, month, day, hour, minute, second). Defined in the cdtime_ module. cf. ``reltime``                                                            |
   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | ``Dictionary``             | An unordered collection of objects, indexed by key. All dictionaries in CDMS are indexed by strings, e.g.: ``axes['time']``                                                                         |
   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | ``Float``                  | Floating-point value.                                                                                                                                                                               |
   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | ``Integer``                | Integer value.                                                                                                                                                                                      |
   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | ``List``                   | An ordered sequence of objects, which need not be of the same type. New members can be inserted or appended. Lists are denoted with square brackets, e.g., ``[1, 2.0, 'x', 'y']``                   |
   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | ``None``                   | No value returned.                                                                                                                                                                                  |
   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | ``Reltime``                | Relative time value, a time with representation (value, “units since basetime”). Defined in the cdtime_ module. cf. ``comptime``                                                                    |
   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | ``Tuple``                  | An ordered sequence of objects, which need not be of the same type. Unlike lists, tuples elements cannot be inserted or appended. Tuples are denoted with parentheses, e.g., ``(1, 2.0, ’x’, ’y’)`` |
   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


.. _cdms2:

CDMS2 Module
------------

Some stuff here

.. _cdtime:

cdtime Module
-------------

cdtime

.. _regrid2:

Regridding Operations
---------------------

regridding

.. _cdms2-mv2:

MV2 Module
----------

The fundamental CDMS data object is the variable. A variable is comprised of:

 - a masked data array, as defined in the NumPy numpy.core.ma module. 
 - a domain: an ordered list of axes and/or grids.
 - an attribute dictionary.

 The MV2 module is a work-alike replacement for the ``numpy.core.ma`` module, that carries along the domain and attribute information where appropriate. MV2 provides the same set of functions as numpy.core.ma. However, MV2 functions generate transient variables as results. Often this simplifies scripts that perform computation. numpy.core.ma is part of the Python NumPy package, documented at http://numpy.scipy.org.


.. _api-databases:

 Databases
 ---------

 This is some stuff on DB