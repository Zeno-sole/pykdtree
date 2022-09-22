========
pykdtree
========

Objective
---------
pykdtree is a kd-tree implementation for fast nearest neighbour search in Python.
The aim is to be the fastest implementation around for common use cases (low dimensions and low number of neighbours) for both tree construction and queries.

The implementation is based on scipy.spatial.cKDTree and libANN by combining the best features from both and focus on implementation efficiency.

The interface is similar to that of scipy.spatial.cKDTree except only Euclidean distance measure is supported.

Queries are optionally multithreaded using OpenMP.

Installation
------------
Default build of pykdtree with OpenMP enabled queries using libgomp

.. code-block:: bash

    $ cd <pykdtree_dir>
    $ python setup.py install

If it fails with undefined compiler flags or you want to use another OpenMP implementation please modify setup.py at the indicated point to match your system.

Building without OpenMP support is controlled by the USE_OMP environment variable

.. code-block:: bash

    $ cd <pykdtree_dir>
    $ export USE_OMP=0
    $ python setup.py install

Usage
-----
The usage of pykdtree is similar to scipy.spatial.cKDTree so for now refer to its documentation

    >>> from pykdtree.kdtree import KDTree
    >>> kd_tree = KDTree(data_pts)
    >>> dist, idx = pkd_tree.query(query_pts, k=8)
    
The number of threads to be used in OpenMP enabled queries can be controlled with the standard OpenMP environment variable OMP_NUM_THREADS.

The **leafsize** argument (number of data points per leaf) for the tree creation can be used to control the memory overhead of the kd-tree. pykdtree uses a default **leafsize=16**. 
Increasing **leafsize** will reduce the memory overhead and construction time but increase query time.    

pykdtree accepts data in double precision (numpy.float64) og single precision (numpy.float32) floating point. If data of another type is used an internal copy in double precision is made resulting in a memory overhead. If the kd-tree is constructed on single precision data the query points must be single precision as well.

Benchmarks
----------
Comparison with scipy.spatial.cKDTree and libANN. This benchmark is on geospatial 3D data with 10053632 data points and 4276224 query points. The results are indexed relative to the construction time of scipy.spatial.cKDTree. A leafsize of 10 (scipy.spatial.cKDTree default) is used.

Note: libANN is *not* thread safe. In this benchmark libANN is compiled with "-O3 -funroll-loops -ffast-math -fprefetch-loop-arrays" in order to achieve optimum performance.

==================  =====================  ======  ========  ==================
Operation           scipy.spatial.cKDTree  libANN  pykdtree  pykdtree 4 threads
------------------  ---------------------  ------  --------  ------------------

Construction                          100     304        96                  96

query 1 neighbour                    1267     294       223                  70

Total 1 neighbour                    1367     598       319                 166

query 8 neighbours                   2193     625       449                 143

Total 8 neighbours                   2293     929       545                 293  
==================  =====================  ======  ========  ==================

Looking at the combined construction and query this gives the following performance improvement relative to scipy.spatial.cKDTree

==========  ======  ========  ==================
Neighbours  libANN  pykdtree  pykdtree 4 threads
----------  ------  --------  ------------------
1            129%      329%                723%                  

8            147%      320%                682%         
==========  ======  ========  ==================

Note: mileage will vary with the dataset at hand and computer architecture. 

Test
----
Run the unit tests using nosetest

.. code-block:: bash

    $ cd <pykdtree_dir>
    $ python setup.py nosetests

Changelog
---------
v0.2 : Reduced memory footprint. Can now handle single precision data internally avoiding copy conversion to double precision. Default leafsize changed from 10 to 16 as this reduces the memory footprint and makes it a cache line multiplum (negligible if any query performance observed in benchmarks). Reduced memory allocation for leaf nodes. Applied patch for building on OS X.

v0.1 : Initial version.
