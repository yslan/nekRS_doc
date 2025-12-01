.. _boundary_conditions:

Boundary conditions
===================

Boundary conditions for each mesh boundary should normally be set in the :ref:`par_file`, using the ``boundaryTypeMap`` parameter.
This is used within the ``FLUID VELOCITY`` or ``SCALAR FOO`` sections to set the boundary conditions of the respective fields.

Available Types
---------------

All available boundary conditions are summarised in the command line help function for *NekRS*.
The user can find these in the *NekRS* manual using:

.. code-block:: bash

   nrsman par

The boundary conditions are also shown below:

.. literalinclude:: ../_includes/parHelp.txt
   :language: none
   :lines: 1-3, 154-182

Assigning boundary condition types in *NekRS* is handled differently depending on if you are using a third-party meshing tool such as *GMSH*, *ICEM*, *Cubit*, etc. and importing the mesh with *Nek5000* :ref:`tool scripts <nek5000_tools>`, such as ``gmsh2nek``, ``exo2nek`` or ``cgns2nek``, or if you are using a mesh generated with a Nek-native tool, such as `Genbox <https://nek5000.github.io/NekDoc/tools/genbox.html?highlight=genbox>`_.

Boundary conditions are typically assigned in the :ref:`Parameter file <par_file>`.
To setup cases that use third party meshes, where boundaries are identified by a unique integer or boundary ID, you will need to use the ``boundaryIDMap`` parameter in the ``MESH`` section to identify the mesh boundaries, which are then mapped to the desired boundary condition in field section using the ``boundaryTypeMap`` key.
As an example, consider an inlet-outlet pipe flow, where three boundary conditions are applied to the mesh boundary IDs 389 (``udfDirichlet`` - inlet), 231 (``zeroNeumann`` - outlet), 4 (``zeroDirichlet`` - walls).
The corresponding boundary condition assignment in ``.par`` file will be as follows,

.. code-block::

   [MESH]
   boundaryIDMap = 389, 231, 4  #boundary IDs assigned by 3rd party mesher

   [FLUID VELOCITY]
   boundaryTypeMap = udfDirichlet, zeroNeumann, zeroDirichlet


.. note::

  For conjugate heat transfer cases, it is necessary to also specify the velocity boundary map, ``boundaryIDMapFluid``, separately to identify the interface conformal walls between the fluid and solid mesh.
  Extending the above example, assume that the pipe is enclosed in a solid annular insulated jacket where the boundary ID 4 corresponds to the conformal wall between the solid and fluid mesh, while remaining insulated walls of the solid mesh have boundary IDs 10, 20 and 30.

    .. code-block::

       [MESH]
       boundaryIDMap = 389, 231 ,4                    #boundary IDs assigned by 3rd party mesher
       boundaryIDMapFluid = 10, 20, 30, 389, 231, 4   #boundary IDs assigned by 3rd party mesher

       [FLUID VELOCITY]
       boundaryTypeMap = udfDirichlet, zeroNeumann, zeroDirichlet

       [SCALAR TEMPERATURE]
       boundaryTypeMap = zeroNeumann, zeroNeumann, zeroNeumann, udfDirichlet, zeroNeumann, none

For meshes generated using the Nek-native meshing tool `Genbox <https://nek5000.github.io/NekDoc/tools/genbox.html?highlight=genbox>`_, the boundary conditions are typically assigned using three character code in the ``.box`` file itself and these are stored in the generated mesh (``.re2``) file.
*NekRS* will automatically read the boundary conditions from this mesh and explicit boundary condition specification in ``.par`` file is not required.

From an implementation standpoint, the available boundary conditions in *NekRS* can be broadly classified into three major categories.
These are discussed in the following sections.

ZeroDirichlet / ZeroNeumann Type Conditions
"""""""""""""""""""""""""""""""""""""""""""

``zeroDirichlet`` and/or ``zeroNeumann`` type boundary conditions are handled by the *NekRS* code internally and do not require explicit user definition in ``.udf`` file.
As the name suggests, these correspond to the specification of Dirichlet or Neumann boundary value to zero. 

