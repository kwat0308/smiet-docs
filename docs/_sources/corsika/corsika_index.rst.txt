.. _corsika_filegeneration:

Tools for CORSIKA/CoREAS
========================

In order to work with template synthesis, you will need to simulate origin showers with
CORSIKA. These are simulation which are set up in CoREAS to split the radio emission in
each antenna into multiple atmospheric depth bins. To easily produce the input files
required for these simulations, we provide some convenience functions in this module.

In particular, we provide the user with two possible options to generate origin showers. 
The first is to generate a single origin shower to test the synthesis procedure. The second approach 
generates a full library of origin showers, mapped as a function of zenith angle and 
:math:`X_\mathrm{max}`. Both approaches are described below.

Generating a Single Origin Shower
---------------------------------

To generate a single origin shower, you can run the two main functions: ``generate_simulation``
and ``write_simulation_to_file``, both contained in the `write_simulation` submodule.
The first one generates the contents of the INP, LIST
and REAS file, using the chosen settings. If you so wish (for example because of the job
submission process on your compute cluster), you can modify the content before writing
it to files. When you are done, you can pass the three arguments to the second function,
which will write them to the files. The names of the files are taken from the simulation
number which is in the INP file.

Generating a Full Library of Origin Showers
-------------------------------------------

The performance of the synthesis degrades with :math:`X_\mathrm{max} - X_\mathrm{max}^\mathrm{origin} > 100`, 
as well as with zenith angles > 8 degrees. To mitigate this, we recommend to use a library of origin showers. 
This is done with the 
``OriginLibraryGenerator`` class in ``generate_origin_library``. This class generates input steering files
(INP, LIST, REAS) for a grid of zenith angles and :math:`X_\mathrm{max}` values, which can then be simulated with CORSIKA/CoREAS.

.. toctree::
   :maxdepth: 3

   file_generation
   origin_shower_library
   package_documentation
