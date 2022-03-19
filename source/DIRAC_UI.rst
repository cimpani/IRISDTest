====================
Submit a simple job
====================

**Details at:**  `Simple_Job <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/CommandLine/index.html>`__

.. code:: python

   bash-4.2$ /raid/scratch/<your-user>/dirac_ui > cat simple.jdl
   JobName = "InputAndOuputSandbox";
   Executable = "pythonV.sh";
   StdOutput = "StdOut";
   StdError = "StdErr";
   InputSandbox = {"pythonV.sh"};
   OutputSandbox = {"StdOut","StdErr"};

   bash-4.2$ /raid/scratch/<your-user>/dirac_ui > cat pythonV.sh
   #!/bin/bash
   /usr/bin/python --version;


Monitor a simple job
=====================

**Details at:**  `Simple_Job <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/CommandLine/index.html>`__

.. code:: python

   bash-4.2$ /raid/scratch/<your-user>/dirac_ui > dirac-wms-job-submit simple.jdl
   JobID = 25104301

   bash-4.2$ /raid/scratch/<your-user>/dirac_ui > dirac-wms-job-status 25104301
   JobID=25104301 Status=Done; MinorStatus=Execution Complete;
   Site=LCG.UKI-NORTHGRID-MAN-HEP.uk;

- The job execution can be seen also on DIRAC `Web-link <https://dirac.gridpp.ac.uk:8443/DIRAC/>`__
(see Applications/Job Monitor -> Owner (your name) -> submit)

Put RASCIL.img in a file catalog
================================

**Details at:**  `File_Catalog <https://dirac.readthedocs.io/en/latest/UserGuide/CommandReference/DataManagement/index.html>`__

.. code:: python
   
   - Accessing File Catalog to add a testFile or a singularity container
   bash-4.2$ dirac-dms-filecatalog-cli
   Starting FileCatalog client
   FC:/> cd /skatelescope.eu/user
   
   - Go to the first letter of your user 
   FC:/skatelescope.eu/user>cd c
   
   - Create (mkdir) or go to your user folder
   FC:/skatelescope.eu/user/c>cd cimpan
   
   - Exit File Catalog
   FC:/skatelescope.eu/user/c/cimpan>exit
   bash-4.2$ 
   
   bash-4.2$ /raid/scratch/<your-user>/dirac_ui > dirac-dms-add-file LFN:/skatelescope.eu/user/<first letter of your user>/<your-user>/rascil/RASCIL.img RASCIL.img UKI-NORTHGRID-MAN-HEP-disk
   # UKI-NORTHGRID-MAN-HEP-disk - SE: DIRAC Storage Element

   Then you will find the file RASCIL.img under: 
   FC:/skatelescope.eu/user/<first letter of your user>/<your-user>/rascil/RASCIL.img

Submitting RASCIL job
=====================

.. code:: python

   cat simpleR1.jdl
   JobName    = "InputAndOuputSandbox";
   Executable = "testR1.sh";
   StdOutput = "StdOut";
   StdError = "StdErr";
   InputSandbox = {"testR1.sh"};
   InputData = {"LFN:/skatelescope.eu/user/c/cimpan/rascil/RASCIL-full1.img"};
   OutputSandbox = {"StdOut","StdErr","imaging_dirty.fits","imaging_psf.fits","imaging_restored.fits"};
   OutputSE ="UKI-NORTHGRID-MAN-HEP-disk";
   Site = "LCG.UKI-NORTHGRID-MAN-HEP.uk";

    cat testR1.sh
   #!/bin/bash
   singularity exec --cleanenv -H $PWD:/srv --pwd /srv -C RASCIL-full1.img python3 /rascil/examples/scripts/imaging.py;

Managing RASCIL job
===================

**Details at:**  `Simple_Job <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/CommandLine/index.html>`__

.. code:: python

   $ dirac-wms-job-submit simpleR1.jdl
   JobID = 25260750

   $ dirac-wms-job-status  25260750
   JobID=25260750 Status=Running; MinorStatus=Input Data Resolution; 
   Site=LCG.UKI-NORTHGRID-MAN-HEP.uk;

   $ dirac-wms-job-status  25260750
   JobID=25260750 Status=Done; MinorStatus=Execution Complete; 
   Site=LCG.UKI-NORTHGRID-MAN-HEP.uk;

Get Output Data RASCIL job
==========================

**Details at:**  `Simple_Job <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/CommandLine/index.html>`__

.. code:: python

   Note: the RASCIL job has 3 image outputs, so we specify them in 
   OutputSandbox and we take the data locally using command

   $ dirac-wms-job-get-output  25260750
   Job output sandbox retrieved in 
   /raid/scratch/<your-user>/dirac_ui/tests/rascilTests/25260750/
   $ cd 25260750
   $ ls
   imaging_dirty.fits  imaging_psf.fits  imaging_restored.fits  StdOut
   $ cat StdOut   #or StdErr

Useful Links

-   `What_is_IRIS <https://www.iris.ac.uk/>`__

-   `GridPP_user-guide <https://github.com/GridPP/user-guides>`__

-   `Getting_on_the_grid <https://github.com/gridpp/user-guides/tree/master/getting-on-the-grid>`__

-   `Grid_user_crash_course <https://www.gridpp.ac.uk/wiki/Grid_user_crash_course>`__

-   `Quick_Guide_to_Dirac <https://www.gridpp.ac.uk/wiki/Quick_Guide_to_Dirac>`__

-   `Getting_started_User_Jobs <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/index.html>`__

-   `Getting_started_Data_Management <https://dirac.readthedocs.io/en/latest/UserGuide/CommandReference/DataManagement/index.html>`__

-   `Getting_started_Command_Line <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/CommandLine/index.html>`__




:Author: Iulia Cimpan
:Date:   Nov 2021
