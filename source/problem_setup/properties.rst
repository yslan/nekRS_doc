.. _properties:

Physical properties
===================

Constant Properties
-------------------

Constant physical properties, including transport and diffusion coefficients, for all equations are specified in the :ref:`Parameter file <par_file>` under respective :ref:`field sections <sec:field_settings>`.
Consider the following template for a :ref:`non-dimensional <nondimensional_eqs>` case setup:

.. code-block::

   [FLUID VELOCITY]
   density = 1.0
   #rho = 1.0

   viscosity = 1./1000.0
   #mu = 1./1000.0
   #mu = -1000.0                   #Reynolds number

   residualTol = 1e-6

   [SCALAR FOO]
   transportCoeff = 1.0 

   diffusionCoeff = 1.0/500.0
   #diffusionCoeff = -500.0       #Peclet number

   residualTol = 1e-6

``density`` or ``rho`` key is used to assign the fluid density :math:`\rho` and ``viscosity`` or ``mu`` key assigns the fluid dynamic viscosity :math:`\mu`.
``transportCoeff`` and ``diffusionCoeff`` assign the transport coefficient, :math:`\rho`, and diffusion coefficient, :math:`\lambda` for a general passive scalar equation, respectively.
For the case where ``FOO`` is temperature field, the transport coefficient is the volumetric heat capacity, :math:`\rho c_p`, and diffusion coefficient is the thermal conductivity, :math:`k`.

.. note::

  Considering the problem is non-dimensionalized such that the characteristic length and velocity scales are unity, the Reynolds number for the problem is :math:`Re=1/\mu`.
  Following the legacy convention in `Nek5000 <https://nek5000.github.io/NekDoc/problem_setup/case_files.html#sec-velpars>`_ the user can also specify -ve ``mu`` as shown above for non-dimensional setup, which is interpreted as :math:`Re` specification.
  Similarly, -ve ``diffusionCoeff`` is interpreted as the Peclet number for the :ref:`non-dimensional scalar equation <intro_energy_nondim>`.

Conjugate Heat Transfer Setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For :ref:`conjugate heat transfer <conjugate_heat_transfer>` cases *NekRS* provides a convenient way of defining both fluid and solid properties for the temperature equation in the :ref:`Parameter file <par_file>` as shown,

.. code-block:: 

   [FLUID VELOCITY]
   boundaryTypeMap = ... , ... , ...
   density = 1.0
   viscosity = 1./1000.0
   residualTol = 1e-6

   [SCALAR TEMPERATURE]
   boundaryTypeMap = ... , ... , ... , ... , ...

   mesh = fluid+solid

   transportCoeff = 1.0 
   diffusionCoeff = 1.0/500.0       #non-dimensional fluid thermal conductivity

   transportCoeffSolid = 1.0
   diffusionCoeffSolid = 1.0/5.0    #non-dimensional solid thermal conductivity

   residualTol = 1e-6

The ``mesh`` key value ``fluid+solid`` informs *NekRS* that this is a conjugate heat transfer case and that the temperature equation is also being solved in the solid part of the domain.
Subsequently, the ``transportCoeffSolid`` and ``diffusionCoeffSolid`` keys are used to assign the material properties in the solid domain.

.. _variable_properties:

Variable Properties
-------------------

Spatially and/or temporally varying transport and diffusion properties can be specified in the ``.udf`` file.
To set these up, begin with assigning a user function pointer to the internal *NekRS* member pointer ``nrs->userProperties`` as follows

.. code-block:: c++

   void UDF_Setup()
   {
      nrs->userProperties = &uservp;
   }

This instructs *NekRS* to look for the function ``uservp`` in ``.udf`` file for property specification.
In ``uservp`` the properties must be populated in the corresponding ``o_prop`` field property arrays which hold both the diffusion and transport coefficients for all :term:`GLL` points.
The user will need to write a custom kernel function in order to populate these arrays.
A template example is as follows:

.. code-block:: c++

   #ifdef __okl__
     @kernel void fillProp(const dlong Nelements,
                           const dfloat Re,
                           const dfloat Pe,
                           @ restrict dfloat* MUE,
                           @ restrict dfloat* RHO,
                           @ restrict dfloat* K,
                           @ restrict dfloat* RHOCP)
     {
        for (dlong e = 0; e < Nelements; ++e; @outer(0)) {
          for (int n = 0; n < p_Np; ++n; @inner(0)) {
             const int id = e * p_Np + n;

             MUE[id] = 1./Re;
             K[id] = 1./Pe;

             RHO[id] = 1.0;
             RHOCP[id] = 1.0;
          }
        }
     }
   #endif

   static int updateProperties = 1;

   void uservp(double time)
   {
      auto& fluid = nrs->fluid;
      auto& scalar = nrs->scalar;

      if(updateProperties) {
        const dfloat Re = 1000.0;     //Reynolds number
        const dfloat Pe = 500.0;      //Peclet number

        auto o_mue = fluid->o_prop.slice(0 * fluid->fieldOffset);
        auto o_rho = fluid->o_prop.slice(1 * fluid->fieldOffset);

        auto o_k = scalar->o_prop.slice(0 * scalar->fieldOffset());
        auto o_rhocp = scalar->o_prop.slice(1 * scalar->fieldOffset());

        fillProp(fluid->mesh->Nelements,
                 Re,
                 Pe,
                 o_mue,
                 o_rho,
                o_k,
                o_rhocp);

        updateProperties = 0;
      }
   }

