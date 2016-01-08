CDMS Python Application Programming Interface
=============================================

This chapter describes the CDMS Python application programming interface (API). Python is a popular public-domain, object-oriented language. Its features include support for object-oriented development, a rich set of programming constructs, and an extensible architecture. CDMS itself is implemented in a mixture of C and Python. In this chapter the assumption is made that the reader is familiar with the basic features of the Python language.

Python supports the notion of a module, which groups together associated classes and methods. The ``import`` command makes the module accessible to an application. This chapter documents the cdms2_, cdtime_, and regrid2_ modules.

The chapter sections correspond to the CDMS classes. Each section contains tables describing the class internal (non-persistent) attributes, constructors (functions for creating an object), and class methods (functions). A method can return an instance of a CDMS class, or one of the Python types:

.. table:: Table 2.1 Basic Python Types

   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   |Type                        | Description                                                                                                                                                                                         |
   +============================+=====================================================================================================================================================================================================+
   |``ndarray``, ``MaskedArray``| NumPy or masked multidimensional data array. All elements of the array are of the same type. Defined in the num py and num py.core.ma modules.                                                      |
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
   | ``Reltime``                | Relative time value, a time with representation (value, "units since basetime"). Defined in the cdtime_ module. cf. ``comptime``                                                                    |
   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | ``Tuple``                  | An ordered sequence of objects, which need not be of the same type. Unlike lists, tuples elements cannot be inserted or appended. Tuples are denoted with parentheses, e.g., ``(1, 2.0, 'x', 'y')`` |
   +----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

The following Python script reads January and July monthly temperature data from an input dataset, averages over time, and writes the results to an output file. The input temperature data is ordered (time, latitude, longitude).

::

   #!/usr/bin/env  python
   import cdms2
   from cdms2 import MV2 as MV

   jones = cdms2.open("/pcmdi/cdms/obj/jones_mo.nc")
   tasvar = jones["tas"]
   jans = tasvar[0::12]
   julys = tasvar[6::12]
   
   janavg = MV.average(jans)
   janavg.id = "tas_jan"
   janavg.long_name = "mean January surface temperature"

   julyavg = MV.average(julys)
   julyavg.id = "tas_jul"
   julyavg.long_name = "mean July surface temperature"

   out = cdms2.open("janjuly.nc", "w")
   out.write(janavg)
   out.write(julyavg)
   out.comment = "Average January/July from Jones dataset"
   jones.close()
   out.close()


.. table:: Example Line Notes

  +---------+------------------------------------------------------------------------------------------------+
  | Line(s) | Notes                                                                                          |
  +=========+================================================================================================+
  | 2,3     | Makes the cdms2 and MV2 modules available. MV2 defines arithmetic functions.                   |
  +---------+------------------------------------------------------------------------------------------------+
  | 5       | Opens a netCDF file read-only. The result jones is a dataset object.                           |
  +---------+------------------------------------------------------------------------------------------------+
  | 6       | Gets the surface air temperature variable. 'tas' is the name of the variable in the input      |
  |         | dataset. This does not actually read the data.                                                 |
  +---------+------------------------------------------------------------------------------------------------+
  | 7       | Read all January monthly mean data into a variable jans. Variables can be sliced like arrays.  |
  |         | The slice operator [0::12] means 'take every 12th slice from dimension 0, starting at index 0  |
  |         | and ending at the last index.' If the stride 12 were omitted, it would default to 1.           |
  |         |                                                                                                |
  |         | Note that the variable is actually 3-dimensional. Since no slice is specified for the second or|
  |         | third dimensions, all values of those dimensions are retrieved. The slice operation could also |
  |         | have been written [0::12, : , :].                                                              |
  |         |                                                                                                |
  |         | Also note that the same script works for multi-file datasets. CDMS opens the needed data files,|
  |         | extracts the appropriate slices, and concatenates them into the result array.                  |
  +---------+------------------------------------------------------------------------------------------------+
  | 8       | Reads all July data into a masked array julys.                                                 |
  +---------+------------------------------------------------------------------------------------------------+
  | 10      | Calculate the average January value for each grid zone. Any missing data is handled            |
  |         | automatically.                                                                                 |
  +---------+------------------------------------------------------------------------------------------------+
  | 11,12   | Set the variable id and long_name attributes. The id is used as the name of the variable when  |
  |         | plotted or written to a file.                                                                  |
  +---------+------------------------------------------------------------------------------------------------+
  | 18      | Create a new netCDF output file named 'janjuly.nc' to hold the results.                        |
  +---------+------------------------------------------------------------------------------------------------+
  | 19      | Write the January average values to the output file. The variable will have id "tas_jan" in    |
  |         | the file.                                                                                      |
  |         |                                                                                                |
  |         | write is a utility function which creates the variable in the file, then writes data to the    |
  |         | variable. A more general method of data output is first to create a variable, then set a slice |
  |         | of the variable.                                                                               |
  |         |                                                                                                |
  |         | Note that janavg and julavg have the same latitude and longitude information as tasvar. It is  |
  |         | carried along with the computations.                                                           |
  +---------+------------------------------------------------------------------------------------------------+
  | 21      | Set the global attribute 'comment'.                                                            |
  +---------+------------------------------------------------------------------------------------------------+
  | 23      | Close the output file.                                                                         |
  +---------+------------------------------------------------------------------------------------------------+

