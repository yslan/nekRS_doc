.. _case:

Case files
==========

NekRS simulation requires a number of case definition files which are described in this page.
An overview of these are presented in the image below.
See also :ref:`file_extensions` for summarized tables of all *NekRS* files and their extensions.

.. _fig:case_overview:

.. figure:: ../_static/img/overview.svg
   :align: center
   :figclass: align-center
   :alt: An overview of nekRS case files

There are a minimum of three files required to run a case:

* **Parameter file**, with ``.par`` extension. This sets simulation parameters used by the case and can be modified between runs.
* **User-defined host file**, with ``.udf`` extension. This is used to set specific equations of the case, initial/boundary conditions, data outputs and other user definable behaviour.
* **Mesh file**, with ``.re2`` extension. This file defines the geometry of the case.

Some optional files can also be included:

* **User-defined okl file**, with ``.oudf`` extension. This file conventionally has all the user defined device kernels. It must be included in :ref:`okl_block` of ``.udf`` file.
* **Legacy Nek5000 user file**, with ``.usr`` extension. This file allows usage of *Nek5000*, legacy, Fortran 77 user routines.
* **Session file**, with ``.sess`` extension. This file is for NekNek only.
  (See :ref:`overlapping_overset_grids` tutorial).

The case name is the default prefix applied to these files - for instance, a complete input description with a case name of "eddy" would be given by the files ``eddy.par``, ``eddy.re2``, ``eddy.udf``.
Optionally, the user can also define the names of ``.udf``, ``.oudf`` and ``.usr`` in the :ref:`sec:generalpars` section and name of ``.re2`` file in :ref:`sec:meshpars` section of ``.par`` file.

The following sections describe the structure and syntax for each of these files for a general case.


.. _par_file:

Parameter File (.par)
---------------------

.. tip::

   The user can access the manual containing specifications of the parameter file using ``nrsman`` as follows

   .. code-block:: bash

      nrsman par

.. warning::

   *nekRS* ``.par`` file is not compatible or interchangeable with *Nek5000* ``.par`` file.

Most information about the problem setup is defined in the parameter (``.par``) file.
This file is organized in a number of **sections**, each with a number of **keys**.
Values are assigned to these keys in order to control the simulation settings.

The general structure of the ``.par`` file is as follows, where ``FOO`` and ``BAR`` are both section names, with a number of (key, value) pairs.

.. code-block:: ini

  # This is a comment
  [FOO]
    key = value # this is also a comment
    baz = bat

  [BAR]
    alpha = beta
    gamma = delta + keyword=value + ...   # composed value with inline keywords
    theta = value1, value2, value3        # comma-separated list

The valid sections for setting up a *NekRS* simulation are:

* ``GENERAL``: generic settings for the simulation (mandatory, see :ref:`General Parameters<sec:generalpars>`)
* ``OCCA``: backend :term:`OCCA` device settings (optional, see :ref:`OCCA section<sec:occa>`)
* ``PROBLEMTYPE``: settings for the governing equations (optional, see :ref:`Problem Type<sec:problemtype>`)
* ``MESH``: mesh related settings (optional, see :ref:`Mesh Parameters<sec:meshpars>`)
* ``GEOM``: **TODO**
* Field Settings (see :ref:`Field Settings<sec:field_settings>`)

  * ``FLUID VELOCITY``: settings for the velocity solver

  * ``FLUID PRESSURE``: settings for the pressure solver

  * ``SCALAR``: default scalar settings

    * ``SCALAR FOO``: settings for the ``FOO`` scalar

* ``BOOMERAMG``: settings for the Hypre's :term:`AMG` solver
* ``NEKNEK``: settings for the *NekNek* module in *NekRS* (see :ref:`NekNek Parameters <sec:neknekpars>`)
* ``CVODE``: settings for the CVODE solver (see :ref:`CVODE Parameters <sec:cvodepars>`)

.. note::

  - Section name and key/value pairs are case insensitive
  - Values are words with all spaces removed
  - Values enclosed within quotes preserve case and whitespace
  - Values prefixed with 'env::' are interpreted as references to environment variables
  - Values separated by commas forms a list

.. _sec:user_section:

User Sections
""""""""""""""""""

Custom sections may be added via the ``userSections`` key to pass additional keys into the code.

.. code-block:: ini

   userSections = CASEDATA

   ...

   [CASEDATA]
   key = value


.. _sec:generalpars:

General Parameters
""""""""""""""""""

.. _tab:generalparams:

