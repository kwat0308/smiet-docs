A User's Guide to Template Synthesis
====================================

Here we give a general overview of how the template synthesis method works. It is not meant as an in-depth
description of the algorithm, but rather as a guide to help you understand how to use the package. In
:doc:`ts_algorithm` we describe the algorithm on a high-level. For more details, please refer to the
publications mentioned on the homepage of the package.

Using Template Synthesis
------------------------

The simplest way to use template synthesis is with the :code:`smiet.core.SMIETRun` class, which handles all necessary steps (loading origin showers, loading target showers, synthesising them, and extracting relevant quantities). More details can be found in :doc:`ts_core`. 

If you need to have more control over the synthesis process, you can also use the lower-level classes directly, as described below.

Origin Showers
--------------

In order to use template synthesis, you can either:
- download the origin shower library (~ 80 GB when unpacked) that contains the pre-simulated origin showers on the vx(vxB) arm, or
- generate your own origin showers.

Some guidance and general tips on how to do this can be found in :doc:`ts_origin_showers`.

Synthesising Target Showers
---------------------------
Once you have the origin showers, you can use them to synthesise traces from  target showers. This is done via `TemplateSynthesis` class. More details are given in :doc:`ts_workflow`.

.. toctree::
    ts_algorithm
    ts_core
    ts_origin_showers
    ts_workflow
    ts_vxvxB_to_starshape
    ts_treatment_of_vxB