These are listed in the following table. 
(Note: :math:`{\bf \hat e_n}` is the unit vector normal to the boundary face, :math:`{\bf \hat e_x}`, :math:`{\bf \hat e_y}`, and :math:`{\bf \hat e_z}` are unit vectors aligned with the Cartesian axes, :math:`{\bf \hat e_t}`, and :math:`{\bf \hat e_b}` are the tangent, and bitangent unit vectors.)

.. _tab:zerobcs:

.. csv-table:: ``boundaryTypeMap`` values for ZeroDirichlet / ZeroNeumann type boundary conditions in ``.par`` file
   :widths: 20,30,25,25
   :header: Key, Alternate Legacy Key(s), Description: Fluid BC, Description: Scalar BC

   ``zeroDirichlet``,``w`` / ``wall`` (fluid only), :math:`\mathbf u = 0`, "N/A" 
   ``zeroNeumann``,"Fluid: ``o`` / ``O`` / ``outlet`` / ``outflow`` |br| Scalar: ``I`` / ``insulated``", :math:`\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n} = 0`,:math:`\lambda \nabla s \cdot \mathbf{\hat{e}_n} = 0` 
   ``zeroDirichletX/zeroNeumann``,``slipx`` / ``symx`` (fluid only),":math:`\mathbf{u} \cdot \mathbf{\hat{e}_x} = 0` |br| :math:`(\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_y} = 0` |br| :math:`(\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_z} = 0` ","N/A"
   ``zeroDirichletY/zeroNeumann``,``slipy`` / ``symy`` (fluid only),":math:`\mathbf{u} \cdot \mathbf{\hat{e}_y} = 0` |br| :math:`(\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_x} = 0` |br| :math:`(\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_z} = 0` ","N/A"
   ``zeroDirichletZ/zeroNeumann``,``slipz`` / ``symz`` (fluid only),":math:`\mathbf{u} \cdot \mathbf{\hat{e}_z} = 0` |br| :math:`(\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_x} = 0` |br| :math:`(\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_y} = 0` ","N/A"
   ``zeroDirichletN/zeroNeumann``,``slip`` / ``sym`` (fluid only),":math:`\mathbf{u} \cdot \mathbf{\hat{e}_n} = 0` |br| :math:`(\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_{t}} = 0` |br| :math:`(\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_{b}} = 0` ","N/A"
   ``zeroDirichletYZ/zeroNeumann``,``onx`` (fluid only),":math:`\mathbf{u} \cdot \mathbf{\hat{e}_y} = 0` |br|  :math:`\mathbf{u} \cdot \mathbf{\hat{e}_z} = 0` |br| :math:`(\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_x} = 0` ","N/A"
   ``zeroDirichletXZ/zeroNeumann``,``ony`` (fluid only),":math:`\mathbf{u} \cdot \mathbf{\hat{e}_x} = 0` |br|  :math:`\mathbf{u} \cdot \mathbf{\hat{e}_z} = 0` |br| :math:`(\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_y} = 0` ","N/A"
   ``zeroDirichletXY/zeroNeumann``,``onz`` (fluid only),":math:`\mathbf{u} \cdot \mathbf{\hat{e}_x} = 0` |br|  :math:`\mathbf{u} \cdot \mathbf{\hat{e}_y} = 0` |br| :math:`(\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_z} = 0` ","N/A"


.. _bcdata:

User Defined Dirichlet/Neumann Conditions
""""""""""""""""""""""""""""""""""""""""""

If a boundary condition requires specifying a value rather than just a type, the value must be defined in the ``udfDirichlet`` or ``udfNeumann`` function calls within the :ref:`okl block <okl_block>` section of ``.udf`` file, for Dirichlet or Neumann boundary types, respectively.
The boundary condition values may be a function of space, time or other user variables.
In order to make it easier for the user to assign varying conditions, ``bcData`` struct is passed as an argument to ``udfDirichlet`` and ``udfNeumann`` functions, which makes several variables availables.