.. csv-table:: ``GENERAL`` keys in the ``.par`` file
   :widths: 20,20,60
   :class: tall
   :header: Key, Value(s), Description/Note(s)/Default Value

   ``polynomialOrder``,``<int>``, "``polynomialOrder`` > 10 is currently not supported"
   ``dealiasing``,``true`` / ``false``, "Enables/disables over-integration of convective term |br| Default = ``true``"
   ``cubaturePolynomialOrder``,``<int>``, "Polynomial order of ``dealiasing`` |br| Default = 3/2*(``polynomialOrder`` + 1) - 1"
   ``verbose``,``true`` / ``false``, "``true`` instructs *NekRS* to print detailed diagnostics to *logfile* |br| Default = ``false``"
   ``redirectOutputTo``,``<string>``,"String entry for the name of the *logfile* to direct *NekRS* output"
   ``startFrom``,"``""<string>"", ...`` |br| ``+ time=<float>`` |br| ``+ x`` |br| ``+ u`` |br| ``+ s or s00 s01 s02 ...`` |br| ``+ int``", "Restart from specified ``<string>`` file |br| reset ``time`` to specified value |br| read mesh coordinates |br| read velocity |br| read all scalar or specified scalars |br| interpolate solution (useful if mesh coordinates are different)"
   ``timeStepper``,``tombo1`` / ``tombo2`` / ``tombo3``,"Order of time discretization for BDFk/EXTk scheme |br| Default = ``tombo2``"
   ``stopAt``,``numSteps`` / ``endTime`` / ``elapsedTime``, "stop criterion |br| Default = ``numSteps``"
   ``numSteps``,``<int>``, "Number of simulation time steps"
   ``endTime``,``<float>``,"Simulation end time"
   ``elapsedTime``,``<float>``,"Simulation time in wall clock minutes"
   ``dt``,``<float>`` |br| ``+ targetCFL = <float>`` |br| ``+ max = <float>`` |br| ``+ initial = <float>`` , "Time step size |br| adjust ``dt`` to match ``targetCFL`` |br| max limit of ``dt`` |br| Initial ``dt`` "
   ``advectionSubCyclingSteps``,``<int>``,"Number of OIFS sub-steps for advection |br| Default = ``0`` (OIFS turned off)"
   ``constFlowRate``,"``meanVelocity = <float>`` |br| ``meanVolumetricFlow = <float>`` |br| ``+ direction = <X,Y,Z>``","Specifies constant flow velocity |br| Specifies constant volumetric flow rate |br| Specifies flow direction"
   ``scalars``,"``<string>, <string> ...``","Name of scalar fields to be solved"
   ``checkPointEngine``,``<string>`` |br| ``nek`` / ``adios``,"Specifies engine to write field files |br| Default = ``nek``"
   ``checkPointPrecision``,``<int>`` |br| ``32`` / ``64``,"Specifies precision of field files |br| Default = ``32``"
   ``checkPointControl``,``steps`` / ``simulationTime``,"Specifies check point frequency control type |br| Default = ``steps``"
   ``checkPointInterval``,``<int>`` / ``<float>`` |br| 0 |br| -1, "Specifies check point frequency (``<int>`` for ``steps`` / ``<float>`` for ``simulationTime``) |br| ``0`` implies at end of simulation |br| ``-1`` disables checkpointing"
   ``udf``,"``""<string>""``","Optional name of user-defined host function file |br| Default is ``<case>.udf``"
   ``oudf``,"``""<string>""``","Optional name of user-defined OCCA kernel function file |br| As a default *NekRS* expects these are defined in :ref:`OKL block <okl_block>` in ``.udf`` file"
   ``usr``,"``""<string>""``","Optional name of user-defined legacy *Nek5000* (fortran) function file |br| Default is ``<case>.usr``"
   ``regularization``,"","Specifies regularization options for all fields |br| See :ref:`common field settings<sec:common_settings>` for details"

.. _sec:occa:

OCCA Parameters
""""""""""""""""
.. _tab:occaparams:

.. csv-table:: ``OCCA`` keys in the ``.par`` file
   :widths: 20,20,60
   :class: tall
   :header: Key, Value(s), Description/Note(s)/Default Value

   ``backend``, |br| ``SERIAL`` / |br| ``CUDA`` / |br| ``HIP`` /|br| ``DPCPP``,"Specifies the *device* for JIT compilation. Default is defined in ``$NEKRS_HOME/nekrs.conf`` |br| CPU |br| NVIDIA GPU (CUDA) |br| AMD GPU (HIP) |br| Intel GPU (oneAPI)"
   ``deviceNumber``,``<int>`` |br| ``LOCAL-RANK``,"Default is ``LOCAL-RANK``"
   ``platformNumber``,``<int>``, "Only used by ``DPCPP`` |br| Default is ``0``"

.. _sec:problemtype:

Problem Type Parameters
""""""""""""""""""""""""""
.. _tab:problemparams:

.. csv-table:: ``PROBLEMTYPE`` keys in the ``.par`` file
   :widths: 20,20,60
   :class: tall
   :header: Key, Value(s), Description/Note(s)/Default Value

   ``equation``,``stokes`` |br| ``navierStokes`` |br| ``+ variableViscosity``, "Stokes solver |br| Navier-Stokes solver |br| uses stress formulation (required for spatially varying viscosity)"

.. _sec:meshpars:

Mesh Parameters
""""""""""""""""
.. _tab:meshparams:

.. csv-table:: ``MESH`` keys in the ``.par`` file
   :widths: 20,20,60
   :class: tall
   :header: Key, Value(s), Description/Note(s)/Default Value

   ``partitioner``,``rbc`` / ``rsb`` / ``rbc+rsb``,"Specifies mesh partitioner |br| Default = ``rbc+rsb`` "
   ``boundaryIDMap``,"``<int>, <int>, ...``", "Map mesh boundary ids to 1,2,3,... |br| See :ref:`boundary conditions<boundary_conditions>` for details"
   ``boundaryIDMapFluid``,"``<int>, <int>, ...``", "Required for conjugate heat transfer cases |br| See :ref:`boundary conditions<boundary_conditions>` for details"
   ``connectivityTol``,"``<float>``","Specifies mesh tolerance for partitioner |br| Default = ``0.2``"
   ``file``,"``""<string>""``","Optional name of mesh (``.re2``) file |br| Default is ``<case>.re2``"
   ``hrefine``,"``<int>, ...``", "Number of splits per direction along an element |br| See :ref:`meshing_hrefine` for details."


.. _sec:field_settings:

Field Settings
"""""""""""""""""""""

The sections for specific fields, including velocity (``FLUID VELOCITY``), pressure (``FLUID PRESSURE``) and scalars (``SCALAR`` or ``SCALAR FOO``) contain keys to describe linear solver setting for the corresponding field.
Most of the keys in the field sections are similar, described in :ref:`Common Field Settings <sec:common_settings>`.
Some specific field keys are shown below:

.. _tab:velocityparams:

.. csv-table:: ``FLUID VELOCITY`` settings in the ``.par`` file
   :widths: 20,20,60
   :class: tall
   :header: Key, Value(s), Description/Note(s)/Default Value

   ``density`` / ``rho``,``<float>``, "Fluid density"
   ``viscosity`` / ``mu``,``<float>``, "Fluid dynamic viscosity"


.. _tab:scalarparams:

.. csv-table:: ``SCALAR FOO`` settings in the ``.par`` file (specific to scalar ``FOO``)
   :widths: 20,20,60
   :class: tall
   :header: Key, Value(s), Description/Note(s)/Default Value

   ``mesh``,``fluid`` |br| ``+ solid``, "Specifies the mesh region where scalar ``FOO`` is solved (relevant to :term:`CHT` case) |br| Default = ``fluid``"
   ``transportCoeff``,``<float>``, "Transport property for the scalar ``FOO`` (e.g., :math:`\rho c_p` for ``TEMPERATURE``) in the ``fluid`` ``mesh``"
   ``diffusionCoeff``,``<float>``, "Diffusion coefficient for the scalar ``FOO`` (e.g., :math:`k` for ``TEMPERATURE``) in the ``fluid`` ``mesh``"
   ``transportCoeffSolid``,``<float>``, "Transport property for the scalar ``FOO`` (e.g., :math:`\rho c_p` for ``TEMPERATURE``) in the ``solid`` ``mesh``"
   ``diffusionCoeffSolid``,``<float>``, "Diffusion coefficient for the scalar ``FOO`` (e.g., :math:`k` for ``TEMPERATURE``) in the ``solid`` ``mesh``"

.. _sec:common_settings:

Common Field Settings
^^^^^^^^^^^^^^^^^^^^^

The following table describes settings and corresponding keys for the linear solver.
The keys are common to all solution fields, including velocity, pressure and scalar fields.
These are to be included in the ``.par`` file under appropriate section for ``FLUID VELOCITY``, ``FLUID PRESSURE``, general ``SCALAR`` and specific scalar (``SCALAR FOO``).

.. note::

   Linear solver settings for all scalar fields can be commonly specified under the ``SCALAR`` section.
   Any setting under the specific ``SCALAR FOO`` section will override the common settings under ``SCALAR`` for ``FOO`` field

.. _tab:commonparams:

.. csv-table:: Common settings for all fields in the ``.par`` file
   :widths: 15,35,50
   :class: tall
   :header: Key, Value(s), Description/Note(s)/Default Value

   ``solver``,"``none`` |br| ``user`` |br| ``cvode`` |br| ``CG`` |br| ``+ combined`` |br| ``+ block`` |br| ``+ flexible`` |br| ``+ maxiter=<int>`` |br| ``GMRES`` |br| ``+ flexible`` |br| ``+ maxiter=<int>`` |br| ``+ nVector=<int>`` |br|  ``+ iR``","Solve off |br| user-specified |br| CVODE solver (see :ref:`sec:cvodepars`) |br| Conjugate gradient solver. **Default solver for velocity and scalar equation** |br| **Default for scalar equation** |br| **Default velocity solver** |br| . |br| . |br| . |br| Generalized Minimal Residual solver. **Default solver for pressure** |br| **Default for pressure** |br| . |br| Dimension of Krylov space |br| Iterative refinment"
   ``residualTol``,"``<float>`` |br| ``+ relative=<float>``","absolute linear solver residual tolerance. Default = ``1e-4`` |br| use absolute/relative residual (whatever is reached first)"
   ``absoluteTol``,"``<float>``","absolute solver tolerance (for CVODE only) |br| Default = ``1e-6``"
   ``initialGuess``,"``previous`` |br| ``extrapolation`` |br| ``projection`` |br| ``projectionAconj`` |br| ``+ nVector=<int>``", ". |br| **Default for velocity and scalars** |br| . |br| Defaults for pressure |br| dimension of projection space"
   ``preconditioner``,"``Jacobi`` |br| ``multigrid`` |br| ``+ multiplicative`` |br| ``+ additive`` |br| ``+ SEMFEM`` |br| ``SEMFEM``","**Default for velocity and scalars** |br| Polynomial multigrid + coarse grid projection. **Default for pressure** |br| Default |br| . |br| smoothed SEMFEM |br| ."
   ``coarseGridDiscretization``,"``FEM`` |br| ``+ Galerkin`` |br| ``SEMFEM``","Linear finite element discretization. Default |br| coarse grid matrix by Galerkin projection |br| Linear FEM approx on high-order nodes"
   ``coarseSolver/semfemSolver``,"``smoother`` |br| ``jpcg`` |br| ``+ residualTol=<float>`` |br| ``+ maxiter=<int>`` |br| ``boomerAMG`` |br| ``+ smoother`` |br| ``+ cpu`` |br| ``+ device`` |br| ``+ overlap``", ". |br| Jacobi preconditioned CG |br| . |br| . |br| Hypre's AMG solver |br| . |br| . |br| . |br| overlap coarse grid solve in additive MG cycle"
   ``pMGSchedule``,"``p=<int> + degree=<int>, ...``","custom polynomial order and Chebyshev order for each pMG level"
   ``smootherType``,"``Jacobi`` |br| ``ASM, RAS`` |br| ``+ Chebyshev`` |br| ``+ FourthChebyshev`` |br| ``+ FourthOptChebyshev`` |br| ``+ maxEigenvalueBoundFactor=<float>``",". |br| overlapping additive/restrictive Schwarz |br| 1st Kind Chebyshev acceleration |br| 4th Kind Chebyshev acceleration |br| 4th Opt Chebyshev acceleration |br| ."
   ``checkPointing``, ``true``/``false``, "Turns on/off checkpointing for specific field |br| Default = ``true``"
   ``boundaryTypeMap``,"``<bcType for ID 1>, <bcType for ID 2>, ...``","See :ref:`boundary_conditions` for details"
   ``regularization``,"``hpfrt`` |br| ``+ nModes=<int>`` |br| ``+ scalingCoeff=<float>`` |br| ``gjp`` |br| ``+ scalingCoeff=<float>`` |br| ``avm`` |br| ``+ c0`` |br| ``+ scalingCoeff=<float>`` |br| ``+ noiseThreshold=<float>`` |br| ``+ decayThreshold=<float>`` |br| ``+ activationWidth=<float>``","High-pass filter stabilization |br| number of modes |br| filter strength |br| Gradient Jump Penalty |br| scaling factor in penalty factor fit |br| Artificial Viscosity Method |br| make viscosity C0 |br| . |br| smaller values will be considered to be noise |br| . |br| half-width of activation function"

.. _sec:cvodepars:

CVODE Parameters
"""""""""""""""""""""
.. _tab:cvodeparams:

