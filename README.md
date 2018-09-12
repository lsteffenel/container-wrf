# container-wrf - ARM version
#
containerized WRF for single-node modeling runs, classroom teaching and training examples.

Based on the NCAR/wrf-container bigwxwrf [https://github.com/NCAR/container-wrf/] set of docker images

This version was specifically modified to compile and run in a Raspberry Pi (tested with RPi 3 and Raspbian Jessie), thanks to the help of [http://supersmith.com/site/ARM_files/wrf_on_arm.pdf] and [https://www.raspberrypi.org/forums/viewtopic.php?t=19248].

Only the wrf executables were modified, you can keep using the wps_geog and data samples from bigwxwrf.

ARM version developed for the **CAPES-Cofecub project MESO - Modeling and forecasting the Secondary Effects of the Antarctic Ozone Hole** [http://meso.univ-reims.fr]. Developed by Luiz Angelo Steffenel at CReSTIC Laboratory, URCA (2018).


## Hints for running on Raspberry Pi 3

* Raspberry has small memory and storage capabilities. Please consider adding an external USB drive for storage and swap.
* bigwxwrf/wps_geog is only a reduced subset of WPS_GEOG. You can have your own WPS_GEOG directory, mounted at /WPS_GEOG as shown below.
* Besides the sample datasets from bigwxwrf (Sandy and Katrina), you can mount your own dataset directory on /wrfinput. Just pay attention to include the following files inside
  * grib2 data files from GFS
  * Vtable.GFS
  * namelist.wps
  * namelist.input - be sure to put ``history_outname``pointing to ``/wrfoutput``
 
To run, you can use the following basic command:

```sh
docker pull lsteffenel/wrf-container-armv7l
docker run -it -v /PATH_TO/WPS_GEOG:/wps_geog -v /PATH_TO/YOUR_DATA:/wrfinput -v /PATH_TO/WRFOUTPUT:/wrfoutput wrf-container-armv7l /wrf/run-wrf
```
