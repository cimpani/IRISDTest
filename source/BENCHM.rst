=====================================================================
Benchmarking processing performance and Parameterized jobs IRIS(cert)
=====================================================================

Benchmarking processing performance
===================================

Benchmarking and environment details:
-------------------------------------

-  where do we run the job

-  how much memory and how much processing the RASCIL imaging script is
   using

-  highly parallel so we may be able to use many processor nodes - how
   much memory do we need?

-  profiling doing: benchmarks in the memory and efficiency use of nodes

**Rules IRIS imaging script**

-  we have the component to run it: eg. RASCIL container on CVMFS (or under FC)

-  the user provides script and data

-  we provide:

   -  the .jdl model

   -  the singularity container on CVMFS, always latest release (or under FC) 

   -  environment has to be fully specified in container

   -  the container doesn’t need to be uploaded by the user

**The .jdl model rules:**

-  the singularity container has to be on CVMFS, always latest release (or under FC) 

-  scripts should be only input sandbox (not on InputData)

-  large input/output datasets and data (including .tar .gz) should be
   on file catalogue only (LFN) - use InputData and OutputData
   parameters

-  OutputSandbox can contain standard output (StdOut), standard error
   (StdErr), small log files, job.info etc (only small files outputs)

The .jdl and .sh models samples
-------------------------------

**Specific .jdl model with test data set for efficiency use of nodes:**

.. code:: python

   jobName = "%n_RASCILpipeline";

   Parameters=3;
   ParameterStart=1;
   ParameterStep=1;

   Executable = "trial.sh";

   StdOutput = "StdOut_%n";
   StdError = "StdErr_%n";

   Tags = {"8Processors","skatelescope.eu.hmem"}; # 2Processors, 4Processors,..,32Processors

   SitesList = "LCG.UKI-NORTHGRID-MAN-HEP.uk";
   SEList = "UKI-NORTHGRID-MAN-HEP-disk";

   InputSandbox = {"trial.sh","input_file1.json"};
   InputData = {"LFN:/skatelescope.eu/user/c/cimpan/rascil/
   emerlin_rascil_pipeline_for1252_1.tar",
   "LFN:/skatelescope.eu/user/w/willice.obonyo/IRIS_RASCIL_test/1252+5634.tar"};

   OutputSandbox = {"StdOut_%n","StdErr_%n","*.log"};
   OutputData = {"outputs.tar"};

   OutputSE = "UKI-NORTHGRID-MAN-HEP-disk";

**Specific .jdl model with test data set for memory benchmark:**

.. code:: python

   jobName = "RASCILpipeline";

   Executable = "trial.sh";

   StdOutput = "StdOut";
   StdError = "StdErr";

   Tags = {"8Processors","skatelescope.eu.hmem"};

   SitesList = "LCG.UKI-NORTHGRID-MAN-HEP.uk";
   SEList = "UKI-NORTHGRID-MAN-HEP-disk";

   InputSandbox = {"trial.sh","input_file1.json"};
   InputData = {"LFN:/skatelescope.eu/user/c/cimpan/rascil/
   prmon_2.0.2_x86_64-static-gnu93-opt.tar.gz",
   "LFN:/skatelescope.eu/user/c/cimpan/rascil/emerlin_rascil_pipeline_for1252_1.tar",
   "LFN:/skatelescope.eu/user/w/willice.obonyo/IRIS_RASCIL_test/1252+5634.tar"};

   OutputSandbox = {"StdOut","StdErr","job.info","*.log"};
   OutputData = {"outputs.tar","prmon.txt","prmon.json"};

   OutputSE = "UKI-NORTHGRID-MAN-HEP-disk";

**Specific .sh model used for both .jdl above:**