.. csv-table:: ``CVODE`` settings in the ``.par`` file
   :widths: 20,20,60
   :class: tall
   :header: Key, Value(s), Description/Note(s)/Default Value

   ``solver``,"``cbGMRES, GMRES`` |br| ``+ nVector=<int>``", "Linear solver |br| Dimension of Krylov space"
   ``gsType``,"``classical, modified``", ""
   ``relativeTol``,"``<float>``", "relative tolerance |br| Default = ``1e-4``"
   ``epsLin``,``<float>``,"ratio between linear and nonlinear tolerances |br| Default = ``0.5``"
   ``dqSigma``,``<float>``,"step size for Jv difference quotient |br| Default = ``automatic``"
   ``maxSteps``,``<int>``,""
   ``sharedRho``,"``true`` / ``false``", "use same *density* field for all but the first scalar |br| Default = ``false``"
   ``jtvRecycleProperties``,"``true`` / ``false``","recycle property (freeze) evaluation for Jv |br| Default = ``true``"
   ``dealiasing``,"``true`` / ``false``",""

.. _sec:neknekpars:

NekNek Parameters
"""""""""""""""""""""
.. _tab:neknekparams:

.. csv-table:: ``NEKNEK`` settings in the ``.par`` file
   :widths: 20,20,60
   :class: tall
   :header: Key, Value(s), Description/Note(s)/Default Value

   ``boundaryEXTOrder``,``<int>``, "Boundary extrapolation order |br| Default = ``1``. >1 may require additional corrector steps"
   ``multirateTimeStepping``,"``true, false`` |br| ``+ correctorSteps=<int>``","Default = ``false`` |br| Outer corrector steps. Default is ``0``. Note: ``boundaryEXTOrder`` > 1 requires ``correctorSteps`` > 0 for stability"


.. _udf_file:

User-Defined Host File (.udf)
-----------------------------

A *UDF* (user-defined function) is an entry point *NekRS* calls during setup and time stepping, letting a case customize behavior without touching core code.
Typical uses include setting initial and boundary conditions, defining custom materials, models, and source terms, and performing sampling and logging for post-processing.
The available entry-point functions and their typical uses are as follows.
All UDF functions are **optional**.
If a function is not provided, NekRS uses a default no-op version.

.. _okl_block:

OKL block
"""""""""

