# container-wrf - ARM version (and x86 too)
#
containerized WRF for single-node modeling runs, classroom teaching and training examples.

Based on the NCAR/wrf-container bigwxwrf [https://github.com/NCAR/container-wrf/] set of docker images

This version was specifically modified to compile and run in a Raspberry Pi (tested with RPi 3 and Raspbian Jessie), thanks to the help of [http://supersmith.com/site/ARM_files/wrf_on_arm.pdf] and [https://www.raspberrypi.org/forums/viewtopic.php?t=19248]. A companion image has been generated to x86 architectures.

The latest version (:latest) was migrated to Ubuntu 16.04 in order to simplify the compiling deployment.

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
 
To run in a single machine, you can use the following basic command:


```sh
docker pull lsteffenel/wrf-container-armv7l
docker run -it -v /PATH_TO/WPS_GEOG:/WPS_GEOG -v /PATH_TO/YOUR_DATA:/wrfinput -v /PATH_TO/WRFOUTPUT:/wrfoutput wrf-container-armv7l /root/run-wrf
```

Or use it in a docker swarm environment:
```sh
docker swarm init (...)
docker stack deploy -c docker-compose.yml mywrf
ssh ssh -o ServerAliveInterval=30 -o StrictHostKeyChecking=no root@localhost -p 2022
## password = meso
docker-master:/root# ./run-wrf [options]
```
### How to use run-wrf parameters: 

* simple run on localhost (WPS+real+WRF) - 4 cores, 4 processes
```# ./run-wrf -hosts localhost:4 -np 4```

* Just real + WRF (met* files from WPS) - 4 cores, 4 processes (don't need to have /WPS_GEOG)
```# ./run-wrf -skip wps -hosts localhost:4 -np 4```

* Just WRF (met* files from WPS and wrfbdy* + wrfinput* from real in /wrfinput) - 2 cores, 2 processes
```# ./run-wrf -skip wps -skip real -hosts localhost:2 -np 2```

* Docker Swarm: 
  * On the Docker swarm mode, auto discovery of nodes (and #of cores) can be enable with -hosts auto (default option)
```# ./run-wrf [-hosts auto] -np 12```
  * Otherwise, one can detail each machine as usual 
```# ./run-wrf -hosts machine1:2,machine2:2 -np 4```