.. _tab:bcdataconst:

.. csv-table:: ``const`` Variables in the ``bcData`` struct available in ``udfDirichlet`` and ``udfNeumann`` 
   :widths: 30,40,40
   :header: Variable, Data Type, Description

   ``fieldName``,``char``,Character array to store field name
   ``fieldOffset``,``int``,array offset
   ``idxVol``,``int``,volume index of boundary GLL
   ``id``,``int``,boundary ID tag
   ``time``,``double``,current simulation time
   "``x`` , ``y`` , ``z``",``dfloat``,location coordinates of boundary GLL
   "``nx`` , ``ny`` , ``nz``",``dfloat``,local normal vector components
   "``t1x`` , ``t1y`` , ``t1z``",``dfloat``,local tangent vector components
   "``t2x`` , ``t2y`` , ``t2z``",``dfloat``,local bi-tangent vector components
   ``idScalar``,``int``,scalar index
   ``transCoeff``,``dfloat``,field transport coefficient
   ``diffCoeff``,``dfloat``,field diffusion coefficient
   ``usrwrk``,``@globalPtr const dfloat*``, pointer to user work array
   "``uxFluidInt`` , ``uyFluidInt`` , ``uzFluidInt``", ``dfloat``, interpolated velocity components (NekNek)
   ``sScalarint``, ``dfloat``, interpolated scalar value (NekNek)

All the above variables are ``const`` type, i.e., they cannot be modified and are only provided to build boundary conditions.
The variables in ``bcData`` struct to which the user can assign boundary conditions are as follows:

.. _tab:bcdata:

.. csv-table:: Variables in the ``bcData`` struct available in ``udfDirichlet`` and ``udfNeumann`` to which boundary values are assigned  
   :widths: 30,10,30,30
   :header: Variable, Data Type, BC Type, Description

   "``uxFluid`` , ``uyFluid`` , ``uzFluid``", ``dfloat``, Dirichlet, fluid velocity components
   ``pFluid``,``dfloat``, Dirichlet, fluid pressure
   "``tr1``",``dfloat``, Neumann, fluid traction along tangent
   "``tr2``",``dfloat``, Neumann, fluid traction along bi-tangent
   ``sScalar``, ``dfloat``, Dirichlet, scalar value 
   ``fluxScalar``,``dfloat``, Neumann, scalar flux
   "``uxGeom`` , ``uyGeom`` , ``uzGeom``", ``dfloat``, Dirichlet, mesh velocity components
   ``h``, ``dfloat``, Robin, Heat transfer coefficient
   ``sInfScalar``, ``dfloat``, Robin, ambient temperature :math:`(T_\infty)`

To specify the user defined boundary condition to the corresponding field, the field is identified in the udf functions using the ``isField()`` function.
The ``isField()`` macro is defined in the ``bcData.h`` file and it takes field string identifier as the parameter. 
The string field identifiers are:

+-----------------------------------+-------------------------------------------+
| String Identifier                 | Field                                     |
+===================================+===========================================+
| ``"fluid velocity"``              | velocity                                  |
+-----------------------------------+-------------------------------------------+
| ``"fluid pressure"``              | pressure                                  |
+-----------------------------------+-------------------------------------------+
| ``"scalar foo"``                  | field corresponding to scalar ``"FOO"``   |
+-----------------------------------+-------------------------------------------+
| ``"geom"``                        | (moving) mesh                             |
+-----------------------------------+-------------------------------------------+

.. note::

    The scalar field string identifiers are declared in :ref:`Parameter file <par_file>` under the :ref:`General Section<sec:generalpars>`.

The boundary condition types that require inclusion of user defined functions in ``.udf`` file are shown in the following table:

.. _tab:udfbcs:

.. csv-table:: ``boundaryTypeMap`` values for user defined boundary conditions in ``.par`` file and corresponding assigned variables in ``bcData`` struct 
   :widths: 20,20,20,20,20
   :header: Key, Alternate Legacy Key(s), ``bcData`` Variable: Fluid , ``bcData`` Variable: Scalar, ``bcData`` Variable: Geom

   ``udfDirichlet``,"Fluid: ``v`` / ``inlet`` |br| Scalar: ``t`` / ``inlet`` |br| Geom: ``t`` / ``inlet``", "``bc->uxFluid`` |br| ``bc->uyFluid`` |br| ``bc->uzFluid``","``bc->sScalar``", "``bc->uxGeom`` |br| ``bc->uyGeom`` |br| ``bc->uzGeom``"
   ``udfNeumann``,"``f`` / ``flux`` (scalar only)", "N/A", "``bc->fluxScalar`` :math:`\left(\lambda \nabla s \cdot \mathbf{\hat{e}_n} \right)`","N/A"
   ``zeroDirichletN/udfNeumann``, "``traction`` / ``shl`` (fluid only)","``bc->tr1``    :math:`\left((\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_{t}} \right)` |br| ``bc->tr2``    :math:`\left((\boldsymbol{\underline \tau} \cdot \mathbf{\hat{e}_n}) \cdot \mathbf{\hat{e}_{b}} \right)` |br| :math:`\mathbf{u} \cdot \mathbf{\hat{e}_n} = 0`    (specified internally)","N/A","N/A"
   ``interpolation``,"Fluid: ``int`` |br| Scalar: ``int``","``bc->uxFluidInt`` |br| ``bc->uyFluidInt`` |br| ``bc->uzFluidInt``","``bc->sScalarInt``","N/A"

For cases where multiple surfaces of the same ``boundaryTypeMap`` exist, the surfaces can be differentiated using the ``id`` variable in ``bcData`` struct.
*NekRS* assigns ``id`` in a sequential manner, starting from ``1``, following the sequence specified by the ``boundaryIDMap`` key in ``MESH`` section in ``.par`` file.
Thus, consider again the specification in ``.par`` file for the inlet-outlet pipe flow example considered above:

.. code-block::

   [MESH]
   boundaryIDMap = 389, 231, 4  #mapped to bc->id = 1, 2, 3

   [FLUID VELOCITY]
   boundaryTypeMap = udfDirichlet, zeroNeumann, zeroDirichlet

   [SCALAR FOO]
   boundaryTypeMap = udfDirichlet, zeroNeumann, udfDirichlet

The inlet surface (``389``) is mapped to ``bc->id=1``, outlet surface to ``bc->id=2`` and walls (``4``) to ``bc->id=3``.

A generic example template of user-defined Dirichlet and Neumann boundary condition specification in :ref:`okl_block` is shown below:

.. code-block:: c++
   
   #ifdef __okl__

   void udfDirichlet(bcData *bc) 
   {
      if(isField("fluid velocity")) {
       if(bc->id == 1) {                // inlet surface id
        bc->uxFluid = 1.0;
        bc->uyFluid = 0.0;
        bc->uzFluid = 0.0;
       }
      }
      else if (isField("fluid pressure")) {
        bc->pFluid = 0.0;
      }
      else if (isField("scalar foo")) {
        if(bc->id == 1) bc->sScalar = 1.0;  // inlet id
        if(bc->id == 3) bc->sScalar = 0.0;  // wall id
      }
   }

   void udfNeumann(bcData *bc)
   {
    if(isField("fluid velocity")) {
      bc->tr1 = 0.0;
      bc->tr2 = 0.0;
    }
    else if (isField("scalar foo")) {
      bc->fluxScalar = 0.0;
    }
   }
   #endif

.. note::

   For the ``interpolation`` boundary condition (used in NekNek case) the relevant ``bcData`` variables must be assigned to the field variables, as shown:

   .. code-block:: c++

      void udfDirichlet(bcData * bc) 
      {
        if(isField("fluid velocity")) {
          bc->uxFluid = bc->uxFluidInt;
          bc->uyFluid = bc->uyFluidInt;
          bc->uzFluid = bc->uzFluidInt;
      }
        else if (isField("scalar foo")) {
          bc->sScalar = bc->sScalarInt;
        }
      }