The ``.udf`` usually contains a preprocessor guard ``#ifdef __okl__`` that encloses all :term:`OKL` kernels and device functions compiled for the selected :term:`OCCA` backend.
For an overview of the OKL language, see the `OKL language guide <https://libocca.org/#/okl/introduction>`_.
NekRS provides default device functions for boundary conditions.
Details of the ``udfDirichlet`` and ``udfNeumann`` functions used for Dirichlet and Neumann boundary conditions, respectively, can be found in :ref:`boundary_conditions`.

.. code-block:: cpp

  #ifdef __okl__

  void udfDirichlet(bcData *bc)
  {
    if(isField("fluid velocity")) {
      bc->uxFluid = 1.0;
      bc->uyFluid = 0.0;
      bc->uzFluid = 0.0;
    }
    else if (isField("fluid pressure")) {
      bc->pFluid = 0.0;
    }
    else if (isField("scalar temperature")) {
      bc->sScalar = 0.0;
    }
  }

  void udfNeumann(bcData *bc)
  {
    if(isField("fluid velocity")) {
      bc->tr1 = 0.0;
      bc->tr2 = 0.0;
    }
    else if (isField("scalar temperature")) {
      bc->fluxScalar = 0.0;
    }
  }

  #endif

Functions marked with the decorate ``@kernel`` are compiled as device kernels and can be launched from UDF host code directly as a regular function.

.. code-block:: cpp

   // -------- Device side (inside the OKL block) --------
   #ifdef __okl__

   @kernel void my_exact(const dlong Ntotal,
                         @ restrict const dfloat *X,
                         @ restrict dfloat *U)
   {
     for (dlong n = 0; n < Ntotal; ++n; @tile(p_blockSize, @outer, @inner)) {
       if (n < Ntotal) {
         U[n] = sin(X[n]);
       }
     }
   }

   #endif

   // -------- Host side (UDF code) --------
   {
     auto mesh = nrs->meshV;
     deviceMemory<dfloat> o_tmp(mesh->Nlocal);
     my_exact(o_tmp.size(), mesh->o_x, o_tmp); // o_tmp = sin(x)
   }

.. tip::

   If the user-defined functions are sufficiently large, it is conventional practice to write them in a ``.oudf`` file which is included within the ``ifdef`` block instead of the functions in the ``.udf`` file, as follows:

  .. code-block:: c++

     #ifdef __okl__

     #include "case.oudf"

     #endif

.. tip::

   Many common operations are available in ``platform->linAlg`` and ``opSEM`` (e.g., fills, axpby, dot products, norms, gradient, divergence).
   Prefer these utilities over writing custom kernels when possible.
   See :ref:`linalg` and :ref:`opsem` for details.

.. _udf_setup0:

UDF_Setup0
""""""""""

``UDF_Setup0`` is called **once** at startup, before *NekRS* initializes the mesh, solvers, or solution arrays.
It receives the *NekRS* :term:`MPI` communicator (``comm``) and the user options object (``options``).
Because the simulation state is not yet constructed, this function is mainly for reading or adjusting user settings.

.. code-block:: c++

   static dfloat P_GAMMA;

   void UDF_Setup0(MPI_Comm comm, setupAide &options)
   {
     platform->par->extract("casedata","p_gamma",P_GAMMA);
   }

   void UDF_LoadKernels(deviceKernelProperties& kernelInfo)
   {
     kernelInfo.define("p_GAMMA") = P_GAMMA;
   }

In this example, ``UDF_Setup0`` reads the ``p_gamma`` key from the ``CASEDATA`` user section in the ``.par`` file (see :ref:`sec:user_section`) using the convenience helper ``platform->par->extract``.
The value is stored in a global ``P_GAMMA`` and then exported as the device macro ``p_GAMMA`` in ``UDF_LoadKernels`` for JIT-compiled kernels.

.. warning::

   Do **not** modify parameters used to compile legacy *Nek5000* in ``UDF_Setup0`` such as ``polynomialOrder``.


UDF_LoadKernels
"""""""""""""""

As shown in the example above, ``UDF_LoadKernels`` is primarily used to attach preprocessor macros (global defines) for device kernels.
It takes ``deviceKernelProperties& kernelInfo`` which holds compilation metadata.
Using ``kernelInfo.define("NAME") = value`` to expose constants to :term:`OKL`.

.. tip::

   For constants passed to device code (via ``UDF_LoadKernels``), it’s recommended to use a ``p_`` prefix (e.g., ``p_gamma``, ``p_Pr``, ``p_Re``) for clarity and consistency.


UDF_Setup
"""""""""

``UDF_Setup`` is called once at the beginning of the simulation *after* *NekRS* initializing the mesh, solution fields, material properties, and boundary mappings.
It is typically the ``.udf`` function users will use most to overwrite defaults and customize settings.
Various operations are performed within this routine, including, but not limited to:

* Assign initial conditions (see :ref:`initial_conditions`).
* Mesh manipulation (see :ref:`tutorial_rans` tutorial).
* Register function pointers for user-defined spatially varying material properties (see :ref:`properties`).
* Register function pointers for user-defined source terms (see :ref:`source_terms`).
* Initialize and set up RANS turbulence models (see :ref:`ktau_model`)
* Initialize and set up the Low-Mach compressible model (see :ref:`lowmach_model`)
* Initialize solution recycling routines and arrays (see :ref:`recycling`)
* Allocate ``bc->o_usrwrk`` for user-defined boundary data (see *TODO*)
* Initialize time-averaging routines and buffers (see *TODO*)


UDF_ExecuteStep
"""""""""""""""