.. _cdms2:

CDMS2 Module
------------

The cdms2 module is the Python interface to CDMS. The objects and methods in this chapter are made accessible with the command:

::

  import cdms2

Version 5 of CDMS is based on the NumPy package. Previous versions were based on the Numeric package, the predecessor to NumPy. In those versions the module was named cdms. Although NumPy and Numeric are quite similar, NumPy is a complete rewrite of Numeric. There are enough differences in the structure and behavior of the packages that a change of module name is warranted. Version 5 includes a utility that simplifies the conversion of Numeric/MA/cdms scripts to run with num py/ma/cdms2. See the transition guide for details.
The functions described in this section are not associated with a class. Rather, they are called as module functions, e.g.,

::

  file = cdms2.open('sample.nc')

.. py:module:: cdms2


.. py:function:: asVariable(s)

  Transform s into a transient variable.

  ``s`` is a masked array (num py.core.ma.MaskedArray), NumPy ndarray, or Variable. If s is already a transient variable, s is returned.

  See also: :py:func:`~cdms2.isVariable`.


.. py:function:: createAxis(data, bounds=None)

  Create a one-dimensional coordinate Axis, which is not associated with a file or dataset. This is useful for creating a grid which is not contained in a file or dataset.

  ``data`` is a one-dimensional, monotonic NumPy array.

  ``bounds`` is an array of shape (len(data),2), such that for all i, data[i] is in the range [bounds[i,0],bounds[i,1]]. If bounds is not specified, the default boundaries are generated at the midpoints between the consecutive data values, provided that the autobounds mode is 'on' (the default). 

  See :py:func:`~cdms2.setAutoBounds`.  Also see: :py:meth:`~cdms2.CdmsFile.createAxis`


.. py:function:: createEqualAreaAxis(nlat)

  Create an equal-area latitude axis. The latitude values range from north to south, and for all axis values x[i], sin(x[i])- sin(x[i+1]) is constant.

  ``nlat`` is the axis length. The axis is not associated with a file or dataset.


.. py:function:: createGaussianAxis(nlat)

  Create a Gaussian latitude axis. Axis values range from north to south.

  ``nlat`` is the axis length.

  The axis is not associated with a file or dataset.


.. py:function:: createGaussianGrid(nlats, xorigin=0.0, order="yx")

  Create a Gaussian grid, with shape ``(nlats, 2*nlats)``.

  ``nlats`` is the number of latitudes.
  ``xorigin`` is the origin of the longitude axis.
  ``order`` is either ``"yx"`` (lat-lon, default) or ``"xy"`` (lon-lat)


.. py:function:: createGenericGrid(latArray, lonArray, latBounds=None, lonBounds=None, order="yx", mask=None)

  Create a generic grid, that is, a grid which is not typed as Gaussian, uniform, or equal-area. The grid is not associated with a file or dataset.

  ``latArray`` is a NumPy array of latitude values. lonArray is a NumPy array of longitude values

  ``latBounds`` is a NumPy array having shape (len(latArray),2), of latitude boundaries.

  ``lonBounds`` is a NumPy array having shape (len(lonArray),2), of longitude boundaries.

  ``order`` is a string specifying the order of the axes, either "yx" for (latitude, longitude), or "xy" for the reverse.

  ``mask`` (optional) is an integer-valued NumPy mask array, having the same shape and ordering as the grid.


