==============
Usage of TENET
==============

DSL and Embedded Operators
--------------------------
We implemented a Keras-like DSL to efficiently describe multi-steps tensor computation. 

It has simple functions like loop-structuring, block-defining, shape inference, and common tensor-computation templates like GEMM, Conv2D embedded.

Take SqueezeNet version1_2 for example::
 
    squeeze(num_outputs) {
        conv2d(num_outputs, rx=1, ry=1, stride=1, padding='valid')
    }

    expand(num_outputs) {
        e1x1 = conv2d(num_outputs, rx=1, ry=1, stride=1, padding = 'valid')
        e3x3 = conv2d(num_outputs, rx=3, ry=3, padding = 'same')
        concat() (e1x1,e3x3)
    }

    fire_module(squeeze_depth,expand_depth) {
        net = _squeeze(squeeze_depth) (input)
        net = _expand(expand_depth) (net)
    }

    result {
        x = conv2d(64, [3,3], strides=2, padding='valid')
        x = max_pool2d(pool_size=(3,3), strides=2) (x)

        x = fire_module(squeeze=16, expand=64) (x)
        x = fire_module(squeeze=16, expand=64) (x)
        x = max_pool2d(pool_size=(3, 3), strides=2)(x)

        x = fire_module(squeeze=32, expand=128) (x)
        x = fire_module(squeeze=32, expand=128) (x)
        x = max_pool2d(pool_size=(3, 3), strides=2)(x)

        x = fire_module(squeeze=48, expand=192) (x)
        x = fire_module(squeeze=48, expand=192) (x)
        x = fire_module(squeeze=64, expand=256) (x)
        x = fire_module(squeeze=64, expand=256) (x)
    }

    result shape=[]

Command Line Interface
----------------------
To specify statement, pe_array and mapping,
use :option:`-s`, :option:`-p`, :option:`-m`::

	tenet -s matmul.s -p systolic64.p -m output_stationary.m --all

This will model the mapping of output stationary matrix multiply dataflow to 8×8 systolic array, and output all metrics.

To model a batch of experiments,
use :option:`-e`::
    
    tenet -e data/alexnet --all

This will run experiments written in ``[project_root_dir]/data/alexnet`` directory.

To specify metric to compute,
use :option:`--factor`, :option:`--latency`, :option:`--peutil`, :option:`--all`.