``UDF_ExecuteStep`` offers the most flexibility. 
It is called once just before time marching begins, and then once **per time step**.
The routine receives the current time (``double t``) and the step index (``int tstep``).
Typical operations include:

* Run time-averaging updates (see *TODO*)
* Invoke solution recycling logic (see :ref:`recycling`).
* Post-processing tasks, such as:

  * Extracting data over a line (see :ref:`extract_line`).
  * Writing custom field files (*TODO*).


.. _usr_file:

Legacy Nek5000 User File (.usr)
--------------------------------

NekRS includes an optional interface to *Nek5000* for using legacy Fortran-77 user routines.
If ``<case>.usr`` is present in the case directory, the file is compiled and linked, though not all *Nek5000* user subroutines are invoked by NekRS.

Many one-time setup routines are still honored so existing *Nek5000* settings can be carried over.
Users can continually use subroutines ``usrdat0``, ``usrdat``, ``usrdat2``, ``usrdat3``, ``usrsetvert`` and ``useric`` to modify mesh geometry, tag boundary surfaces, adjust connectivity and set initial conditions.
Compute intensive subroutines such as ``userbc``, ``userf``, ``userq`` and ``uservp`` are **not** used by *NekRS*. Users need to convert them into UDF or OKL implementation.

The ``userchk`` is **not** called automatically. 
If needed, call it manually from UDF (e.g., ``UDF_ExecuteStep``) for post-processing or other setup tasks, for example:

.. code-block:: c++

   void UDF_ExecuteStep(double time, int tstep)
   {
     if(nrs->checkpointStep) { // occasionally call when a checkpoint step
        nrs->copyToNek(time, tstep); // push device field to Nek5000 arrays
        nek::userchk();              // call userchk in .usr
        nrs->copyFromNek(time);      // only if userchk modify solutions
     }
   }

.. note::

   - ``nrs->copyToNek`` copies all solution fields from :term:`OCCA` device arrays to *Nek5000* (Fortran) host arrays.
     Use it only when ``nek::userchk`` needs current solutions.
   - This transfer is expensive.
     Wrap the call under a suitable condition to avoid excessive overhead.
   - Call ``nrs->copyFromNek`` only if the legacy routine modifies solution data that must be synchronized back to the device.

For background on Nek5000, see the `Nek5000 documentation <https://nek5000.github.io/NekDoc/>`_.
Details on these initialization routines are documented `here <https://nek5000.github.io/NekDoc/problem_setup/usr_file.html#initialization-routines>`_.
To migrate a case, follow the *TODO* tutorial for converting a *Nek5000* case to *NekRS*.
In this section, we focus on the interface for referencing variables between ``.udf`` and ``.usr``.

