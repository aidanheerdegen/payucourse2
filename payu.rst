=====================
Shiney new Payu stuff
=====================

:subtitle: New and upcoming features
:author: Aidan Heerdegen
:description: A training course to introduce new and upcoming features
:date: 17 October 2018



What is Payu?
=============

Payu is
-------

 ... a python based "scientific workflow manager"

Huh?
----

That means it runs your model for you. in short:

* Setup model run directory

* Run the model

* Move outputs/restarts to an archive directory

* Clean up the run directory

* Run again (if instructed to do so)
  

Using Payu
==========

Using an experiment
-------------------

(Recap from last week)

Clone an existing experiment (usually in ``$HOME``):

.. code:: sh

   cd $HOME
   mkdir -p mom
   cd mom
   git clone /short/public/mxw900/payucourse/expt/bowl1

This is the "*control directory*" for ``bowl1``


Run the experiment
------------------

Use the system payu::

   module load payu/0.9.2

This job is pre-configured, run it!

.. code:: sh

   cd bowl1
   payu run

* Model will run in ``work/``

* Output saved to ``archive/``


New features
============

Fast MOM (FMS) collation
------------------------

There is a new mppnccombine in town ... and it's fast.

How fast? 

----

Probably not faster than the Waco Kid

.. image:: img/waco_kid.gif


----

.. notes::
   That is it collates tiled outputs to multiple files, which makes model input and output faster

   Directly copies the compressed chunks from one file to another, skipping the decompress/recompress step



But seriously fast, hence the name **mppnccombine-fast**

https://github.com/coecms/mppnccombine-fast

* Written by Scott Wales

* Collates any FMS model output (MOM5/MOM6/GOLD) that is tiled, that is output to multiple files to make input and output faster

* Particularly fast with netCDF4 compressed data


Requirements
------------

* ``mppnccombine-fast`` executable. Either copy from ``/short/public/access-om2/mppnccombine-fast`` or get source code and compile

* Place in ``/short/$PROJECT/$model/bin`` or somewhere else and specify the fullpath

* A version of ``payu`` of ``0.10`` or greater (``module load payu/0.10`` on ``raijin``)

* Updated ``config.yaml`` syntax


Old Syntax
----------

.. code:: yaml

    collate: true
    collate_mem: 16GB
    collate_queue: express
    collate_ncpus: 4
    collate_flags: -n4 -r


New syntax
----------

.. notes::
   Must specify mpi to use mppnccombine-fast.
   Minimum of 2 cpus, so can't use copyq
   ncpus per thread is ncpus / nthreads
   nthreads defaults to 1
   ncpus defaults to 2
   enable defaults to true
   Don't need to specify flags, enable or exe
   Fewer flags, as mppnccombine-fast has fewer options
   

Replaces kludgey ``collate_`` options with dictionary

.. code:: yaml

    collate:
         enable: true
         queue: express
         memory: 4GB
         walltime: 00:30:00
         mpi: true
         ncpus: 4
         threads: 2
         # flags: -v
         # exe: /full/path/to/mppnccombine-fast


Resource requirements
---------------------

.. notes:: 
    Memory use should only depend on chunksize in the compressed file, not on the overall size of the 
    file being written, so resolution independent.

    Unfortunately a memory leak bug in the underlying ``HDF5`` library means memory use will go up with 
    the number of times data is written to a collated file. It is difficult to predict, but 2-4GB per 
    thread has been the upper limit observed so far.

    No speed-up for low resolution outputs (MPI overhead swamps fast run times). Quarter degree 10-50x faster. 
    Tenth 100x faster.

* Memory independent of resolution (<4GB per thread)

* Walltime in minutes

* No speed-up for low resolution (1 deg global model) 

* Minimum of 2 cpus


Upcoming features
=================

File Tracking
-------------

Wanted to do this since forever.


Key Advantages
--------------

* Track input files use for each model run

* Reproducibly re-run previous experiment

* Share experiments more easily as input files all specified

* Flexibility with specifying path to input files


What is tracked?
----------------

.. notes:: 
   Executables and inputs are not expected to change. Can specify a flag to either warn 
   if they do and stop, or update manifest and continue
   
   Restarts are the opposite, and by default are always expected to be different for each
   run, unless a flag is specified to reproduce a run, in which case any difference will
   flag an error and stop

=========== ===================
Executables ``mf_exe.yml``
Inputs      ``mf_inputs.yml``    
Restarts    ``mf_restarts.yml``
=========== ===================


How is it tracked?
------------------

yamanifest file, which is a ``YaML`` file (like ``config.yml``)

.. code::yaml

    format: yamanifest
    version: 1.0
    ---
    work/fms_MOM_SIS.intel14:
      fullpath: /short/v45/aph502/mom/bin/fms_MOM_SIS.intel14
      hashes:
        binhash: 74b079574d3160fd2024ca928f3097a0
        md5: e10bf223ae2564701ae310d341bbe63b


ACCESS-OM2 Model Configs
========================


ACCESS-OM2 model hierarchy from 1 degree global to 0.1 degree global, Ocean/Ice
model forced with atmospheric data and almost identical model parameters.

Single ``access-om2`` repository

https://github.com/OceansAus/access-om2

Components
----------

========== ================
Ocean      ``MOM5``        
Ice        ``CICE5``       
Atmosphere ``libaccessom2``
Coupler    ``OASIS3-MCT``  
========== ================

Code
----

================ =========================================
``MOM5``         https://github.com/mom-ocean/MOM5
``CICE5``        https://github.com/OceansAus/cice5
``libaccessom2`` https://github.com/OceansAus/libaccessom2
``OASIS3-MCT``   https://github.com/OceansAus/oasis3-mct
================ =========================================

ACCESS-OM2
----------

Nominal 1 degree global resolution

ACCESS-OM2-025
--------------

Nominal 0.25 degree global resolution


ACCESS-OM2-01
--------------

Nominal 0.1 degree global resolution


Forcing Data
------------

JRA-55-do RYF and IAF (1955-present)

https://github.com/OceansAus/1deg_jra55_iaf
https://github.com/OceansAus/1deg_jra55_ryf