.. code:: python

   vi trial.sh
   #printenv;
   echo "==============================================";
   singularity --version;

   echo "Printing parameters"
   echo $0
   echo $1 #nprocs
   echo $2 #id_start
   echo $3 #id_end
   echo $4 #experiment
   echo "Processors: ${OMP_NUM_THREADS}";

   tar -xzvf 1252+5634.tar
   tar -xzvf emerlin_rascil_pipeline_for1252_1.tar

   echo "Extracting Process Monitor - This is to monitor the processes that we will run"

   mkdir -p prmon && tar xf prmon_2.0.2_x86_64-static-gnu93-opt.tar.gz -C prmon
   --strip-components 1

   echo "Running prmon"
   ./prmon/bin/prmon -p $$ -i 0 -u &


   time singularity exec --cleanenv -H $PWD:/srv --pwd /srv -C
   /cvmfs/sw.skatelescope.eu/images/rascil.img python3
   emerlin_rascil_pipeline/erp2_script.py --params input_file1.json

   tar czf outputs.tar *.fits
   

Note: the input_file1.json file can be downloaded from `here <https://github.com/cimpani/IRISDocumentation/blob/main/input_file1.json>`__

**How .jdl model for efficiency use of nodes (modelcpu.jdl) works:**

The .jdl can be used for 2Processors, 4Processors,..,32Processors
Parameters=3; means 3 jobs will be submitted (you can also choose
Parameters=10);

.. code:: python

   bash-4.2$ dirac-wms-job-submit modelcpu.jdl
   JobID = [26381707, 26381708, 26381709]
   Output data and logs
   bash-4.2$ dirac-wms-job-get-output 26381829
   bash-4.2$ dirac-wms-job-get-output-data 26381829
   Job 26381829 output data retrieved
   bash-4.2$ ls
   erp.log outputs.tar StdErr_2 StdOut_2
   bash-4.2$ tar -xzvf outputs.tar
   eMERLIN_testing_pipeline_1252+5634_cip_deconvolved_moment0.fits
   eMERLIN_testing_pipeline_1252+5634_cip_residual_moment0.fits
   eMERLIN_testing_pipeline_1252+5634_cip_restored_moment0.fits



We use the benchmarking `script <https://github.com/cimpani/IRISDocumentation/blob/main/benchm>`__

.. code:: python

   bash-4.2$ ./benchm 26381707 26381708 26381709

The output is stored in paramslog.csv, which can be opened on own workstation using Excel.

.. figure:: table.png
   :alt: Jobs parameters
   :name: fig:param

   Jobs parameters

Efficiency is calculated as TotalCPUTime(s)/(WallClockTime(s)*Number of
Processors) Mean and standard deviation can be calculated on efficiency
and then error bars can be plotted against mean WallClockTime(s). Below
is a plot for 10 jobs ran on processors 2 to 32.

.. figure:: 1252meaneff.png
   :alt: TotalCPUTime(s)/(WallClockTime(s)*Number of Processors)
   :name: fig:meaneff

   TotalCPUTime(s)/(WallClockTime(s)*Number of Processors)

**How .jdl model for efficiency use of nodes (modelm.jdl) works:**

The model uses `PRMON  <https://github.com/HSF/prmon/blob/main/README.md>`__ (PRocess MONitor) program. The output files are "prmon.txt","prmon.json" where "prmon.txt" can be plotted using “prmon_plot.py”. Example of plots are in figure below:

.. figure:: pr1252.png
   :alt: TotalCPUTime(s)/(WallClockTime(s)*Number of Processors)
   :name: fig:pr1252

   TotalCPUTime(s)/(WallClockTime(s)*Number of Processors)
   



Parameterized jobs - ALMA RASCIL vs. ALMA CASA tests
=====================================================

This documentation offers CASA vs. RASCIL scripts on ALMA datasets, for results comparison:

-   `ALMA_RASCIL <https://ska-telescope.gitlab.io/external/rascil/installation/RASCIL_docker.html#singularity>`__

-   `ALMA_CASA <https://casaguides.nrao.edu/index.php/ALMA2014_LBC_SVDATA>`__