Legacy Data Interface
"""""""""""""""""""""

NekRS does not “send” values or arrays to *Nek5000*.
Instead, it shares them by **registering pointers**.
On the Fortran side, any variable or array you expose must have a stable address, so put it in a ``COMMON`` block or give it the ``SAVE`` attribute.
Use the Fortran routine ``nekrs_registerPtr`` to register variables/arrays so NekRS can reference them by a string key.

Here is an example ``.usr`` file:

.. code-block:: fortran

   c-----------------------------------------------------------------------
         subroutine usrdat0
         real gamma
         save gamma

         gamma = 1.4

         call nekrs_registerPtr('gamma', gamma)

         return
         end
   c-----------------------------------------------------------------------
         subroutine userchk()
         include 'SIZE'
         include 'TOTAL'

         common /exact/ uexact(lx1,ly1,lz1,lelt*3),
        &               texact(lx1,ly1,lz1,lelt)

         real uexact, texact

         call nekrs_registerPtr('uexact', uexact(1,1,1,1,1))
         call nekrs_registerPtr('vexact', uexact(1,1,1,1,2))
         call nekrs_registerPtr('wexact', uexact(1,1,1,1,3))
         call nekrs_registerPtr('texact', texact)

         ! update uexact/texact here

         return
         end

In this example, ``COMMON /exact/`` block holds two arrays store exact solutions (e.g., velocity and temperature).
We register the **first-element addresses** of each velocity components and of temperature with ``nekrs_registerPtr`` inside ``userchk``.
The scalar ``gamma`` is registered in ``usrdat0`` and kept alive with ``SAVE``.
Any data you register must be in a ``COMMON`` block or ``SAVE`` so its address remains valid.

These pointers can then be looked up in the ``.udf`` and used to exchange data between *NekRS* and *Nek5000*. Example usage in ``.udf``:

.. code-block:: c++

   static dfloat gamma;

   void UDF_Setup0(MPI_Comm comm, setupAide &options)
   {
     gamma = *nek::ptr<double>("gamma");
   }

   void UDF_LoadKernels(deviceKernelProperties& kernelInfo)
   {
     kernelInfo.define("p_GAMMA") = gamma;
   }

   void UDF_ExecuteStep(double time, int tstep)
   {
     if (nrs->lastStep) {
       auto mesh = nrs->meshV;
       auto Nlocal = mesh->Nlocal;

       nek::userchk(); // compute exact solutions in .usr

       // Host pointers exported from usr
       auto *u_ex = nek::ptr<double>("uexact");
       auto *v_ex = nek::ptr<double>("vexact");
       auto *w_ex = nek::ptr<double>("wexact");

       // Range constructor (copy values into std::vector)
       std::vector<double> t_ex(nek::ptr<double>("texact"),
                                nek::ptr<double>("texact") + Nlocal);

       // Host buffer
       std::vector<dfloat> uexact(Nlocal), uexact(Nlocal), wexact(Nlocal), texact(Nlocal);

       // Convert from double to dfloat
       for (int i = 0; i < Nlocal; i++) {
         u_exact[i] = static_cast<dfloat>(u_ex[i]);
         v_exact[i] = static_cast<dfloat>(v_ex[i]);
         w_exact[i] = static_cast<dfloat>(w_ex[i]);
         t_exact[i] = static_cast<dfloat>(t_ex[i]);
       }

       // Device buffer
       auto o_uexact = platform->device.malloc<dfloat>(nrs->fluid->fieldOffsetSum);
       auto o_texact = platform->device.malloc<dfloat>(Nlocal);

       // copyFrom(src pointer, count, dest offset, src offset)
       o_uexact.copyFrom(uexact.data(), Nlocal, 0 * nrs->fluid->fieldOffset, 0);
       o_uexact.copyFrom(vexact.data(), Nlocal, 1 * nrs->fluid->fieldOffset, 0);
       o_uexact.copyFrom(wexact.data(), Nlocal, 2 * nrs->fluid->fieldOffset, 0);
       o_texact.copyFrom(texact.data(), Nlocal);

       //compute error here...
     }
   }

As shown, ``nek::ptr`` returns the registered pointers by the string keys defined in the ``.usr`` file.
In ``UDF_Setup0``, the value referenced by ``gamma`` is read into a C++ static and later exported as a device macro in ``UDF_LoadKernels``.

For the arrays (``uexact``, ``vexact``, ``wexact``, ``texact``), we show two ways, viz., use a raw host pointer from ``nek::ptr<double>(...)`` for velocity components or use a range constructor to make a copy into ``std::vector`` for temperature.
Then, all are converted from ``double`` to ``dfloat`` before copying to :term:`OCCA` arrays.

.. note::

   *NekRS* compiles Fortran ``real`` as ``real*8`` (i.e., C++ ``double``).
   In *NekRS*, ``dfloat`` may be ``double`` (default) or ``float`` (FP32 mode).
   Converting on the host as shown works for both settings.
   If you always run in FP64, you can skip the cast.
   Alternatively, you can convert on device via ``platform->copyDoubleToDfloatKernel``.

.. note::

   **Nek5000/NekRS sizes and offsets**

   - ``mesh->Nlocal``: number of local points on this rank (e.g., ``lx1*ly1*lz1*nelv``; fluid+solid mesh uses ``nelt``).

   - ``fieldOffset``: padded device stride (alignment + max local size).
     It is **not** equal to ``lx1*ly1*lz1*lelv``.

   - ``fieldOffsetSum``: sum of component strides.

     -- For velocity: ``fieldOffsetSum = 3 * fieldOffset``.

     -- For scalar: ``fieldOffsetSum = nrs->Nscalar * fieldOffset``.

   Use ``fieldOffset``/``fieldOffsetSum`` for declaring multi-component device arrays and indexing.
   Use ``Nlocal`` for loop lengths and buffer sizes.

.. _re2_file:

Mesh File (.re2)
----------------

The ``.re2`` mesh format originates from *Nek5000*, and NekRS uses the **same** binary format, so meshes made for *Nek5000* remain compatible with *NekRS*.
This section summarizes the ``.re2`` layout and conventions.
See :ref:`meshing` for mesh creation and conversion workflows.

You can obtain ``.re2`` by:

* Porting from an existing *Nek5000* case,
* Using a :ref:`mesh converter<meshing_convert>` from other format, or
* Generating it with :ref:`Nek5000 meshing tools <meshing_nek5000_tools>`.

There are three main limitations for the *NekRS* mesh:

* **3D hex only**: meshes must be hexahedral and three-dimensional.
* **Contiguous boundary IDs**: face boundary IDs must be 1, 2, 3, … without gaps.
* **Element types**: the main supported types are HEX8 and HEX20.

.. tip::

   Lower-dimensional (2D/1D) setups can run on a 3D mesh by enforcing zero-gradient boundary conditions on all solution variables in the out-of-interest directions.
   Material properties and forcing terms must likewise be constant (or zero-gradient) in those directions.
   See :ref:`mesh_setup_connectivity` for examples.

The header of a ``.re2`` file is a single 80-character line. 
While minor variants exist across versions, they all encode: ``(version) (total elements) (dimension) (fluid elements)``.
For example, the ``ethier.re2`` has 32 elements:

.. code-block:: none

   #v003       32  3       32 this is the hdr

.. note::

   In conjugate heat transfer (:term:`CHT`) cases, one mesh file contains both fluid and solid regions, with the fluid mesh stored first, followed by the solid mesh.
   Consequently, the total element count exceeds the fluid count.

   ``conj_ht`` example (96 fluid + 96 solid = 192 total, 3D):

   .. code-block:: none

      #v002      192  3       96  this is the hdr

   In :term:`CHT` runs, make sure you use the correct ``mesh`` in your ``.udf``.
   Typically, reference the mesh object under the corresponding solver:

   - ``nrs->fluid->mesh`` for fluid solver mesh
   - ``nrs->scalar->mesh("temperature")`` or, if it's the first scalar,
     ``nrs->scalar->mesh[0]`` for temperature mesh

   See the :ref:`conjugate_heat_transfer` tutorial for details, and :ref:`Conjugate Heat Transfer Meshing <meshing_cht>` for creating :term:`CHT` meshes.

After the header and an endianness check, a ``.re2`` file stores, for each element, the coordinates of its 8 vertices (HEX8).
If curvature is present, upto 12 mid-edge nodes are added, yielding HEX20.
Some legacy curved primitives (e.g., cylinders or spheres) use special curve records for higher-order accuracy, but in general edges are represented by second-order polynomials.
If needed, users can recover exact geometry in ``usrdat2`` or by reading all higher-order grid points from a checkpoint file.

Boundary faces are tagged for boundary conditions. 
The fluid mesh is always present, and scalars may have their own tag sets; in :term:`CHT` cases, fluid and solid regions each have separate boundary maps.
These tags are later mapped to *Nek5000* arrays ``CBC(f,e,ifield)`` and to boundary IDs ``boundaryID(f,e)`` (and ``boundaryIDt(f,e)`` for :term:`CHT`).


.. _sess_file:

NekNek Session File (.sess)
---------------------------

NekNek allows coupling of multiple overlapping subdomains, relaxing the need for conformal meshes.
Each *NekRS* instance uses its own `.par` file.
In the ``.sess`` file, each line lists one instance as:
``<path-to-par-file>:<num-mpi-ranks>;``

Paths may be absolute or relative, and the sum of ranks over all instances must equal the total MPI ranks requested. Here is the ``eddyNekNek.sess`` example:

.. code-block:: none

   inside/inside:1;
   outside/outside:1;

.. tip::

   Using a subfolder per case is optional but recommended. 
   On HPC systems, try to size each session in units of a node.
   For example, if a node has 4 GPUs, it’s often best to make each session’s MPI ranks a multiple of 4.


.. _upd_file:

Trigger File (nekrs.upd)
------------------------

The trigger file lets you change selected parameters **at runtime** so you can adjust a simulation without killing the job.
Before launching, set a catchable signal in ``NEKRS_SIGNUM_UPD`` (see :ref:`nekrs_signal`).
At runtime, write the requested options in ``nekrs.upd`` under the case directory.
Accepted keys include:

.. code-block:: ini

   checkpoint = true

   [GENERAL]
   endTime = 10.0
   numSteps = 10000
   writeinterval = 1.0

When *NekRS* receives the configured signal, it reloads ``nekrs.upd`` and applies the changes on the next timestep. You can trigger this multiple times during a run.

.. _file_extensions:

NekRS Files & Extensions
---------------------------

.. csv-table:: Case Files & Extensions
   :header: "Extension","Name","Required?","Notes / Typical usage"
   :widths: 20,30,20,30
   :quote: "
   :delim: ,

   ".sess","NekNek sessions file","Yes (Only for NekNek)","See :ref:`sess_file`"
   ".par","Main parameter file","Yes","See :ref:`par_file`"
   ".udf","User-defined functions","Yes","See :ref:`udf_file`"
   ".oudf","Extra OKL kernels file","No","See :ref:`okl_block`"
   ".upd","Trigger file","No","See :ref:`upd_file`"

   ".usr","Legacy Nek5000 user file","No","See :ref:`usr_file`"
   ".re2","Mesh file","Yes","See :ref:`re2_file`"
   ".co2","Legacy connectivity file","No","See `Nek5000 co2 example <https://nek5000.github.io/NekDoc/tutorials/multi_rans.html?highlight=co2#generating-connectivity-file-co2>`_"

   ".yaml","YAML config file","No","Used for ``nekAscent.hpp`` in-situ visualization"

.. csv-table:: Output Files & Extensions
   :header: "Extension","Name","Notes / Typical usage"
   :widths: 20,40,40
   :quote: "
   :delim: ,

   "\*0.f0000","Nek format checkpoint file","0-based index. See :ref:`qstart_vis`"
   ".nek5000","Nek format checkpoint metadata file", "See :ref:`qstart_vis`"
   ".bp/","ADIOS2 format checkpoint (folder)",""
   ".log.*","Log files","From ``nrsbmpi``, suffixed by rank count and rollover to ``.log1``"
   ".vtu/.vtk","VTK files","Mainly for particle data"
