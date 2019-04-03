.. include:: ../../../replaces.txt
.. _newgeodomain-sec:

**************************
Adding new meshing domains
**************************

.. toctree::


As explained in chapter FIXME |CMB| provides meshing capabilities interfacing
GMSH via data files produced from pre-defined templates. The template files
describe parameterized geometric models. When mesh is generated the
parameters are substituted in Matlab producing input files for GMSH. The
generated mesh is read back from mesh file and presented to user as a Mesh
object.

The whole machinery of templates handling and calling GMSH is hidden behind
convenient object oriented interface described in section FIXME. Here we take a
closer look at this machinery by following a step by step explanation how a new
geometric model can be added to |CMB|. The steps involve:

* Creation of non-templatized geometric model for GMSH.
* Implementation of a class derived from mp.Geom to wrap the geometric model.
* Templatization of GMSH input file and linking it with Matlab class.
* Introduction of more advanced features as subdomain and meshing control variables.

These steps will be illustrated in details on the example of geometric model of
a quarter of notched rectangle domain shown in :numref:`fig_nrq_domain`.

.. _fig_nrq_domain:
.. figure:: tutorial/notchedrectquarter.png

   Geometric domain of a quarter of notched rectangle.

Creating GMSH geometry input file
=================================

The starting point in adding new meshing domain is to prepare a valid GMSH geometry
file describing the desired geometry. This file will will be then turned into template
file processed by Matlab, but first one has to make sure that there is no error in
geometry definition and GMSH is able to produce a mesh from this file.

To complete this step some knowledge of GMSH scripting language is required. In GMSH
geometries are described using boundary representation (B-Rep). The model is built
by successively defining primitives of increased complexity and dimension, starting
from points, curves, surface and volumes. 

When looking at :numref:`fig_nrq_domain` we can see that it is parameterised by
three parameters :math:`W,H` and :math:`R`. It is advantageous to introduce these
parameters right from the start in the GMSH geometry file, for the moment giving them
some fixed value. When creating B-Rep representation of our geometry we should already
envisage how it should be decomposed into geometric elements in order to make convenient
application of attributes such as boundary conditions, loads or material properties.
Having this in mind the arc is split into two pieces by a point :math:`(x_a, y_a)`. 
To make this splitting flexible we introduce additional parameter 
:math:`t \in [0, 1]` that describes the position
of the point on arc via formula

.. math::
   :nowrap:

   \begin{align}
      x_a   &= W - R \cos(t \frac{\pi}{2} )\\
      y_a   &= R \sin(t \frac{\pi}{2}) 
   \end{align}

The complete GMSH input file is shown in listing :numref:`lst_notchedrectquarter`.
Additional parameter :math:`lc`, defined in the first line, is the desired edge
length of the generated mesh in the neighbourhood of the given point.

**IMPORTANT** The interface to GMSH implemented in |CMB| assumes that in the
generated mesh at least one physical region is present. Thus, while it is not strictly
necessary for running GMSH, in the last line in :numref:`lst_notchedrectquarter`
a region with name ``d_whole`` is defined. The names assigned to regions are arbitrary
as long as they can be used as Matlab variable names.

.. _lst_notchedrectquarter:
.. literalinclude:: tutorial/notchedrectquarter.geo
   :caption: GMSH geometry input file for domain in :numref:`fig_nrq_domain`

Having the file defined we can load it into GMSH to check if the geometry
is defined properly as illustrated in :numref:`fig_nrq_gmsh_geo`.

.. _fig_nrq_gmsh_geo:
.. figure:: tutorial/notchedrectquarter_gmsh_geo.png
   :scale: 40 %

   GMSH visualisation of geometry defined in :numref:`lst_notchedrectquarter`.

The last step is to run GMSH to generate mesh for the defined geometry
as illustrated in :numref:`fig_nrq_gmsh_mesh`.
   
