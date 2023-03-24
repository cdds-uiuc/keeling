# Porting CESM

(work in progress)

## Checking out and downloading the model

Make a directory for your CESM models: `mkdir CESM`

`cd CESM`

Checking out the model: `svn co https://svn-ccsm-models.cgd.ucar.edu/cesm1/release_tags/cesm1_2_1 cesm1_2_1`
- This downloads cesm1_2_1 to a directory called cesm1_2_1 in the current directory.
- When prompted for password for `'<keeling_account>':` put in your password for your keeling account.
- When prompted for username afterwards, input "guestuser" and password "friendly". This will happen a couple times.
- The system will then ask "Store password unencrypted?". Type "yes". This will occur several times.

When this message pops up, type "p":
```
> Error validating server certificate for 'https://svn-homme-model.cgd.ucar.edu:443':
 - The certificate is not issued by a trusted authority. Use the fingerprint to validate the certificate manually!
Certificate information:
 - Hostname: *.cgd.ucar.edu
 - Valid: from Wed, 17 Nov 2021 00:00:00 GMT until Sun, 18 Dec 2022 23:59:59 GMT
 - Issuer: InCommon, Internet2, Ann Arbor, MI, US
- Fingerprint: 35:da:70:4d:b6:cf:01:a3:fc:c2:63:6f:eb:15:de:81:fc:18:69:e3

- > (R)eject, accept (t)emporarily or accept (p)ermanently? p
 ```

### Checking the download was successful

The following should be in cesm1_2_1:
- ChangeLog
- ChangeLog_template
- Copyright
- README
- SVN_EXTERNAL_DIRECTORIES
- models
- scripts
- tools

In scripts:
- ChangeLog
- README
- SVN_EXTERNAL_DIRECTORIES
- ccsm_utils
- create_clone
- create_newcase
- create_test
- doc
- link_dirtree
- query_tests
- sample_pes_file.xml
- validation_testing

In models:
- atm
- csm_share
- dead_share
- drv
- glc
- ice
- lnd
- ocn
- rof
- utils
- wav

In tools:
- cprnc
- mapping

## Creating a new case

In $HOME/CESM/cesm1_2_1/scripts: `./create_newcase -case test1 -res f45_g37 -compset X -mach userdefined`
- -case test1 - This sets the name of the case to test1
- -res f45_g37 - This is the resolution of the model
- -compset X - We're using the x component set, which is obsolete but easy to run
- -mach userdefined - Keeling isn't recognized by CESM, so we need to use userdefined.

`cd test1`

Check for what we need to run the model: `./cesm_setup`
This error should pop up:
```
Use of qw(...) as parentheses is deprecated at ./cesm_setup line 252.
ERROR: must set xml variable OS to generate Macros file
ERROR: must set xml variable MPILIB to build the model
ERROR: must set xml variable RUNDIR to build the model
ERROR: must set xml variable DIN_LOC_ROOT to build the model
ERROR: must set xml variable COMPILER to build the model
ERROR: must set xml variable EXEROOT to build the model
ERROR: must set xml variable MAX_TASKS_PER_NODE to build the model
Correct above and issue cesm_setup again 
```
This involves going into each of the following xml files and fixing the highlighted variables.

This step also requires you to make some new directories, which for our purposes will all be in a new directory called `$HOME/a/CESM_DATA`.

Use `mkdir <directory_name>` to create the following subdirectories of CESM_DATA:
- run - A run directory
- CESM_INPUT_DATA - For input data
- CESM_EXE_ROOT - Where the model will be run.

### env_build.xml
- OS - LINUX
- MPILIB - openmpi
- COMPILER - intel
- EXEROOT - /data/keeling/a/(illinoisid)/a/CESM_DATA/CESM_EXE_ROOT

### env_case.xml
No need to change anything.

### env_mach_pes.xml
In order to edit the file, the following needs to be run.

`./cesm_setup -clean`

`./cesm_setup` Then you can edit.

- MAX_TASKS_PER_NODE - 8

