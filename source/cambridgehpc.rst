===============================================
Before starting: useful tools, basic knowledge
===============================================


Cambridge CSD3 cheat sheet
==========================

The `Cambridge CSD3 system
<https://docs.hpc.cam.ac.uk/hpc/index.html>`_ is documented
extensively on its own web pages. These are internal notes for quick
start for our project.

.. _cambridgehpc-login:

Login
-----

Log in with your username to the CPU system (DIRAC username):

	.. code-block:: console

		(host) $ ssh <username>@login-cpu.hpc.cam.ac.uk

If you have set up an authorized key, you may use this to bypass the
password prompt:

	.. code-block:: console

		(host) $ ssh -i ~/.ssh/dirac_rsa <username>@login.hpc.cam.ac.uk

Work area
---------

The project shared disk space is ``/rds/project/rds-bRdYdViqoGA/``.
Make yourself a working directory there.

Jobs
----

The system uses Slurm (though for those familiar with Torque, ``qsub``
and ``qstat`` commands work more or less as you would expect on the
login nodes).

There are three main types of compute nodes on the cluster, Skylake,
Cascade Lake and Ice Lake (from earlier to later generation). Skylake
nodes are 32-core nodes, Cascade Lake 56 core, and Ice Lake 76 core
nodes. All are Intel-based. These show up as partitions (like Torque
queues) on the cluster, i.e. you are explicitly selecting the type of
machine you will get when you submit to the cluster using the
partitions ``skylake``, ``cclake`` or ``icelake``. There are also
himem (high-memory) version of each partition.

To get a single core for testing purposes on one of these machines do
e.g.

	.. code-block:: console

		(CSD3) $ sintr -p icelake

This drops you in a ``screen`` session on a compute node. You can
specify wall time, numbers of cores etc on the command line.

To run a batch job do e.g.

	.. code-block:: console

		(CSD3) $ sbatch example_slurm_script

You can find template job scripts in
``/usr/local/Cluster-Docs/SLURM``. A useful option in batch mode is
``#SBATCH --mail-type=ALL`` which ensures that you will be e-mailed on
start, end and error. Job output appears in the directory where you
ran the ``sbatch`` command.

To view the queue in native Slurm mode use the ``squeue`` command.

Note that if (and only if) you have a Slurm job running on a compute
node you are allowed to ``ssh`` into it to check on the job, run
``htop`` etc.
		
Modules
-------

Modules work as on the Hertfordshire system, e.g. ``module load
singularity`` gets you access to the singularity command. There are
many more modules available though.




Slurm Cheat Sheet
=================

The Cambridge HPC uses slurm to manage resources and schedule jobs. The following are a list of commands that may prove useful when managing slurm jobs.

For more detailed descriptions of slurm commands there are several available online resources including:

* `https://www.cism.ucl.ac.be/Services/Formations/slurm/2016/slurm.pdf <https://www.cism.ucl.ac.be/Services/Formations/slurm/2016/slurm.pdf>`_
* `https://slurm.schedmd.com/man_index.html <https://slurm.schedmd.com/man_index.html>`_

Additional information about each of the commands described, for example ``sinfo``, can also be obtained using the man command, for example ``man sinfo``.


Account Balance
---------------

Use the following command to get information about the amount of hours available for use on each of the accounts to which you have access:

	.. code-block:: console

		(host) $ mybalance

This will return a list similar to the following:

+------------+------------------------+-------------+-----------------+
|User / Usage|Account / Usage         |Account Limit|Available (hours)|
+============+========================+=============+=================+
|dc-webs1 / 1|DIRAC-TP001-CPU / 12,000|500,000      |488,000          |
+------------+------------------------+-------------+-----------------+
|dc-webs1 / 0|DIRAC-TP001-GPU /0      |2,000        |2,000            |
+------------+------------------------+-------------+-----------------+

* User is the name of the user whose account information is being shown
* Account shows the account names available to the user along with the amount of hours that have already been spent against that account
* Account Limit details the maximum number of hours allocated to each account
* Available lists the amount of hours still available to be spent


Getting Cluster Info
--------------------

To get information about the available nodes and partitions use the following command:

	.. code-block:: console

		(host) $ sinfo

This will return a list similar to the following. Each partition is listed with separate entries for each unique node state.

+---------+-----+----------+-----+-----+--------------------------------------------+
|PARTITION|AVAIL|TIMELIMIT |NODES|STATE|NODELIST                                    |
+=========+=====+==========+=====+=====+============================================+
|icelake  |up   |7-00:00:00|18   |idle |cpu-q-[58,97,99-100,169-176,397-400,411-412]|
+---------+-----+----------+-----+-----+--------------------------------------------+
|icelake  |down |7-00:00:00|1    |down |cpu-q-244                                   |
+---------+-----+----------+-----+-----+--------------------------------------------+

* PARTITION is the partition name
* AVAIL states whether the partition is currently available or not
* TIMELIMIT gives the maximum time limit for any user job in days-hours:minutes:seconds. `infinite` is displayed if there is no time limit
* NODES gives the number of nodes in this state
* STATE gives the current state of the nodes. The most common states are `allocated` if the node has been allocated to a job, `down` if the node is unavailable and `idle` if the node is unallocated
* NODELIST lists the names of every node in this state

Monitoring Jobs
---------------

To see the status of all jobs submitted by an individual user use the following command replacing `username` with your user name:

	.. code-block:: console

		(host) $ squeue -u username

