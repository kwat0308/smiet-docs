Transforming synthesised traces on the vxvxB axis to a star-shape antenna layout
==================================================================================

When using the origin library provided with template synthesis, the origin showers are simulated
on the :math:`\vec{v} \times \vec{v} \times \vec{B}` axis. This means that the synthesised
traces are also generated on this axis. However, in many cases, the user might want to have
traces on a star-shape antenna layout in the shower plane.

To achieve this, we provide the function :func:`smiet.numpy.utilities.reconstruct_starshape_from_vxvxB`,
which simply takes the synthesised traces on the :math:`\vec{v} \times \vec{v} \times \vec{B}` axis
and copies them to the desired star-shape antenna positions, applying a rotation in the shower plane.
This approach is possible since we expect the geomagnetic and charge-excess emission to
be rotationally symmetric in strength.
The number of arms can also be chosen freely, with the default being 8 arms (i.e. a star shape).

Example (NumPy)
----------------
.. code-block:: python

    from smiet.numpy.utilities import reconstruct_starshape_from_vxvxB

    synthesis = TemplateSynthesis()
    # .....
    # perform synthesis with origin shower on vxvxB arm only
    # .....
    synthesised_geo_vvB, synthesis_ce_vvB = synthesis.map_template(...)

    # we take only the y-positions of the antennas, since the origin shower
    # is simulated on the vxvxB axis (i.e. x=0)
    ant_positions_vvB = synthesis.antenna_information["position_showerplane"][:,1]

    # reconstructing to star-shape antenna layout
    synthesised_geo_starshape, synthesised_ce_starshape = reconstruct_starshape_from_vxvxB(
        synthesised_geo_vvB,
        synthesis_ce_vvB,
        ant_positions_vvB,
        number_of_arms = 8 # number of arms in the star-shape layout
    )

A similar implementation is provided in the JAX implementation of SMIET, which can be imported from
:func:`smiet.jax.utilities.utilities.reconstruct_star_from_vxvxB`.

Example (JAX)
----------------
.. code-block:: python

    from smiet.jax.utilities import reconstruct_starshape_from_vxvxB

    synthesis = TemplateSynthesisJAX()
    # .....
    # perform synthesis with origin shower on vxvxB arm only
    # .....
    synthesised_geo_vvB, synthesis_ce_vvB = synthesis.map_template(...)

    # we take only the y-positions of the antennas, since the origin shower
    # is simulated on the vxvxB axis (i.e. x=0)
    ant_positions_vvB = synthesis.ant_positions_vvB[:,1]

    # reconstructing to star-shape antenna layout
    synthesised_geo_starshape, synthesised_ce_starshape = reconstruct_starshape_from_vxvxB(
        synthesised_geo_vvB,
        synthesis_ce_vvB,
        ant_positions_vvB,
        number_of_arms = 8 # number of arms in the star-shape layout
    )
    