Three ALMA datasets are already uploaded under LFN, these are: HLTau_Band6, HLTau_Band7 and Mira_Band6. ALMA datasets are stored under LFN as: %s_CalibratedData.tgz, where %s is the name of the parameter: 

.. code:: python


   Parameters = {"HLTau_Band6","HLTau_Band7","Mira_Band6"};

   see the parameters in the .jdl 


ALMA RASCIL
-----------

-  Scripts:

   Folder `alma_rascil <https://github.com/cimpani/AlmaTests/tree/main/alma_rascil>`__ contains all scripts. Download files and submit to IRIS three jobs, by using the below command:
 
.. code:: python


   $ dirac-wms-job-submit alma_rascil.jdl

   
   
More parameters can be added to the Parameters={"HLTau_Band6",,,,} list above, along with the ALMA datasets and their own configuration files. See example config files in configfiles.tar.gz. The steps, to add another dataset to be processed, are:
	- download ALMA dataset and store it under LFN with the name %s_CalibratedData.tgz (%s being one of the parameters, eg. HLTau_Band6)
	- create a configuration file for this dataset and add it to the configfiles.tar.gz
	- add the new parameter to the list of parameters in the .jdl file, Parameters = {"HLTau_Band6","HLTau_Band7","Mira_Band6"};
 
 
ALMA RASCIL tests are using a RASCIL container build from a recipe to enable the usage of config files. An example of recipe file that can be used is: 
 
 
 .. code:: python
 
    recipe file that uses an already pulled local RASCIL container: RASCIL-fullN.img
    
    $ cat SingRascilCasa.recipe 
    bootstrap: localimage 
    from: /home/<user>/RASCIL-fullN.img 
    %post
	cd /var/lib
	git clone https://github.com/casacore/casacore
	apt-get update && \
           apt-get -y install sudo	
	sudo apt-get -y install build-essential cmake gfortran g++ libncurses5-dev \
   		libreadline-dev flex bison libblas-dev liblapacke-dev libcfitsio-dev \
   		wcslib-dev libfftw3-dev
	sudo apt-get -y install libhdf5-serial-dev python-numpy \
   		 libboost-python-dev libpython3.7-dev  libpython2.7-dev
	cd casacore
	mkdir build
	cd build
	cmake ..
	make
	make install
 
 
     building the new container from recipe uses the command:
     
     $ sudo singularity build  RASCIL-full.img SingRascilCasa.recipe
   
   
ALMA CASA
----------

-  Scripts:

   Folder `alma_casa <https://github.com/cimpani/AlmaTests/tree/main/alma_casa>`__ contains all scripts. Download files and submit to IRIS three jobs, by using the below command:
   
.. code:: python


   $ dirac-wms-job-submit alma_casa.jdl 

   
ALMA CASA scripts have been downloaded and their name has been changed to %s_Imaging.py, where %s is the name of the parameter (because of use of parametrized jobs). The files will be then archived into imgfiles.tar. These scripts are using the ALMA datasets %s_CalibratedData.tgz mentioned above.



RESULTS HLTau_Band6 DATASET
----------------------------

These are the results for HLTau_Band6 DATASET `ALMA_RASCIL <https://github.com/cimpani/AlmaTests/tree/main/results%20HLTAU6%20rascil>`__  and `ALMA_CASA <https://github.com/cimpani/AlmaTests/tree/main/results%20HLTAU6%20casa>`__ 
Two fits images (RASCIL vs CASA outputs) converted to png are being shown below:

- RASCIL output `png <https://github.com/cimpani/AlmaTests/blob/main/ical_restored_image_for_HL_Tau.png>`__  (conversion of output image ical_restored_image_for_HL_Tau.fits)

- CASA output `png <https://github.com/cimpani/AlmaTests/blob/main/HLTau_B6cont_mscale_ap.image.png>`__  (conversion of output image HLTau_B6cont_mscale_ap.image.fits)