.. py:function:: createGlobalMeanGrid(grid)

  Generate a grid for calculating the global mean via a regridding operation. The return grid is a single zone covering the range of the input grid.
  
  ``grid`` is a RectGrid.


.. py:function:: createRectGrid(lat, lon, order, type="generic", mask=None)

  Create a rectilinear grid, not associated with a file or dataset. This might be used as the target grid for a regridding operation.

  ``lat`` is a latitude axis, created by :py:func:`cdms2.createAxis`.

  ``lon`` is a longitude axis, created by :py:func:`cdms2.createAxis`.

  ``order`` is a string with value "yx" (the first grid dimension is latitude) or "xy" (the first grid dimension is longitude).

  ``type`` is one of 'gaussian','uniform','equalarea', or 'generic'

  If specified, ``mask`` is a two-dimensional, logical NumPy array (all values are zero or one) with the same shape as the grid.
  
  This function creates a global grid. If the latitude bounds are not specified with lat, they are assumed to extend from -90 degrees to 90 degrees.


.. py:function:: createUniformGrid(startLat, nlat, deltaLat, startLon, nlon, deltaLon, order="yx", mask=None)

  Create a uniform rectilinear grid. The grid is not associated with a file or dataset. The grid boundaries are at the midpoints of the axis values.

  ``startLat`` is the starting latitude value.

  ``nlat`` is the number of latitudes. If ``nlat`` is 1, the grid latitude boundaries will be ``startLat`` +/- ``deltaLat``/2.

  ``deltaLat`` is the increment between latitudes.

  ``startLon`` is the starting longitude value.

  ``nlon`` is the number of longitudes. If ``nlon`` is 1, the grid longitude boundaries will be startLon +/- deltaLon/2.

  ``deltaLon`` is the increment between longitudes.

  ``order`` is a string with value "yx" (the first grid dimension is latitude) or "xy" (the first grid dimension is longitude).

  If specified, ``mask`` is a two-dimensional, logical NumPy array (all values are zero or one) with the same shape as the grid.


.. py:function:: createUniformLatitudeAxis(startLat, nlat, deltaLat)

  Create a uniform latitude axis. The axis boundaries are at the midpoints of the axis values. The axis is designated as a circular latitude axis.
  
  ``startLat`` is the starting latitude value.
  
  ``nlat`` is the number of latitudes.
  
  ``deltaLat`` is the increment between latitudes.

.. py:function:: createZonalGrid(grid)

  Create a zonal grid. The output grid has the same latitude as the input grid, and a single longitude. This may be used to calculate zonal averages via a regridding operation.
  
  ``grid`` is a ``RectGrid``.


.. py:function:: createUniformLongitudeAxis(startLon, nlon, deltaLon)

  Create a uniform longitude axis. The axis boundaries are at the midpoints of the axis values. The axis is designated as a circular longitude axis.
  
  ``startLon`` is the starting longitude value.

  ``nlon`` is the number of longitudes.

  ``deltaLon`` is the increment between longitudes.


.. py:function:: createVariable(array, typecode=None, co py=0, savespace=0, mask=None, fill_value=None, grid=None, axes=None, attributes=None, id=None)
  
  This function is documented in Table 2.34 on page 90.


.. py:function:: getAutoBounds()

  Get the current autobounds mode. Returns 0, 1, or 2. See :py:func:`~cdms2.setAutoBounds`.


.. py:function:: getNumericCompatibility()

  Get the Numeric compatibility mode. See :py:func:`~cdms2.setNumericCompatibility`.


.. py:function:: isVariable(s)

  Return 1 if s is a variable, 0 otherwise. See also: :py:func:`~cdms2.asVariable`.


.. py:function:: open(url,mode='r')

  Open or create a ``Dataset`` or ``CdmsFile``.
  
  ``url`` is a Uniform Resource Locator, referring to a cdunif or XML file. If the URL has the extension '.xml' or '.cdml', a Dataset is returned, otherwise a CdmsFile is returned. If the URL protocol is 'http', the file must be a '.xml' or '.cdml' file, and the mode must be 'r'. If the protocal is 'file' or is omitted, a local file or dataset is opened.

  ``mode`` is the open mode. See Table 2.24 on page 70.

  **Example:** Open an existing dataset:
  ::
  
    f = cdms2.open("sampleset.xml")
  
  **Example:** Create a netCDF file:
  ::
  
    f = cdms2.open("newfile.nc",'w')