The above example illustrates how the properties are filled in the ``o_prop`` arrays for fluid and temperature (or any scalar) fields.
The fluid property array is referenced using the ``nrs->fluid`` object while the scalar property with the ``nrs->scalar`` object.
Note that the diffusion coefficients and transport coefficients are stored contiguously in the respective ``o_prop`` :term:`OCCA` arrays.
As shown above, the pointer to the coefficient arrays can be conveniently isolated using the ``slice`` operation on ``o_prop``.
The ``slice`` function takes an argument to specify the memory location in :term:`OCCA` array.
The extent or size of each coefficient array is given by ``fieldOffset`` (equal to total number of :term:`GLL` points on a processor). 
Thus, the pointer to fluid dynamic viscosity is located at the first memory location in ``fluid->o_prop(0 * fluid->fieldOffset)`` and the fluid density is located at ``fluid->fieldOffset``, i.e., ``fluid->o_prop.slice(1 * fluid->fieldOffset)``.
Similar slicing operations can pe performed to isolate scalar diffusion and transport coefficient arrays, as shown above.

The ``fillProp`` custom kernel above provides a template example for how to fill the property arrays. 
Although the kernel specifies constant tranport coefficient and diffusion coefficient value for both fluid and scalar properties, it can be customized to include spatial dependence based on target application.

Since the properties are not temporally varying in the above example, the relevant code in ``uservp`` is placed in an ``if(updateProperties)`` condition block which populates the property arrays only once in the simulation, avoiding unnecessary repetitive operations.
It is important to note that *NekRS* will call ``uservp`` at the beginning of each time step in the simulation.
The ``time`` parameter is passed to the ``uservp`` function to provide the current time step for the user, in case temporally varying properties need to be specified. 

.. note::

   If variable properties are being assigned in ``.udf`` file, the corresponding keys in ``.par`` file are optional.
   Any constant value assigned in ``.par`` file will be over-written by user-defined kernel in ``uservp``.

   .. code-block::

      [FLUID VELOCITY]
      residualTol = 1e-6
      # density = 1.0        #optional for variable property specification
      # mu = 1./1000.0       #optional for variable property specification

      [SCALAR TEMPERATURE]
      residualTol = 1e-6
      # transportCoeff = 1.0        #optional for variable property specification
      # diffusionCoeff = 1.0/500.0  #optional for variable property specification
    
.. warning::

   If the fluid dynamic viscosity is spatially varying, the ``equation`` key must be included in ``PROBLEMTYPE`` section in ``.par`` file to specify *NekRS* to use full stress formulation by adding the ``variableViscosity`` value to the key:

   .. code-block::

      [PROBLEMTYPE]

      equation = navierStokes + variableViscosity

.. _properties_cht:

Conjugate Heat Transfer Setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As for the constant property case, setting :ref:`conjugate heat transfer <conjugate_heat_transfer>` case with variable properties requires special consideration in ``.udf`` file.
The following template illustrates the setup for a typical :term:`CHT` case.

.. code-block:: c++

   static int updateProperties = 1;

   #ifdef __okl__
                           
    @kernel void cFill(const dlong Nelements,
                   const dfloat CONST1,
                   const dfloat CONST2,
                   @ restrict const dlong *eInfo,
                   @ restrict dfloat *QVOL)
    {
      for (dlong e = 0; e < Nelements; ++e; @outer(0)) {
        const dlong solid = eInfo[e];
        for (int n = 0; n < p_Np; ++n; @inner(0)) {
          const int id = e * p_Np + n;
          QVOL[id] = CONST1;
          if (solid) {
            QVOL[id] = CONST2;
          }
        }
      }
    }
   #endif

   void uservp(double time)
   {
      auto& fluid = nrs->fluid;
      auto& scalar = nrs->scalar;

      if(updateProperties) {
        const dfloat rho = 1.0;   
        const dfloat mue = 1.0 / 1000.0;

        //fluid
        const auto o_mue = fluid->o_prop.slice(0 * fluid->fieldOffset);
        const auto o_rho = fluid->o_prop.slice(1 * fluid->fieldOffset);
        cFill(fluid->mesh->Nelements, mue, 0, fluid->mesh->o_elementInfo, o_mue);
        cFill(fluid->mesh->Nelements, rho, 0, fluid->mesh->o_elementInfo, o_rho);

        //temperature
        const dfloat rhoCpFluid = rho * 1.0;
        const dfloat conFluid = mue;
        const dfloat rhoCpSolid = rhoCpFluid * 0.1;
        const dfloat conSolid = 10 * conFluid;

        auto mesh = scalar->mesh("temperature"); 

        const auto o_con = scalar->o_diffusionCoeff("temperature");
        const auto o_rhoCp = scalar->o_transportCoeff("temperature");

        cFill(mesh->Nelements, conFluid, conSolid, mesh->o_elementInfo, o_con);
        cFill(mesh->Nelements, rhoCpFluid, rhoCpSolid, mesh->o_elementInfo, o_rhoCp);
      }
      updateProperties = 0;
   }

   void UDF_Setup()
   {
      nrs->userProperties = &uservp;
   }

In the above example a common custom kernel ``cfill`` is written to fill out all property arrays for fluid and temperature fields.
The essential difference between the :term:`CHT` case setup and a non-CHT case is the use of ``mesh->o_elementInfo`` in the custom kernel.
This array marks the elements in the ``mesh`` that reside in the solid part of the domain and, therefore, can be easily used to differentiate fluid and solid elements in the ``cfill`` kernel as shown above. 
Note that when the fluid mesh object is passed to cfill, i.e., ``fluid->mesh->o_elementInfo``, there are no solid elements in this mesh object and hence the ``if(solid)`` condition block in ``cfill`` kernel is not executed.