.. _fig_nrq_gmsh_mesh:
.. figure:: tutorial/notchedrectquarter_gmsh_mesh.png
   :scale: 50 %

   Mesh generated from the file shown in :numref:`lst_notchedrectquarter`.
   

Creating geometry template file 
===============================

Having GMSH input file properly defined we can not turn it into geometry
template file. We just need to copy the file into specific folder and change
its extension to :code`*.tpl`. In |CMB| all geometry template files are store
in folder ``mesher/gmsh/geomodels``. For the subsequent steps we assume that
the code in :numref:`lst_notchedrectquarter` is saved in file
``mesher/gmsh/geomodels/notchedrectquarter.tpl``.


Defining Matlab class for geometric model
=========================================

To nicely handle geometric domain defined in ``notchedrectquarter.lst`` we need
to define a class derived from :code:`mp.GeomModel`. The base class
:code:`mp.GeomModel` provides functionality common to all geometric domains.
It also defines interface than each specific geometry class must obey to
smoothly integrate with the rest of |CMB| package. Here we assume that our
class will be called :code:`NotchedRQGeom` [#f1]_. The definition of this class
shown in :numref:`lst_NotchedRQGeom_v1` is put in file
``package/+mp/+geoms/NotchedRQGeom.m`` alongside other geometric domain
classes.

.. _lst_NotchedRQGeom_v1:
.. literalinclude:: tutorial/NotchedRQGeom_v1.m
   :linenos:
   :language: matlab
   :emphasize-lines: 1,7,11,23,26,29
   :caption: Definition of class :code:`NotchedRQGeom` 

Lines 11-22 define :code:`NotchedRQGeom` constructor. The constructors for
geometric domain classes look virtually the same. The real work to construct an
object is delegated to :code:`setup` method defined at lines 7-8. For a moment
:code:`setup` method is empty, but later we extend it when adding more
functionality to our class. The :code:`legacyID` variable define in line 13 is
the reminiscence of the old design in which each distinct geometric model had
unique numeric ID. The value assigned to :code:`legacyID` is arbitrary with one
condition that it should be greater than 100.

Attention should be paid to line 18. At this line the constructor of base class
is called. The second argument of :code:`mp.GeomModel` constructor is the
geometric dimension of the model. All geometric models are embedded in
3-dimensional real space, but themselves they can be 1D, 2D or 3D objects.

As our :code:`NotchedRQGeom` class is derived from :code:`mp.GeomModel`
abstract base class it has to define the following three methods

* :code:`regions(obj)` - method that return names of physical regions as a cell
  array of strings.  
* :code:`templateName(obj)` - method that returns the name
  of geometry template file.
* :code:`coarsest_lc(obj)` - method that return the real value used as mesh
  density parameter. |CMB| provides some convenience functions to visualize
  geometric domains but this can be done only via meshes generated on them. The
  mesh density parameter returned by :code:`coarsest_lc` method is just an
  educated guess about mesh density that will still capture geometric features
  of the domain but the same time will have the smallest number of elements to
  not induce unnecessary overhead. 
 
Having :code:`NotchedRQGeom` class defined we can use it in the meshing
framework. The example shown in :numref:`lst_Notched_use_v1` produces mesh
shown in :numref:`fig_Notched_mesh_v1`.

.. _lst_Notched_use_v1:
.. literalinclude:: tutorial/demo_Notched_v1.m
   :linenos:
   :language: matlab
   :caption: Example of using :code:`NotchedRQGeom` class.

.. _fig_Notched_mesh_v1:
.. figure:: tutorial/NotchedRQGeom_mesh_v1.png

   Mesh generated and visualized in :numref:`lst_Notched_use_v1`.

Extending :code:`NotchedRQGeom` class
=====================================

At this moment we can notice that the class :code:`NotchedRQGeom` is sort of
a *dumb* one. We can use it to generated mesh in the specific geometry, but we
have no means of controlling the model dimensions nor meshing parameters.  This
is related to the fact that the geometry template file
``notchedrectquarter.lst`` is all the time the same, thus GMSH produces all the
time the same mesh. 

The first step in adding features to :code:`NotchedRQGeom` is to turn
``notchedrectquarter.lst`` into true template file as shown in
:numref:`lst_notched_template_v1`.  In highlighted lines instead of specific
values we put named placeholders obeying the syntax ``<`` *name* ``>``. The
placeholder name can be arbitrary as long as it is a valid Matlab identifier
[#f2]_.

.. _lst_notched_template_v1:
.. literalinclude:: tutorial/notched_template_v1.tpl
   :emphasize-lines: 1-5
   :caption: GMSH input file turned into template.

The second step is to modify definition of :code:`NotchedRQGeom` class by
defining parameters corresponding to placeholders in geometry template file and
respectively fixing :code:`setup` method to properly initialize the parameters
either form constructor arguments or from default values. New version of
:code:`NotchedRQGeom` class is shown in :numref:`lst_NotchedRQGeom_v2` with new
lines highlighted.


.. _lst_NotchedRQGeom_v2:
.. literalinclude:: tutorial/NotchedRQGeom_v2.m
   :language: matlab
   :lines: 1-13, 37
   :emphasize-lines: 4,8-11
   :caption: Fragment of modified definition of class :code:`NotchedRQGeom` with changes highlighted.

With geometry template file and the class augmented in the above way it is now
possible to control both geometry and mesh density as shown in :numref:`lst_Notched_use_v2`.

.. _lst_Notched_use_v2:
.. literalinclude:: tutorial/demo_Notched_v2.m
   :linenos:
   :language: matlab
   :caption: Controling geometry and mesh densit with extended :code:`NotchedRQGeom` class.

.. _fig_Notched_mesh_v2:
.. figure:: tutorial/NotchedRQGeom_mesh_v2.png

   Mesh generated and visualized in :numref:`lst_Notched_use_v2`.

The exercise of extending the class :code:`NotchedRQGeom` clearly shows how
simple is the mechanism for passing data from Matlab level to GMSH.  This
mechanism is based on textual substitution of placeholders in a template file
by values of corresponding Matlab variables. The substitution values come
either from geometric model object, for instance model dimensions,  or form
mesher object, for instance mesh density parameters. The values passed passed
from Matlab to GMSH, can be either scalars or vectors. In case of vector valued
parameters the substitution string is formed by values of vector elements
separated by comas without any other syntactic characters. Thus to properly
handle such substitutions on GMSH side, one has to make sure that proper GMSH
syntax for list literals is used. This is illustrated in
:numref:`lst_gmsh_lists_placeholder`.

.. _lst_gmsh_lists_placeholder:
.. code-block:: none
   :caption: Handling substitution of vector valued variables.

   vecparam = {<vecparam>};

Similar rule applies to the case of string valued parameters.
Further extensions to a geometric model class and template file are case specific and will strongly dependon functionality provided by GMSH scripting language. Below we discuss some extensions that are likely to be relevant to many geometric domains. They are:

* subdivision of domain into subdomains,
* selection of the type of generated elements,
* management of mesh density by local refinements,
* handling boundary regions,
* independent meshing of regions and their combinations.

Subdivision of meshing domain into subdomains
---------------------------------------------

.. _fig_Notched_mesh_v3:
.. figure:: tutorial/NotchedRQGeom_mesh_v3.png

   Meshing domain subdivided into subdomain.

.. rubric:: Footnotes

.. [#f1] While not strictly necessary the naming convention addopted in |CMB| is such that names of classes for geometric models have the ``Geom`` suffix.

.. [#f2] While not strictly necessary the naming convention addopted in |CMB| is such that names of placeholders corresponding to dimension parameters have ``d`` prefix, for instance ``dL``, ``dH``, etc.