### env_run.xml
- RUNDIR - /data/keeling/a/(illinoisid)/a/CESM_DATA/run
- DIN_LOC_ROOT - /data/keeling/a/(illinoisid)/a/CESM_DATA/CESM_INPUT_DATA

#### (Optional) Short term archiving for output data   
Normally, the model output will go in the /run directory. However, if you'd like the output to be more organized, you can activate short term archiving, which organizes the output by model in different subdirectories.

You will need an output directory. Here, I'll be creating a new directory in `$HOME/a/CESM_DATA` called CESM_OUTPUT_DATA.

In env_run.xml, set the following:
- DOUT_S - TRUE
- DOUT_S_ROOT - /data/keeling/a/(illinoisid)/a/CESM_DATA/CESM_OUTPUT_DATA

When all the variables are put in, try `./cesm_setup` again.
There should be new files/directories in your test1 directory:
- CaseDocs (directory)
- Macros
- env_derived
- test1.run
- user_nl_cpl

## Changing Macros

Make these two edits to Macros:
- SLIBS+=$(shell $(NETCDF_PATH)/bin/nc-config --flibs)
- NETCDF_PATH:= /sw/netcdf4-4.7.4-intel-15.0.3


## Modules

In order to have all the modules active and running, write the following lines into a file called $HOME/.modules7
```
module load intel/intel-15.0.3
module load intel/hdf5-1.10.6-intel-15.0.3
module load intel/netcdf4-4.7.4-intel-15.0.3
module load intel/openmpi-4.1.2-intel-15.0.3 
```
You might need to log into Keeling again for these to work.

## Outdated genf90 and pio libraries

Some of the libraries have dead links due to Googlecode going offline. 

Go into `$HOME/CESM/cesm1_2_1/tools/cprnc` and in `SVN_EXTERNAL_DIRECTORIES`  
Remove this line:   
`genf90     http://parallelio.googlecode.com/svn/genf90/trunk_tags/genf90_140121`   
Add this line:  
`genf90    https://github.com/PARALLELIO/genf90/tags/genf90_140121` 

Run 
`svn propset svn:externals -F SVN_EXTERNAL_DIRECTORIES . `  
`svn update`

Go back up to the main directory:  
`cd cesm1_2_1`  
In `SVN_EXTERNAL_DIRECTORIES`:   
Remove this line:  
`models/utils/pio      http://parallelio.googlecode.com/svn/trunk_tags/pio1_8_12/pio`   
Add this line:  
`models/utils/pio     https://github.com/NCAR/ParallelIO.git/tags/pio1_7_2/pio `

Run   
`svn propset svn:externals -F SVN_EXTERNAL_DIRECTORIES . `  
`svn update`  


## Building the case

Now, run the following the build your case.  
`cd cesm1_2_1/scripts/test1/`  
`./test1.build`  

If you receive an error or need to fix anything, run `./test1.clean_build` before building again.

## Running the case

In your test1 case directory, there should be a test1.run file. Add the following lines right under the first USERDEFINED category.
```
#SBATCH --job-name=test1
#SBATCH --partition=sesempi
#SBATCH --nodes=2
#SBATCH --ntasks=16
#SBATCH --time=1-00:00:00
#SBATCH --mem-per-cpu=5g
#SBATCH --constraint=j48
#       --mail-type=BEGIN
#SBATCH --mail-type=FAIL
#SBATCH --mail-type=END
#SBATCH --mail-user=(email of your choice)
#
```   
After the second USERDEFINED entry, remove the hash mark from the mpirun line so it looks like below:    
`mpirun -np 16 $EXEROOT/cesm.exe >&! cesm.log.$LID`

Save your changes and run the model! `sbatch test1.run`   
You should receive a batch job number.

Within the next minute or so, you should receive an email of with a title of the following format:   
Slurm Job_id=(job id) Name=test1 Ended, Run time (run time), (COMPLETED or FAILED), ExitCode (exit code)

If the job failed, look in the CaseStatus file and any build logs, especially the CESM build logs.    
If the job was successful, all the build logs should be gzip files.   
Timing info is in the timing directory in both your run and case directory.   

## Output

Any output from the model should be in your run directory.   
The main output you may want to access should be in the NetCDF file.