This will return a list similar to the following:

+--------+---------+--------------------+--------+--+----+-----+----------------+
|JOBID   |PARTITION|NAME                |USER    |ST|TIME|NODES|NODELIST(REASON)|
+========+=========+====================+========+==+====+=====+================+
|51610182|icelake  |VLA-Combining-Images|dc-webs1|R |5:12|1    |cpu-q-79        |
+--------+---------+--------------------+--------+--+----+-----+----------------+
|51610183|icelake  |VLA-Basic-Imaging   |dc-webs1|PD|0:00|1    |                |
+--------+---------+--------------------+--------+--+----+-----+----------------+

* JOBID is the id assigned to the job by slurm
* PARTITION describes which partition the job is scheduled to be run on, icelake in the above example.
* NAME gives the name of the script as defined using the ``#SBATCH -J`` command
* USER is the name of the user who submitted the job, dc-webs1 in the above example
* ST is the state of the job. The most common states are `R` if the job is running, `PD` if the job is pending, `F` if the job has failed and `CD` if the job completed successfully
* TIME is the amount of time used by the process
* NODES is the number of nodes allocated to the job
* NODELIST(REASON) lists the names of the nodes allocated to the job

Controlling Jobs
----------------

To cancel a job use the following command replacing `jobid` with the identifier of the job to be cancelled:

	.. code-block:: console

		(host) $ scancel jobid




.. _VLA-basic-imaging:

VLA Basic Imaging
=================

This tutorial uses the measurement set from the CASA tutorial `VLA Continuum Tutorial 3C 391 <https://casaguides.nrao.edu/index.php?title=VLA_Continuum_Tutorial_3C391-CASA6.2.0>`_ to generate an image of the supernova remnant 3C 391. The data in the measurement set was taken using a bandwidth of 128 MHz centred at 4.6 GHz with the VLA in D-configuration. The data set includes polarisation data that is not used in this tutorial.

.. _VLA-basic-imaging-getting-started:

Getting Started
---------------

#. The data required for this workflow can be downloaded from the CASA tutorial web page `VLA Continuum Tutorial 3C 391 <https://casaguides.nrao.edu/index.php?title=VLA_Continuum_Tutorial_3C391-CASA6.2.0>`_. N.B. This data set has been modified to include only the data necessary to run this tutorial, if desired the full observation can be downloaded fro the `VLA Archive <https://archive.nrao.edu/archive/advquery.jsp>`_ using the archive file id ``TDEM0001_sb1218006_1.55310.33439732639``. If downloading from the VLA archive it is normally easiest to download it as a tar file.

   After downloading the data untar it:

	.. code-block:: console

		(host) $ tar -xzvf 3c391_ctm_mosaic_10s_spw0.ms.tgz

   There are four fields in this observation

	* \field 0 is 3C 286 (also referred to as J1331+3030), the bandpass calibrator
	* \field 1 is J1822-0938, the phase calibrator
	* \fields 2-8 inclusive are the mosaicked fields that make up 3C 391, the target
	* \field 9 is 3C 84 (also referred to as J0319+4130), the polarization calibrator. This field is unused in this workflow
	
#. This observation was taken on the 24th April 2010 between 08:02 and 16:00. The operators log can be downloaded from `here <http://www.vla.nrao.edu/cgi-bin/oplogs.cgi>`_.

#. Load the singularity module by entering:

	.. code-block:: console

		(host) $ module load singularity

#. Download the singularity image created in :ref:`use-of-singularity`. The following command downloads the most up to date image which is 1.3 GB and so this may take some time!

	.. code-block:: console

		(host) $ singularity pull library://mhardcastle/default/casa

#. CASA is started automatically when the singularity image is run:

	.. code-block:: console

		(host) $ singularity run casa_latest.sif

   N.B. If the data and singularity are not installed in the users home directory the above command will load CASA but any files saved will be output to the home directory rather than the current working directory. To specify which directory on the server is to be  treated as the home directory within the singularity use the -H command. For example, to change the singularity home directory to */beegfs/lofar/user* start the singularity using:

	.. code-block:: console

		(host) $ singularity run -H /beegfs/lofar/user:/home casa_latest.sif

Flagging
--------

All the following commands are run from the CASA command prompt inside the singularity

#. The operators log states that antenna *ea13* has no receiver and *ea15* has corrupted data. Both antennas are therefore flagged:

	.. code-block:: console

		(CASA) $ flagdata(vis='3c391_ctm_mosaic_10s_spw0.ms', flagbackup=True, mode='manual', antenna='ea13,ea15')


#. Generate a file containing the observation log. The following command will create a file called 3C391.listobs.

	.. code-block:: console

		(CASA) $ listobs(vis='3c391_ctm_mosaic_10s_spw0.ms', verbose=True, listfile='3C391.listobs')

#. From the observation log it can be seen that the first scan of the bandpass calibrator, 3C 286, was extremely short (20 seconds). This was a setup scan which is therefore flagged:

	.. code-block:: console

		(CASA) $ flagdata(vis='3c391_ctm_mosaic_10s_spw0.ms', flagbackup=True, mode='manual', scan='1')

#. At the start of each scan it typically takes a few moments for the VLA antennas to settle into position. As a result it is common practice to remove the first few seconds of data from the start of each scan. In the example below we flag (or `quack`) the first 10 seconds of each scan.

	.. code-block:: console

		(CASA) $ flagdata(vis='3c391_ctm_mosaic_10s_spw0.ms', flagbackup=True, mode='quack', quackinterval=10.0, quackmode='beg')