.. tip::

    Some example cases from the `NekRS Github repo <https://github.com/Nek5000/nekRS/tree/next>`_ are listed below to show the usage of different user defined boundary condition types:

    +------------------+-------------------------------------+------------------------------------------------------------------------------------+
    | BC type          | Field                               | Example case                                                                       |
    +------------------+-------------------------------------+------------------------------------------------------------------------------------+
    | ``udfDirichlet`` | ``"fluid velocity"``                | `ethier <https://github.com/Nek5000/nekRS/tree/next/examples/ethier>`_             |
    |                  +-------------------------------------+------------------------------------------------------------------------------------+
    |                  | ``"fluid velocity"`` (recycling)    | `turbPipe <https://github.com/Nek5000/nekRS/tree/next/examples/turbPipe>`_         |
    |                  +-------------------------------------+------------------------------------------------------------------------------------+
    |                  | ``"fluid velocity"`` (interpolated) | `eddyNekNek <https://github.com/Nek5000/nekRS/tree/next/examples/eddyNekNek>`_     |
    |                  +-------------------------------------+------------------------------------------------------------------------------------+
    |                  | ``"fluid pressure"``                | `hemi <https://github.com/Nek5000/nekRS/tree/next/examples/hemi>`_                 |
    |                  +-------------------------------------+------------------------------------------------------------------------------------+
    |                  | ``"scalar XXX"``                    | `conj_ht <https://github.com/Nek5000/nekRS/tree/next/examples/conj_ht>`_           |
    |                  +-------------------------------------+------------------------------------------------------------------------------------+
    |                  | ``"scalar XXX"`` (interpolated)     | `eddyNekNek <https://github.com/Nek5000/nekRS/tree/next/examples/eddyNekNek>`_     |
    |                  +-------------------------------------+------------------------------------------------------------------------------------+
    |                  | ``"geom"``                          | `mv_cyl <https://github.com/Nek5000/nekRS/tree/next/examples/mv_cyl>`_             |
    +------------------+-------------------------------------+------------------------------------------------------------------------------------+
    | ``udfNeumann``   | ``"fluid velocity"``                | `gabls1 <https://github.com/Nek5000/nekRS/tree/next/examples/gabls1>`_             |
    |                  +-------------------------------------+------------------------------------------------------------------------------------+
    |                  | ``"scalar XXX"``                    | `ethier <https://github.com/Nek5000/nekRS/tree/next/examples/ethier>`_             |
    +------------------+-------------------------------------+------------------------------------------------------------------------------------+    

.. _periodic_boundary:

Internal / Periodic
"""""""""""""""""""

``None`` is assigned to the ``boundaryTypeMap`` when an internal boundary condition is required or a periodic boundary condition has been set as part of the mesh.
Periodicity is linked to the mesh connectivity and is handled by the meshing tool.

.. note::

   Periodic pairs are part of the mesh connectivity stored in the ``.re2``
   file. Simply changing a boundary condition from ``wall`` to ``periodic`` in
   ``.par`` will most likely not work. See :ref:`mesh_setup_connectivity` for
   details.

Special Cases
-------------

.. _recycling:

Turbulent Inflow (for LES/DNS)
"""""""""""""""""""""""""""""""

Velocity Recycling
^^^^^^^^^^^^^^^^^^^^

NekRS offers an in-built technique to impose fully developed turbulent inflow conditions.  
It comprises recycling the velocity from an offset surface in the domain interior to the inlet boundary resulting in a quasi-periodic domain. 
The required routines are available through the ``planarCopy.hpp`` header file. 
Sample ``.udf`` code to use velocity recycling is as follows,