.. py:function:: order2index (axes, orderstring)

  Find the index permutation of axes to match order. Return a

  ``list`` of indices

  ``axes`` is a list of axis objects.

  ``orderstring`` is defined as in :py:func:`~cdms2.orderparse`.


.. py:function:: orderparse(orderstring)

  Parse an order string. Returns a list of axes specifiers. 

  ``orderstring`` consists of:

  - Letters t, x, y, z meaning time, longitude, latitude, level
  - Numbers 0-9 representing position in axes
  - Dash (-) meaning insert the next available axis here.
  - The ellipsis ... meaning fill these positions with any remaining axes.
  - (name) meaning an axis whose id is name


.. py:function:: setAutoBounds(mode)

  Set autobounds mode. In some circumstances CDMS can generate boundaries for 1-D axes and rectilinear grids, when the bounds are not explicitly defined. The autoBounds mode determines how this is done:

  - If mode is 'grid' or 2 (the default), the getBounds method will automatically generate boundary information for an axis or grid if the axis is designated as a latitude or longitude axis, and the boundaries are not explicitly defined.
  - If mode is 'on' or 1, the getBounds method will automatically generate boundary information for an axis or grid, if the boundaries are not explicitly defined.
  - If mode is 'off' or 0, and no boundary data is explicitly defined, the bounds will NOT be generated; the getBounds method will return None for the boundaries.

  **Note: In versions of CDMS prior to V4.0, the default mode was 'on'.**
 

.. py:function:: setClassifyGrids(mode)

  Set the grid classification mode. This affects how grid type is determined, for the purpose of generating grid boundaries.

  - If mode is 'on' (the default), grid type is determined by a grid classification method, regardless of the value of grid.getType().
  - If mode is 'off', the value of grid.getType() determines the grid type
 

.. py:function:: setNumericCompatibility(mode)

  Set the mode for backward compatibility with the Numeric module.
  
  - If mode is False (the default), 0-D slices of CDMS variables are returned as scalars, and MV functions with an axis arg have a default axis value of None, meaning to ravel the array.
  - If mode is True, 0-D slices are returned as 0-D arrays, and the default axis arg of MV functions is 0, meaning the first axis.
  
  This function is new in Version 5.


.. py:function:: writeScripGrid(path, grid, gridTitle=None)

  Write a grid to a SCRIP grid file.

  path is a string, the path of the SCRIP file to be created. grid is a CDMS grid object. It may be rectangular. gridTitle is a string ID for the grid.


.. table:: Table 2.2 Class Tags

  +-----------+------------------+
  | Tag       | Class            |
  +===========+==================+
  | 'axis'    | Axis             |
  +-----------+------------------+
  | 'database'| Database         |
  +-----------+------------------+
  | 'dataset' | Dataset, CdmsFile|
  +-----------+------------------+
  | 'grid'    | RectGrid         |
  +-----------+------------------+
  | 'variable'| Variable         |
  +-----------+------------------+
  | 'xlink'   | Xlink            |
  +-----------+------------------+

.. py:class:: CdmsObj

  A CdmsObj is the base class for all CDMS database objects. At the application level, CdmsObj objects are never created and used directly. Rather the subclasses of CdmsObj (Dataset, Variable, Axis, etc.) are the basis of user application programming.

  .. py:attribute:: attributes
    
    A Python dictionary, which contains all the external (persistent) attributes associated with the object. This is in contrast to the internal, non-persistent attributes of an object, which are built-in and predefined. When a CDMS object is written to a file, the external attributes are written, but not the internal attributes.
  
    **Example:** Get a list of all external attributes of the object.
    ::
    
      extatts = obj.attributes.keys()

    All attributes, external and internal, can be accessed and set using the Python dot notation (".")

    **Example:** Get and set an external attribute
    
    ::

      >> obj.my_custom_attribute = "hello"
      >> print obj.my_custom_attribute
      
      hello


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

 - a masked data array, as defined in the NumPy num py.core.ma module. 
 - a domain: an ordered list of axes and/or grids.
 - an attribute dictionary.

 The MV2 module is a work-alike replacement for the ``num py.core.ma`` module, that carries along the domain and attribute information where appropriate. MV2 provides the same set of functions as num py.core.ma. However, MV2 functions generate transient variables as results. Often this simplifies scripts that perform computation. num py.core.ma is part of the Python NumPy package, documented at http://num py.sci py.org.


.. _api-databases:

 Databases
 ---------

 This is some stuff on DB