#. Sharp peaks in RFI may cause Gibbs ringing. This usually occurs for narrow band RFI and is observable as a zig-zag pattern in the neighbouring channels. To prevent this the data can be Hanning smoothed. N.B. Hanning smoothing decreases the spectral resolution by a factor of two and may not be appropriate when performing spectral analysis.

	.. code-block:: console

		(CASA) $ hanningsmooth(vis='3c391_ctm_mosaic_10s_spw0.ms', outputvis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', datacolumn='data')

#. Using *tfcrop* to automatically flag any visibility amplitude outliers. In the code below it flags data more than 3 standard deviations away from both the time and frequency fits.

	.. code-block:: console

		(CASA) $ flagdata(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', mode='tfcrop', datacolumn='data', timecutoff=3.0, freqcutoff=3.0)

#. RFI-rich spectral windows may still contain significant amounts of RFI. To improve flagging we increase the contrast between clean and affected data by performing a coarse preliminary bandpass calibration to take out the bandpass shape from the data. 

	.. code-block:: console

		(CASA) $ gencal(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', caltable='3c391_ctm_mosaic_10s_spw0-smoothed.antpos', caltype='antpos')

		(CASA) $ gaincal(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', caltable='3c391_ctm_mosaic_10s_spw0-smoothed.G0', gaintype='G', calmode='p', solint='int', field='0', refant='ea21', minsnr=5.0, spw='0:27~36', gaintable=['3c391_ctm_mosaic_10s_spw0-smoothed.antpos'])

		(CASA) $ gaincal(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', caltable='3c391_ctm_mosaic_10s_spw0-smoothed.K0', gaintype='K', solint='inf', field='0', refant='ea21', minsnr=5.0, spw='0:5~58', combine='scan', gaintable=['3c391_ctm_mosaic_10s_spw0-smoothed.antpos','3c391_ctm_mosaic_10s_spw0-smoothed.G0'])

		(CASA) $ bandpass(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', caltable='3c391_ctm_mosaic_10s_spw0-smoothed.B0', solint='inf', field='0', refant='ea21', spw='', combine='scan', gaintable=['3c391_ctm_mosaic_10s_spw0-smoothed.antpos','3c391_ctm_mosaic_10s_spw0-smoothed.G0','3c391_ctm_mosaic_10s_spw0-smoothed.K0'])

		(CASA) $ applycal(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', calwt=False, gaintable=['3c391_ctm_mosaic_10s_spw0-smoothed.antpos','3c391_ctm_mosaic_10s_spw0-smoothed.G0','3c391_ctm_mosaic_10s_spw0-smoothed.K0','3c391_ctm_mosaic_10s_spw0-smoothed.B0'])

#. Having done the preliminary bandpass we can now use *rflag* to do some more flagging. In this example we flag RFI that is more than 5 standard deviations away from both the time and frequency-calculated median values.

	.. code-block:: console

		(CASA) $ flagdata(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', mode='rflag', datacolumn='corrected', timedevscale=5.0, freqdevscale=5.0, flagbackup=True)

Calibration
-----------

#. Before calculating the final calibration tables we must remove the preliminary calibration that was done in order to run *rflag*

	.. code-block:: console

		(CASA) $ clearcal(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms')

		(CASA) $ setjy(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', field='0', model='3C286_C.im', standard='Perley-Butler 2017')

#. Now that all the RFI is (hopefully) removed, we calculate the final calibration tables for the primary calibrator, which in this example is 3C 286 (field 0).

	.. code-block:: console

		(CASA) $ gaincal(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', caltable='3c391_ctm_mosaic_10s_spw0-smoothed.G1', gaintype='G', calmode='p', solint='int', field='0', refant='ea21', spw='0:27~36', minsnr=5.0, gaintable=['3c391_ctm_mosaic_10s_spw0-smoothed.antpos'])

		(CASA) $ gaincal(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', caltable='3c391_ctm_mosaic_10s_spw0-smoothed.K1', gaintype='K', solint='inf', field='0', refant='ea21', spw='0:5~58', combine='scan', minsnr=5.0, gaintable=['3c391_ctm_mosaic_10s_spw0-smoothed.antpos','3c391_ctm_mosaic_10s_spw0-smoothed.G1'])

		(CASA) $ bandpass(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', caltable='3c391_ctm_mosaic_10s_spw0-smoothed.B1', solint='inf', field='0', refant='ea21', spw='', combine='scan', gaintable=['3c391_ctm_mosaic_10s_spw0-smoothed.antpos','3c391_ctm_mosaic_10s_spw0-smoothed.G1','3c391_ctm_mosaic_10s_spw0-smoothed.K1'])

#. Calculate the gain calibration table for the primary and secondary calibrators simultaneously

	.. code-block:: console

		(CASA) $ gaincal(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', caltable='3c391_ctm_mosaic_10s_spw0-smoothed.G2', gaintype='G', calmode='ap', solint='inf', field='0,1', refant='ea21', spw='0:5~58', gaintable=['3c391_ctm_mosaic_10s_spw0-smoothed.antpos','3c391_ctm_mosaic_10s_spw0-smoothed.K1','3c391_ctm_mosaic_10s_spw0-smoothed.B1'])

#. Scale the amplitude gains, N.B. Setting *incremental=False* makes a new output table that replaces the gain calibration table.

	.. code-block:: console

		(CASA) $ fluxscale(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', caltable='3c391_ctm_mosaic_10s_spw0-smoothed.G2', fluxtable='3c391_ctm_mosaic_10s_spw0-smoothed.fluxscale2', reference='0', transfer=['1'], incremental=False)

#. Apply the calibration

	.. code-block:: console

		(CASA) $ applycal(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', field='0', gaintable=['3c391_ctm_mosaic_10s_spw0-smoothed.antpos','3c391_ctm_mosaic_10s_spw0-smoothed.fluxscale2','3c391_ctm_mosaic_10s_spw0-smoothed.K1','3c391_ctm_mosaic_10s_spw0-smoothed.B1'], gainfield=['','0','',''], interp=['','nearest','',''], calwt=False)

		(CASA) $ applycal(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', field='1', gaintable=['3c391_ctm_mosaic_10s_spw0-smoothed.antpos','3c391_ctm_mosaic_10s_spw0-smoothed.fluxscale2','3c391_ctm_mosaic_10s_spw0-smoothed.K1','3c391_ctm_mosaic_10s_spw0-smoothed.B1'], gainfield=['','1','',''], interp=['','nearest','',''], calwt=False)

		(CASA) $ applycal(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', field='2~8', gaintable=['3c391_ctm_mosaic_10s_spw0-smoothed.antpos','3c391_ctm_mosaic_10s_spw0-smoothed.fluxscale2','3c391_ctm_mosaic_10s_spw0-smoothed.K1','3c391_ctm_mosaic_10s_spw0-smoothed.B1'], gainfield=['','1','',''], interp=['','linear','',''], calwt=False)

Imaging
-------

#. Before starting to image the data it is recommended to run *statwt* to remove any noise scatter that may have been caused by flagging.

	.. code-block:: console

		(CASA) $ statwt(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', datacolumn='data')

#. When cleaning an image, it is recommended that the pixel size is set so that there are at least 4-5 pixels across the beam. The image in this example has a resolution of 12 arcsec and so a pixel size of 2.5 arcsec is chosen.

#. When using the VLA there is significant sensitivity outside of the main lobe of the primary beam. Any sources detected in the sidelobes need cleaning and removing from the dirty beam. This can be done either by using outlier fields or, as in this tutorial, creating very large images that encompass these interfering sources. We therefore create images that are approximately twice the size of the primary beam. In this example this is 20 arcmin.

#. In addition, the CASA algorithm *tclean* is computationally faster when using image sizes of 5\*2 :superscript:`n` \*3 :superscript:`m` pixels where n and m are integer numbers. Therefore, in this workflow we generate images that are 480 pixels both vertically and horizontally

#. Create the dirty image. 

	.. code-block:: console

		(CASA) $ tclean(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', field='2~8',imagename='3C391_Dirty', cell=['2.5arcsec','2.5arcsec'], imsize=[480,480], niter=0, stokes='I', gridder='mosaic')

#. Export the dirty image to a fits file

	.. code-block:: console

		(CASA) $ exportfits(imagename='3C391_Dirty.image', fitsimage='3C391_Dirty.fits', dropstokes=True, dropdeg=True)

#. Find the rms across the whole image. N.B. The following command does not exclude the source region, instead it uses the entire image to calculate the RMS in Jy.

	.. code-block:: console

		(CASA) $ stats = imstat(imagename='3C391_Dirty.image')

		(CASA) $ stats['rms']

#. Create the clean image.

	.. code-block:: console

		(CASA) $ tclean(vis='3c391_ctm_mosaic_10s_spw0-smoothed.ms', field='2~8',imagename='3C391_Clean', cell=['2.5arcsec','2.5arcsec'], imsize=[480,480], niter=20000, threshold='1.0mJy', stokes='I', gridder='mosaic', deconvolver='multiscale', scales=[0, 5, 15, 45], smallscalebias=0.9, weighting='briggs', robust=0.5, pbcor=True)

#. Finally, export the image into a fits-format file

	.. code-block:: console

		(CASA) $ exportfits(imagename='3C391_Clean.image', fitsimage='3C391_Clean.fits', dropstokes=True, dropdeg=True)

.. _VLA-basic-imaging-running-as-a-script:

Running as a Script
-------------------

The script used in this example is called VLA_Basic_Imaging_Script.py and can be downloaded :download:`here <scripts/vla/VLA_Basic_Imaging_Script.py>`.

After downloading the measurement set and starting the singularity instance as described in :ref:`VLA-basic-imaging-getting-started`, the above commands can be run as a single script using the following command:

	.. code-block:: console

		(CASA) $ execfile('VLA_Basic_Imaging_Script.py')

Running the script as a Slurm job on Cambridge CSD3
---------------------------------------------------

#. Start by logging on to the `Cambridge CSD3 system <https://docs.hpc.cam.ac.uk/hpc/index.html>`_ as described in :ref:`cambridgehpc-login`. If necessary download the casa singularity as described in :ref:`VLA-basic-imaging-getting-started`.

#. Create the following slurm script. This script used in this exmaple is named :download:`VLA_Basic_Imaging.slurm <scripts/vla/VLA_Basic_Imaging.slurm>`, though it can be given any name. Note, the slurm script calls the same :download:`VLA_Basic_Imaging_Script.py <scripts/vla/VLA_Basic_Imaging_Script.py>` script used in :ref:`VLA-basic-imaging-running-as-a-script`. If necessary, download and install this script into the working directory.

	.. code-block:: console

		#!/bin/bash
		#SBATCH -J VLA-Basic-Imaging
		#SBATCH -A DIRAC-TP001-CPU
		#SBATCH -p icelake
		#SBATCH --nodes=1
		#SBATCH --ntasks=1
		#SBATCH --time=00:45:00
		#SBATCH --mail-type=ALL
		#SBATCH --no-requeue

		#! Enter the script to run here
		. /etc/profile.d/modules.sh
		module load rhe18/default-icl
		module load singularity
		singularity exec casa_latest.sif casa -c VLA_Basic_Imaging_Script.py

#. Note the following points about the slurm script:

	* The command ``#SBATCH -J VLA-Basic-Imaging`` names the job VLA-Basic-Imaging
	* The command ``#SBATCH -A DIRAC-TP001-CPU`` is the name of the project under which time has been allocated
	* The command ``#SBATCH -p icelake`` ensures we are using the icelake cluster
	* The command ``#SBATCH --nodes=1`` states we only require a single node
	* The command ``#SBATCH --ntasks=1`` states we are running a single task/process. By default slurm allocates one task per cpu and so we are effectively asking for a single cpu. On icelake each CPU has a single core and by default each icelake CPU is allocated 3380 MiB of memory.
	* The command ``#SBATCH --time=00:45:00`` is requesting 45 (wall-clock) minutes of processing time.
	* The command ``#SBATCH --mail-type=ALL`` means email messages will be sent at the start and end of the job or (if applicable) when an error occurs. To disable this set the option to ``NONE``.
	* The command ``#SBATCH --no-requeue`` means that if this job is interrupted by a node failure/system downtime it will `not` be automatically rescheduled.
	* The command ``. /etc/profile.d/modules.sh`` enables the module command
	* The command ``module load rhe18/default-icl`` loads the basic environment needed by icelake
	* The command ``module load singularity`` loads the singularity module
	* The command ``singularity exec casa_latest.sif casa -c VLA_Basic_Imaging_Script.py`` executes the command ``casa -c VLA_Basic_Imaging_Script.py`` within the singularity environment. This is the command that runs the casa script.

#. Run the slurm script by entering

	.. code-block:: console

		(host) $ sbatch VLA_Basic_Imaging.slurm






Parallel Processing
===================

This tutorial describes how to set up a slurm script to simultaneously process multiple VLA images. This tutorial processes two images taken of the wide angle tail 3C 465. One image was taken in A configuration and the other was taken in B configuration. Both images were taken in the L-band. The data used in this tutorial can be downloaded from the `VLA archive <https://science.nrao.edu/facilities/vla/archive/index>`_ by entering the project code ``12A-195`` and choosing the B-configuration data taken in the 28th May 2012 and the A-configuration data taken on the 31st October 2012.

This script makes use of the ``parallel`` module to manage resources and enable parallel processing. The parallel module utilises all available processors, so that several processes can be run in parallel with each other. If there are more processes called than there are available processors, the parallel module will wait for a process to complete before starting the next process. For example, if there are three tasks (A, B and C) and only two processors parallel will initially execute tasks A and B. It will then only start running task C once either task A or B has completed.


.. _Parallel-Processing-getting-started:

Getting Started
---------------

#. After downloading the data untar it using the command

	.. code-block:: console

		(host) $ tar -xvf 12A-195.sb12378614.eb13812032.56231.2797265625.ms.tar
		(host) $ tar -xvf 12A-195.sb7351094.eb10491731.56075.655723206015.ms.tar

   This will unpack the two measurement sets:

	* 12A-195.sb12378614.eb13812032.56231.2797265625.ms
	
	* 12A-195.sb7351094.eb10491731.56075.655723206015.ms

   Both observations contain the following fields:

	+-----+----------+------------------------------------------+
	|Field|Field Name|Comments                                  |
	+=====+==========+==========================================+
	|0    |3C48      |Setup scan                                |
	+-----+----------+------------------------------------------+
	|1    |3C48      |Primary Calibrator                        |
	+-----+----------+------------------------------------------+
	|2    |J2340+2641|Phase Calibrator                          |
	+-----+----------+------------------------------------------+
	|3    |3C 465    |Target	                            |
	+-----+----------+------------------------------------------+
	|4    |J0313+4120|Phase Calibrator (unused in this tutorial)|
	+-----+----------+------------------------------------------+
	|5    |3C83.1B   |Target (unused in this tutorial)          |
	+-----+----------+------------------------------------------+

	
#. The A configuration observation was taken on the 31st October 2012 between 06:45 and 09:15. The B configuration image was taken on the 28th May 2012 between 15:44 and 18:13. The operators log for both observations can be downloaded from `here <http://www.vla.nrao.edu/cgi-bin/oplogs.cgi>`_.

#. Load the singularity module by entering:

	.. code-block:: console

		(host) $ module load singularity

#. Download the singularity image created in :ref:`use-of-singularity`. The following command downloads the most up to date image which is 1.3 GB and so this may take some time!

	.. code-block:: console

		(host) $ singularity pull library://mhardcastle/default/casa

Create the slurm script
-----------------------

#. The slurm script used in this tutorial is called :download:`VLA_Parallel_Processing.slurm <scripts/vla/VLA_Parallel_Processing.slurm>` and contains the following lines of code:

	.. code-block:: console

		#!/bin/bash
		#SBATCH -J VLA-Parallel-Processing
		#SBATCH -A DIRAC-TP001-CPU
		#SBATCH -p icelake
		#SBATCH --nodes=1
		#SBATCH --ntasks=2
		#SBATCH --time=03:00:00
		#SBATCH --mail-type=ALL
		#SBATCH --no-requeue

		#! Enter the script to run here
		. /etc/profile.d/modules.sh
		module load rhe18/default-icl
		module load singularity
		module load parallel

		# Describe the measurement sets to be processed
		INPUTS=("12A-195.sb12378614.eb13812032.56231.2797265625.ms A L" "12A-195.sb7351094.eb10491731.56075.655723206015.ms B L")

		# Set up the srun command
		# The -N1 -n1 options allocate a single core to each task
		srun="srun --exclusive -N1 -n1"

		# Set up the parallel command
		# The delay of 0.2 prevents overloading the controlling node
		# -j is the number of tasks to run simultaneously
		# --joblog and --resume combine to create a task log that can be used to monitor progress
		parallel="parallel --delay 0.2 -j $SLURM_NTASKS --joblog runtask.log --resume"
		
		# Run the command
		$parallel "$srun singularity exec casa_latest.sif casa -c VLA_Process_3C465_Images.py {1} ::: "${INPUTS[@]}"

Note the following points about the slurm script:

	* The command ``#SBATCH -J VLA-Parallel-Processing`` names the job VLA-Processing-Multiple-Images
	* The command ``#SBATCH -A DIRAC-TP001-CPU`` is the name of the project under which time has been allocated
	* The command ``#SBATCH -p icelake`` ensures we are using the icelake cluster
	* By default slurm allocates one cpu per task and so the commands ``#SBATCH --nodes=1``  and ``#SBATCH --ntasks=2`` combine to ask for two CPUs on a single node. On icelake each node has 76 CPUs with all CPUs on the same node sharing memory resources. Changing the nodes variable to 2 would have the effect of asking for two CPUs on different machines. Since the CPUs would no longer be sharing memory each task will run slightly quicker however the job is likely to take longer to schedule and is an inefficient use of resources.
	* The command ``#SBATCH --time=03:00:00`` is requesting 3 (wall-clock) hours of processing time.
	* The command ``#SBATCH --mail-type=ALL`` means email messages will be sent at the start and end of the job or (if applicable) when an error occurs. To disable this set the option to ``NONE``.
	* The command ``#SBATCH --no-requeue`` means that if this job is interrupted by a node failure/system downtime it will `not` be automatically rescheduled.
	* The command ``. /etc/profile.d/modules.sh`` enables the module command
	* The command ``module load rhe18/default-icl`` loads the basic environment needed by icelake
	* The two module load commands load the singularity and parallel modules
	* The `INPUTS` command defines a list of parameters that will be passed to the casa script. In this example the list is typed directly into the script but this could be altered to read the parameters from a file.
	* The ``srun --exclusive -n1 -N1`` allocates exclusive use of a single core to each task
	* The ``parallel --delay 0.2 -f $SLURM_NTASKS`` tells the parallel process that we are running ``ntasks`` parallel processes. In this case ntasks=2, so we are running two parallel processes.
	* The ``$parallel...`` command iterates through the ``INPUTS`` list calling the ``srun`` command for each element in the list. For each call of ``srun``, parallel replaces the placeholder `{1}` with the list element. The command ``srun`` uses the casa_latest.sif singularity to call the VLA_Process_3C465_Images.py script within casa, sending it the parameters within the `{1}` placeholder.

.. _Parallel-Processing-Create-the-casa-script:

Create the CASA script
----------------------

#. The casa script used in this tutorial is called :download:`VLA_Process_3C465_Images.py <scripts/vla/VLA_Process_3C465_Images.py>` and is based on the code used in the :ref:`VLA-basic-imaging-getting-started` tutorial. The download file contains the full script with a summarised version given below:

	.. code-block:: console

		from sys import argv

		params = argv[1].split()
		vis = params[0]
		config = params[1]
		band = params[2]

		smoothed_vis = vis[:-3]+'-smoothed.ms'
		primary_calibrator = '1'
		phase_calibrator = '2'
		target_field='3'
		refant = 'ea21'

		caltable_antpos = smoothed_vis[:-3]+".antpos"

		listobs(vis=vis, verbose=True, listfile=vis[:-3]+'.listobs')

		# Standard casa data flagging and calibration commands go here

		# Set up the variables used in imaging. The values depend upon the configuration
		if config=='A':
			cell=['0.25arcsec','0.25arcsec']
			imsize=[11250,11250]
			scales=[0,10,26]
		elif config=='B':
			cell=['1arcsec','1arcsec']
			imsize=[3072,3072]
			scales=[0,9,22]
		elif config=='C':
			cell=['3arcsec','3arcsec']
			imsize=[1024,1024]
			scales=[0,9,23]
		elif config=='D':
			cell=['10arcsec','10arcsec']
			imsize=[320,320]
			scales=[0,9,23]

		# Extract data used for imaging from the measurement set
		rms = stats['rms'][0]

		tclean(vis=smoothed_vis, field=target_field, imagename=smoothed_vis[:-3]+'-Clean', cell=cell, imsize=imsize, niter=20000, threshold=str(rms*5)+'Jy', stokes='I', deconvolver='multiscale', scales=scales, smallscalebias=0.9, weighting='briggs', robust=0.5, pbcor=True)

Note the following points about the casa script:

	* The ``params = argv[1].split()`` command imports the parameter string that was supplied by the call to parallel in the slurm script and splits it into its components. The next few lines populate the variables used throughout the script. In this example the name of the measurement set as well as the VLA configuration and band of the measurement set are all supplied. This could be expanded to include any additional information desired.
	* The ``listobs`` and ``tclean`` commands give a simple example of how the variables can be used within the script
	* The nested ``if`` block is an example of how to use the data to set up the variables used during imaging. This script only uses the ``config`` variable but this could easily be expanded to include additional variables such as ``band``.

Running the scripts
-------------------

#. Log on to the `Cambridge CSD3 system <https://docs.hpc.cam.ac.uk/hpc/index.html>`_ as described in :ref:`cambridgehpc-login`. 

#. If necessary download the casa singularity as described in :ref:`VLA-basic-imaging-getting-started`.

#. Run the slurm script by entering

	.. code-block:: console

		(host) $ sbatch VLA_Parallel_Processing.slurm

#. Check the casa `.log` and `runtask.log` files for any errors. An exit value of `1` in the `runtask.log` file indicates a terminal error occurred and the process was terminated prematurely.






.. _VLA-using-casa-plotms:

Using Casa's Plotms
===================

``plotms`` is a GUI tool supplied with casa for plotting visibility and calibration data. The HPC services offered by, for example, the `Cambridge CSD3 system
<https://docs.hpc.cam.ac.uk/hpc/index.html>`_, are intended to be used with batch submission scripts where GUI tools cannot be used. Fortunately, ``plotms`` can be called from the command line and used to save plots without using the GUI functionality. This tutorial describes how to use ``plotms`` as part of a batch submission script without using its GUI functionality. This allows users to exploit the full functionality of ``plotms``.

.. _VLA-using-casa-plotms-buiding-the-singularity-container:

Building the Singularity Container
----------------------------------

#. The basic casa singularity described in :ref:`use-of-singularity` must be modified in order to allow access to ``plotms`` from within a submission script. The following script, named :download:`casa_plotms.def <scripts/vla/casa_plotms.def>`, can be used to create a singularity container capable of running ``plotms`` as part of a batch submission script:

	.. code-block:: bash

		Bootstrap: docker
		From: scientificlinux/sl

		%post
		yum -y update
		yum -y upgrade
		yum -y install xorg-x11-server-Xvfb
		yum -y install wget perl less

		#Install casa dependencies
		yum -y install fontconfig freetype freetype-devel fontconfig-devel libstdc++

		#Install casa
		cd /usr/local
		wget https://casa.nrao.edu/download/distro/casa/release/rhel/casa-6.4.3-27-py3.8.tar.xz
		tar xf casa-6.4.3-27-py3.8.tar.xz
		rm casa-6.4.3-27-py3.8.tar.xz

		#The above casa installation contains three AppImage files
		#AppImages only work within singularity containers if they are unpacked
		#The following replaces the three packed AppImages with unpacked versions
		cd /usr/local/casa-6.4.3-27/lib/py/lib/python3.8/site-packages/casaplotms/__bin__/
		./casaplotms-x86_64.AppImage --appimage-extract
		chmod -R 755 /usr/local/casa-6.4.3-27/lib/py/lib/python3.8/site-packages/casaplotms/__bin__/squashfs-root/
		rm ./casaplotms-x86_64.AppImage
		ln -s ./squashfs-root/AppRun ./casaplotms-x86_64.AppImage

		cd /usr/local/casa-6.4.3-27/lib/py/lib/python3.8/site-packages/casaplotserver/__bin__/
		./casaplotserver-x86_64.AppImage --appimage-extract
		chmod -R 755 /usr/local/casa-6.4.3-27/lib/py/lib/python3.8/site-packages/casaplotserver/__bin__/squashfs-root/
		rm ./casaplotserver-x86_64.AppImage
		ln -s ./squashfs-root/AppRun ./casaplotserver-x86_64.AppImage

		cd /usr/local/casa-6.4.3-27/lib/py/lib/python3.8/site-packages/casaviewer/__bin__/
		./casaviewer-x86_64.AppImage --appimage-extract
		chmod -R 755 /usr/local/casa-6.4.3-27/lib/py/lib/python3.8/site-packages/casaviewer/__bin__/squashfs-root
		rm ./casaviewer-x86_64.AppImage
		ln -s ./squashfs-root/AppRun ./casaviewer-x86_64.AppImage

		%environment
		export LC_ALL=C
		export PATH=/usr/local/casa-6.4.3-27/bin:$PATH

		%runscript
		xvfb-run casa --nologger --log2term --nogui

		%labels
		Author IRIS-Radioastronomy

	Note the following points about the definition script:

		* The ``xorg-x11-server-Xvfb`` (X Virtual Frame Buffer) is an X server package that allows software that would normally require an X display to be run on machines with no display hardware. Rather than interacting with a monitor ``plotms`` interacts with this package.
		* When casa is installed, it also installs three AppImage files one of which, ``casaplotms-x86_64.AppImage``, is used to run ``plotms``. AppImage's require access to FUSE in order to run. However, using FUSE to mount a filesystem inside a container may undermine the security features offered by singularity. This script therefore adopts an alternative, safer, solution and unpacks/extracts the AppImage before redirecting any calls to the original AppImage to the unpacked version. Whilst strictly only the ``casaplotms`` AppImage is needed to run ``plotms`` this script unpacks all three AppImages for completeness.
		* The runscript command ``xvfb-run casa ...`` uses Xvfb to launch casa so that all calls to the display are redirected to Xvfb.

#. Using this script a singularity container, in this case named ``casa_plotms.sif``, can be built by entering the following command:

.. code-block:: console

	(host) $ singularity build --fakeroot casa_plotms.sif casa_plotms.def


.. _VLA-using-casa-plotms-create-the-slurm-script:

Create the slurm script
-----------------------

#. The slurm script used in this tutorial is called :download:`casa_plotms.slurm <scripts/vla/casa_plotms.slurm>` and contains the following lines of code:

	.. code-block:: bash

		#!/bin/bash
		#SBATCH -J Casa-Plotms
		#SBATCH -A DIRAC-TP001-CPU
		#SBATCH -p icelake
		#SBATCH --nodes=1
		#SBATCH --ntasks=1
		#SBATCH --time=00:45:00
		#SBATCH --mail-type=ALL
		#SBATCH --no-requeue

		#! Enter the script to run here
		. /etc/profile.d/modules.sh
		module load rhel8/default-icl
		module load singularity
		singularity exec casa_plotms.sif xvfb-run casa -c 3C391_script.py

This script is nearly identical to the one described in :ref:`VLA-basic-imaging`. The important difference is in the last line which executes a command within the singularity container ``casa_plotms.sif``. The execute command does not trigger the run script within the singularity and so the command ``xvfb-run casa ...`` is needed to use Xvfb to launch casa which in turn calls the casa script named 3C391_script.py. 

.. _VLA-using-casa-plotms-create-the-casa-script:

Create the CASA script
----------------------

#. The casa script used in this tutorial is named :download:`3C391_script.py <scripts/vla/3C391_script.py>`. This script follows the `VLA Continuum Tutorial 3C 391 <https://casaguides.nrao.edu/index.php?title=VLA_Continuum_Tutorial_3C391-CASA6.2.0>`_ to generate an image of the supernova remnant 3C 391. The measurement set used in this tutorial can be downloaded from the VLA tutorial website. Shown below are two of the calls made to ``plotms`` within the script:

	.. code-block:: python

		...

		plotms(vis=vis,selectdata=True,correlation='RR,LL',averagedata=True,avgchannel='64',coloraxis='field',showgui=False,plotfile='plotms_3c391-Time.png',highres=True)

		...

		plotms(vis='3c391_ctm_mosaic_10s_spw0.G0all',xaxis='time',yaxis='phase',coloraxis='corr',iteraxis='antenna',exprange='all',plotrange=[-1,-1,-180,180],showgui=False,plotfile='plotms_3c391-G0all-phase.png',highres=True)	

		...


	Note the following points about the casa script
		* Every call to ``plotms`` sets the argument ``showgui=False``. This is necessary in order for the script to work.
		* Every call to ``plotms`` sets the argument ``highres=True``. Setting this variable to False causes it to save the images using the screen resolution which fails causing it to revert to saving in high resolution. Setting the ``highres`` argument to True causes it to go straight to saving in high resolution saving time
		* If using the ``iteraxis`` variable, ``exprange`` must not be null. In the above example we have set ``exprange='all'`` which causes ``plotms`` to save one image for every ``iteraxis`` page.

.. _VLA-using-casa-plotms-running-the scripts:

Running the scripts
-------------------

#. Log on to the `Cambridge CSD3 system <https://docs.hpc.cam.ac.uk/hpc/index.html>`_ as described in :ref:`cambridgehpc-login`.

#. If necessary install the plotms-enabled singularity container described in :ref:`VLA-using-casa-plotms-buiding-the-singularity-container`

#. Run the slurm script by entering

	.. code-block:: console

		(host) $ sbatch casa_plotms.slurm

#. Check the casa `.log` and `runtask.log` files for any errors. An exit value of `1` in the `runtask.log` file indicates a terminal error occurred and the process was terminated prematurely.

Generating Single Plots
-----------------------

When generating a small number of images, rather than running a batch script it may be more efficient to generate images using command line prompts. To do this:

#. Log on to the `Cambridge CSD3 system <https://docs.hpc.cam.ac.uk/hpc/index.html>`_ as described in :ref:`cambridgehpc-login`.

#. If necessary install the plotms-enabled singularity container described in :ref:`VLA-using-casa-plotms-buiding-the-singularity-container`

#. Run the plotms-enabled singularity container by entering

	.. code-block:: console
	
		(host) $ run singularity casa_plotms.sif

#. This will load the singularity and start casa automatically returning the casa command prompt. In order to generate an image enter a ``plotms`` command making sure to set ``showgui=False`` and setting the argument ``plotfile`` to the name of the file you wish to create. If using the ``iteraxis`` argument it is advisable to also set the ``exprange`` argument to tell plotms which of the iteraxis pages you wish to be saved. For example:

	.. code-block:: console

		(CASA) $ plotms(vis='3c391_ctm_mosaic_10s_spw0.G0',xaxis='time',yaxis='phase',coloraxis='corr',field='J1331+3030',iteraxis='antenna',exprange='all',plotrange=[-1,-1,-180,180],timerange='08:02:00~08:17:00',showgui=False,plotfile='plotms_3c391-G0-phase.png',highres=True)










