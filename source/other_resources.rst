.. _other_resources:

Other Resources
===============

.. _linalg:

Linear Algebra Functions (linAlg)
---------------------------------

*TODO*

.. _opsem:

SEM operators (opSEM)
---------------------

*TODO*

.. _mesh_t:

Mesh Class (``mesh_t``)
-----------------------

This section describes commonly-used data structures related to the mesh, the first of which is ``mesh_t``.
For the fluid domain, all mesh information is stored
in the ``nrs->mesh`` object, while for scalars such as temperature, mesh information is stored on the
``nrs->cds->mesh`` object. These meshes differ in cases such as conjugate heat transfer, where the
velocity mesh is distinct from the temperature mesh. NekRS performs domain decomposition to ensure
an even split of the mesh across the number of specified MPI tasks, in order to keep the computational load
across all MPI tasks as even as possible. As a result, the mesh gets split into pieces with approximately
equal degrees of freedom across each MPI task. The ``mesh_t`` members therefore correspond to the local
section of the mesh accessed by the given MPI task. For example, ``Nelements`` corresponds to the local
number of elements allotted to the given MPI task.

To keep the following summary table general, the variable names are referred to simply as living on
the ``mesh`` object, without any differentiation between whether that ``mesh`` object is the object on
``nrs`` or ``nrs->cds``.

.. table:: Important ``mesh_t`` members
   :name:  mesh_data

   ================== ============================ ================== =================================================
   Variable Name      Size                         Host or Device?           Meaning
   ================== ============================ ================== =================================================
   ``comm``           1                            Host               MPI communicator
   ``device``         1                            Host               backend device
   ``dim``            1                            Host               spatial dimension of mesh
   ``elementInfo``    ``Nelements``                Host               phase of element (0 = fluid, 1 = solid)
   ``EToB``           ``Nelements * Nfaces``       Both               mapping of elements to type of boundary condition
   ``N``              1                            Host               polynomial order for each dimension
   ``NboundaryFaces`` 1                            Host               *total* number of faces on a boundary (rank sum)
   ``Nelements``      1                            Host               number of local elements owned by current process
   ``Nfaces``         1                            Host               number of faces per element
   ``Nfp``            1                            Host               number of quadrature points per face
   ``Np``             1                            Host               number of quadrature points per element
   ``rank``           1                            Host               parallel process rank
   ``size``           1                            Host               size of MPI communicator
   ``Nfields``        1                            Host               Number of fields passed to the PDE solver
   ``cht``            1                            Host               conjugate heat transfer status (0 = off, 1 = on)
   ``vmapM``          ``Nelements * Nfaces * Nfp`` Both               quadrature point index for faces on boundaries
   ``x``              ``Nelements * Np``           Both               :math:`x`-coordinates of physical quadrature points
   ``y``              ``Nelements * Np``           Both               :math:`y`-coordinates of physical quadrature points
   ``z``              ``Nelements * Np``           Both               :math:`z`-coordinates of physical quadrature points
   ``Nvgeo``          ``<chk>``                    <chk>              Volumetric geometric factors
   ``Nggeo``          ``<chk>``                    <chk>              Second-order volumetric geometric factors
   ``vertexNodes``    ``<chk>``                    <chk>              Vertex nodes' indices
   ``edgeNodes``      ``<chk>``                    <chk>              Edge nodes' indices
   ``edgeNodes``      ``<chk>``                    <chk>              List of element reference interpolation nodes on element faces
   ``o_LMM``          ``<chk>``                    Device             Lumped mass matrix
   ``U``              ``Nelements*Np``             Both               Mesh velocity (often used with ALE solver)
   ``D``              ``Nelements*Np``             Both               1D Differentiation matrix
   ``o_vgeo``         ``<chk>``                    Device             Volume geometric factors
   ``o_sgeo``         ``<chk>``                    Device             Surface geometric factors
   ================== ============================ ================== =================================================


The second most important structure is ``bcData``. It is often referred to in the ``oudf`` kernels to set boundary conditions.
Its members are typically accessed on the device, and the kernels for setting boundary conditions will iterate over all the GLL points
on a given boundary, setting these values appropriately for each point. The following table details what the most important members
of this structure mean.

.. table:: Important ``bcData`` members
   :name:  bcData_members

   ===================== =======================================================
   Variable Name                Meaning
   ===================== =======================================================
   ``idM``                Element's mesh ID (?)
   ``fieldOffset``        Size of a field (offset for a given component)
   ``id``                 Sideset ID
   ``time``               Current time
   ''x/y/z``              X/Y/Z coordinates
   ``nx/ny/nz``           X/Y/Z normals
   ``t1x/t1y/t1z``        X/Y/Z tangents
   ``t2x/t2y/t2z``        X/Y/Z bitangents
   ``p/u/v/w``            Pressure and the 3 velocity components
   ``scalarID``           ID of the scalar as per the ``par`` file
   ``s``                  Scalar value
   ``flux``               Flux value for flux BC
   ``meshu/meshv/meshw``  Mesh velocity components (used in ALE framework)
   ``trans/diff``         Mesh transport/diffusion coefficients (ALE framework)
   ===================== =======================================================


.. _nek5000_tools:

Building the Nek5000 Tools
--------------------------

*NekRS* does not package the legacy toolkit available with :term:`Nek5000`, e.g., tools for creating or adapting meshes.
It relies instead on the toolbox available with :term:`Nek5000`, which must be installed separately.
To build these scripts, clone the Nek5000 repository, and then navigate to the ``tools`` directory. 
The desired tools can be built here using `./maketools`.

For example, if you want to make the ``genbox`` tool, 

.. code-block:: bash

  cd ~
  git clone https://github.com/Nek5000/Nek5000.git
  cd Nek5000/tools
  ./maketools genbox

This will create binary executables in the ``Nek5000/bin`` directory. 
You may want to add this folder to your ``PATH`` environment variable for quick access to these tools,

.. code-block:: bash

    export PATH+=:$HOME/Nek5000/bin

Additional information about *Nek5000* tools can be found `here <https://nek5000.github.io/NekDoc/tools.html>`_.

.. note::

  The `genbox <https://nek5000.github.io/NekDoc/tools/genbox.html>`_ tool creates simple 2D and 3D meshes that can be used for both *Nek5000* and *NekRS* simulations. 
  For *NekRS* applications, this is most likely the only tool that be of use to most users. 
  Other *Nek5000* tools will be rarely used.

.. note::

   Some tools require additional dependencies such as ``cmake``, ``wget`` (and
   internet access), or X11 libraries. Check the messages from the
   ``maketools`` script and the corresponding ``build.log`` files in each tool
   directory for any errors.

