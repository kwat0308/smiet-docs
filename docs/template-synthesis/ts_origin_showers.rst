Origin Showers
==============
Template synthesis relies on pre-simulated origin showers to create templates from which
we can synthesise new showers. These origin showers are simulated with CoREAS in a special
mode which splits the radio emission into multiple atmospheric depth bins.

Currently, origin showers are generated on the :math:`\vec{v} \times \vec{v} \times \vec{B}` axis.
This is done to reduce the computational cost of the origin shower simulations, while still
capturing the relevant features of the radio emission. The synthesised traces can then be 
rotated to the desired star-shape antenna positions in the shower plane. If star-shape origin showers are 
necessary, then please refer to the section on generating your own origin showers below.

Origin Shower Library
=====================
The main limitation of the template synthesis method is that the performance degrades
when the difference between the :math:`X_\mathrm{max}` and zenith angle of the target shower and
the origin shower increases. To mitigate this, we have created a library of origin showers
spanning a grid of zenith angles and :math:`X_\mathrm{max}` values. This library can be used
to select the best matching origin shower for a given target shower. Furthermore, it can be readily applied with the interpolated synthesis approach, which is prefered to remove the asymmetric bias inherent to the template synthesis method.

The library contains showers with zenith angles from 0 to 60 degrees in steps of 3 degrees, and
:math:`X_\mathrm{max}` values from 600 to 1200 g/cm\ :sup:`2` in steps of 100 g/cm\ :sup:`2`.
Each configuration is simulated with three different random seeds to account for shower-to-shower
fluctuations. This results in a total of 441 origin showers. The mass composition and primary energy
are fixed to proton and 10\ :sup:`17` eV, respectively, as the radio emission does not depend strongly
on these parameters. The thinning level is set to 10\ :sup:`-7`.

Currently, origin showers are simulated with the LOFAR configuration, with the magnetic field from the 
LOFAR site with a US standard atmosphere. More configurations will be added in the future.

The origin shower library can be downloaded from the following link.

.. warning::
    The zipped library is 62GB in size, and about 160 GB in size after unzipping. Please ensure you have enough disk space
    before downloading.

We also provide a module to generate your own origin shower library. This is described in
the :doc:`corsika <../corsika/corsika_index>` module.

Generating the origin showers
=============================

Alternatively, you can generate your own origin showers. This gives more control to the user to 
setup their ideal antenna layout, primary particle type, energy, thinning level, and other
simulation parameters.
In order to generate the origin showers, we need to run CoREAS with the correct settings. This can
easily be achieved using the tools provided in the :doc:`corsika <../corsika/corsika_index>` module.

Currently we recommend to run simulations with about 20 antennas on the
:math:`\vec{v} \times \vec{v} \times \vec{B}` axis. This is a good number to apply interpolation on,
later on. However, with 250 slices, this results in 5000 configured observers in CoREAS. From our
testing, this number is way too high to be run on a single node (runtimes exceeded the limit of 10
days with :math:`10^{-7}` thinning).

.. tip::
    If you really need to run one node, we suggest running 4-6 antennas. This should keep the
    runtimes within 48-72 hours, which can still be acceptable for most computing clusters. Lowering
    the thinning will also help of course.

To resolve this issue, you can either use MPI (recommended, if you computing environment supports it) or
split the simulations into multiple runs with smaller antenna sets. There should also be a mode in CORSIKA
to mimic MPI by using a set of scripts that do this splitting automatically, but we have no experience
with using that module. The file generation functions from the :doc:`corsika <../corsika/corsika_index>`
module configure the settings required for MPI automatically.