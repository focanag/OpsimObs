OpsimObs
========

Code to recreate opsim observing conditions (seeing, clouds, skybrightness, 5-sigma limit) for a user-defined set of pointings.



This code is intended to replicate (more or less) the observing conditions that would come from the LSST Operations Simulation software, if that software was given the same MJD/RA/Dec of observations as you can pass to the code here. There are some differences: 
 - the code here interpolates between the recorded measurements for seeing and cloud in the relevant observational history files (SeeingPachon.txt and TololoCloud.txt) while the opsim code just reports the nearest recorded value. Since most of the observations tested with code are likely to be close in time (DD sequences), I think the interpolation is more useful. 
 - there are some minor differences in the reported Sun Alt/Az and Moon Alt/Az values. I have not been able to track down the cause of these, but the differences are pretty minor so should not cause a large difference in overall simulation characteristics. 
 - the code here calculations the 5sigma limiting magnitudes using a different method than the OpSim is currently using (they will be moving to this method in the future). This means that the 5sigma values are a bit different (and generally, a bit deeper than opsim values at the moment). Keep an eye on this, but I think these are more correct. 

Prerequisites for this code: 
You must have numpy and pyslalib installed. 
Pyslalib is available from https://github.com/scottransom/pyslalib
You must also have the LSST package 'throughputs' installed. You can download this at http://dev.lsstcorp.org/cgit/LSST/sims/throughputs.git/snapshot/throughputs-1.2.tar.gz (or you can git clone the package if you are familiar with that). Note that you do not need to have the LSST software stack installed to use the throughputs package; you can just source the throughputs.csh or throughput.sh file in throughputs-1.2/ups (if you do have the LSST software stack installed, just do the usual eups declare for the package and then you can 'setup throughputs'). 

How to use this code: 

Before you run the code for the first time in a shell: source throughputs-1.2/ups/throughputs.csh (or .sh) to set various environment variables, specifically LSST_THROUGHPUTS_DEFAULT 

Then:
python opsimObs.py inputobs_file [override_file]

- make an input observation file (such as test.input) which contains the RA (deg) / Dec (deg) / Date (MJD) / Filter (ugrizy) / ExpTime (seconds) 
  ... a note on the ExpTime: this can be for the entire visit (as in the example) or for a single exposure. If you are giving single exposures, change the value of 'nexp_visit' in opsimSky.conf to 1; if you are using visits with 2 exposures, use 'nexp_visit' = 2. 
- the default parameters are currently set to assume that each line in the input file is equivalent to the visit records from opsim; that is, each 'visit' (== line of input file) is two exposures, that implicitly includes shutter time for two exposures, readout time for one exposure, and a single slew/settle time (which inclues the second readout) within the reported 'exptime'. If you want to override these parameters to specify single exposures where the open shutter time is given as 'ExpTime', you can: just edit the opsimSky.conf file (set nexp_visit=1, add_shutter=1, add_slew=0 if you want no dither, 1 if you want to dither after each exposure), and then specify your override file when starting the code. 


What does the code do: 

First it reads the default configuration values, then overrides these are needed if an override_file is specified (note that you can override any of the parameters specified in setDefaultConfigs in opsimObs.py using the keyword in the config dictionary). 
Then it reads the weather history files for seeing and clouds and stores these in memory, as well as the Downtime (both scheduled and unscheduled) history. 
Next it reads your observation history file, and does a preliminary test to see if the observations are 'reasonable' - that is, it checks to be sure that you have not asked for too many filters in a single night (LSST only holds 5 at a time), checks to see that you have not requested observations spaced too closely in time to permit shutter open/close, readout, slew, and filter changes. 
Then the code calculates the weather (seeing and clouds) conditions at all the times of observations, as well as checking the downtime status. 
Next it calculates the position of the Sun and Moon for all observations. 
With these pieces of information, it then calculates the expected sky brightness and resulting 5-sigma limiting magnitude for each observation. 
Finally all of these pieces of information are printed to the screen. Note that observation conditions are reported even if the telescope would have been in 'Downtime', but the downtime status column will be set to '1'. 


Questions or comments, please let me know, especially as I have not been able to test this code under all conditions. 
This code should be considered 'under development', as it is likely to change to respond to requests from the LSST Deep Drilling Interest Group or other science collaboration requests. 