.. code-block::

  #include "planarCopy.hpp"

  deviceMemory<int> o_bID;
  planarCopy* recyc = nullptr;
  dfloat area, uBulk;

  #ifdef __okl__
   void udfDirichlet(bcData * bc)
   {
    if (isField("fluid velocity")) {
      bc->uxFluid = bc->usrwrk[bc->idxVol + 0 * bc->fieldOffset];
      bc->uyFluid = bc->usrwrk[bc->idxVol + 1 * bc->fieldOffset];
      bc->uzFluid = bc->usrwrk[bc->idxVol + 2 * bc->fieldOffset];
    }
   }
  #endif

  void UDF_Setup()
  {
    auto mesh = nrs->meshV;
    const auto zmax = platform->linAlg->max(mesh->Nlocal, mesh->o_z, platform->comm.mpiComm());
    const auto zmin = platform->linAlg->min(mesh->Nlocal, mesh->o_z, platform->comm.mpiComm());
    const auto zLen = abs(zmax - zmin);
                    
    platform->app->bc->o_usrwrk.resize(mesh->dim * nrs->fluid->fieldOffset);

    uBulk = 1.0;                            // mean bulk velocity
    const dfloat Dh = 1.0;                  // hydraulic diameter
    const dfloat xOffset = 0.0;             // x-offset of recycling surface 
    const dfloat yOffset = 0.0;             // y-offset of recycling surface
    const dfloat zOffset = 10.0 * Dh;       // z-offset of recycling surface

    std::vector<int> bID;
    bID.push_back(1);
    o_bID.resize(bID.size());
    o_bID.copyFrom(bID);

    deviceMemory<dfloat> o_tmp(mesh->Nlocal);
    platform->linAlg->fill(o_tmp.size(), 1.0, o_tmp);
    area = mesh->surfaceAreaMultiplyIntegrate(o_bID, o_tmp);

    recyc = new planarCopy(mesh, 
                           nrs->fluid->o_solution(),
                           mesh->dim,
                           nrs->fluid->fieldOffset,
                           xOffset,
                           yOffset,
                           zOffset,
                           bID[0],
                           platform->app->bc->o_usrwrk);
  }

  void UDF_ExecuteStep(double time, int tstep)
  {
    recyc->execute();

    const auto flux = mesh->surfaceAreaNormalMultiplyVectorIntegrate(nrs->fluid->fieldOffset, o_bID, platform->app->bc->o_usrwrk);
    platform->linAlg->scale(nrs->fluid->fieldOffsetSum, -uBulk * area / flux, platform->app->bc->o_usrwrk);
  }

The above example assumes that the inlet surface, with boundary ID (``bID``) equal to 1, is perpendicular to the *x-y* plane.
The recycling surface is located at an offset distance equal to :math:`20 Dh` from the inlet surface, where :math:`Dh` is the hydraulic diameter of the inlet cross-section. 
``uBulk`` is the desired mean velocity at the inlet boundary. 

.. note::

  The recommended recycling length is :math:`8-10 Dh` [Lund1998]_ to allow the resolution of largest eddies. The user must ensure that the boundary layer is not acted upon by significant pressure gradients or geometrical changes between the inlet and recycle plane.

The interpolated velocities from the recycling surface are stored in the ``o_usrwrk`` array which is accessible in the ``udfDirichlet`` kernel.
Therefore, ``o_usrwrk`` vector must be resized to accomodate the velocity data to ``mesh->dim * nrs->fluid->fieldOffset`` length.
The ``planarCopy()`` call setups up the necessary interpolation procedures to map the velocity from domain interior to the inlet surface.
Note the parameters passed as arguments to the setup routine.
The ``execute()`` routine is called from the ``UDF_ExecuteStep`` routine at every time step.
It, internally, populates the ``o_usrwrk`` array with the scaled instantaneous velocity at the recycle plane based on specified ``ubulk``.
Finally, the captured velocity is specified as inlet Dirichlet boundary condition in ``udfDirichlet`` in the :ref:`okl block <okl_block>` section, as shown above.
Note the proper indexing and offset specified using ``bc->idxVol`` and ``bc->fieldOffset`` for the three velocity components.

.. _dong_outflow:

Stabilized (Dong) Outflow
"""""""""""""""""""""""""""""""""""""""""""""""

*TODO*
