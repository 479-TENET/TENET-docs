====================
Command Line Options
====================

Check out how to use these flags in the :doc:`usage`.

.. option:: -s <statement file>, --statement <statement file>

	Specify the tensor computation statement.


.. option:: -p <hardware description file>, --pe_array <hardware description file>

    Specify the PE Array scale, PE interconnect, on-chip SRAM size, bandwidth.


.. option:: -m <mapping file>, --mapping <mapping file>

    Specify the temporal and spatial mapping to the hardware.
    Todo: specify SRAM data layout


.. option:: -e <experiment directory>, --experiment <experiment directory>

    Run all experiments in the folder.


.. option:: -n <DSL file>, --network <DSL file>

    To get per-layer metrics of neural network, write description file with TENET's DSL.


.. option:: --factor, --latency, --peutil, --all
    
    Compute certain metrics or all.
    

.. option:: --excel
    
    Output the results to excel file