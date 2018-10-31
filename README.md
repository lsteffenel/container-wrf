# container-wrf - ARM version
#
containerized WRF for single-node modeling runs, classroom teaching and training examples.

Based on the bigwxwrf set of docker images

This version was specifically modified to compile and run in a Raspberry Pi (tested with RPi 3 and Raspbian Jessie), thanks to the help of [http://supersmith.com/site/ARM_files/wrf_on_arm.pdf] and [https://www.raspberrypi.org/forums/viewtopic.php?t=19248].

The centos version (:centos) is based on the original bigxwrf Dockerfile.
The latest version (:latest) was migrated to Ubuntu 16.04 in order to better support kubernetes/cluster deployment.

Only the wrf executables were modified, you can keep using the wps_geog and data samples from bigwxwrf.


NCAR developed WRF in Docker containers, related code and data sets for community exploration purposes.
Unsupported by UCAR/NCAR staff, yet community discussions and feedback will be channeled through google group or here on github soon.
(C) UCAR 2017. Developed at the Research Applications Laboratory at National Center for Atmospheric Research.

ARM version developed for the **CAPES-Cofecub project MESO - Modeling and forecasting the Secondary Effects of the Antarctic Ozone Hole** [http://meso.univ-reims.fr].
(C) URCA 2018. Developed by Luiz Angelo Steffenel at CReSTIC Laboratory.
