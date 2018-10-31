# libjasper-dev and others are no longer available in 18.04
from arm32v7/ubuntu:16.04
MAINTAINER Angelo Steffenel <angelo.steffenel@univ-reims.fr>
# 
#

#gcc gfortran g++ mpich  

RUN apt-get -y update \
 && apt-get install -y csh curl wget vim openssh-server gfortran m4 mpich libhdf5-mpich-dev \
 && apt-get install -y libpng-dev libjasper-dev libnetcdff-dev libnetcdf-dev netcdf-bin ncl-ncarg mlocate \
    && apt-get clean
#
# This Dockerfile compiles WRF from source during "docker build" step
ENV WRF_VERSION 3.9.1.1
ENV WPS_VERSION 3.9.1
RUN curl -SL https://ral.ucar.edu/sites/default/files/public/projects/ncar-docker-wrf/ucar-bsd-3-clause-license.pdf > /UCAR-BSD-3-Clause-License.pdf

WORKDIR /wrf
#
# Download original sources
#
RUN curl -SL http://www2.mmm.ucar.edu/wrf/src/WRFV$WRF_VERSION.TAR.gz | tar zxC /wrf \
 && curl -SL http://www2.mmm.ucar.edu/wrf/src/WPSV$WPS_VERSION.TAR.gz | tar zxC /wrf

RUN updatedb && locate jasper
#
# Set environment for interactive container shells
#
RUN echo export LDFLAGS="-lm" >> /etc/bashrc \
 && echo export NETCDF="/usr" >> /etc/bashrc \
 && echo export WRFIO_NCD_LARGE_FILE_SUPPORT=1 >> /etc/bashrc \
 && echo export LD_LIBRARY_PATH="/usr/lib/mpich/lib" >> /etc/bashrc \
 && echo export JASPERINC=/usr/include/ >> /etc/bashrc \
 && echo export JASPERLIB=/usr/lib/arm-linux-gnueabihf/ >> /etc/bashrc \
 && echo export PATH="/usr/lib/mpich/bin:$PATH" >> /etc/bashrc \
 && echo setenv LDFLAGS "-lm" >> /etc/csh.cshrc \
 && echo setenv NETCDF "/usr" >> /etc/csh.cshrc \
 && echo setenv WRFIO_NCD_LARGE_FILE_SUPPORT 1 >> /etc/csh.cshrc \
 && echo setenv LD_LIBRARY_PATH "/usr/lib/mpich/lib" >> /etc/csh.cshrc \
 && echo setenv JASPERINC "/usr/include/" >> /etc/csh.cshrc \
 && echo setenv JASPERLIB "/usr/lib/arm-linux-gnueabihf/" >> /etc/csh.cshrc \
 && echo setenv PATH "/usr/lib/mpich/bin:$PATH" >> /etc/csh.cshrc \
 && echo setenv J \"-j 2\" >> /etc/csh.cshrc \
 && export LDFLAGS="-lm" \
 && export NETCDF="/usr" \
 && export WRFIO_NCD_LARGE_FILE_SUPPORT=1 \
 && export LD_LIBRARY_PATH="/usr/lib/mpich/lib" \
 && export JASPERINC="/usr/include/" \
 && export JASPERLIB="/usr/lib/arm-linux-gnueabihf/"  \
 && export PATH="/usr/lib/mpich/bin:$PATH"  

#
 ENV LD_LIBRARY_PATH /usr/lib/mpich/lib
 ENV PATH  /usr/lib/mpich/bin:$PATH
#
#
# Get and build netcdf first
#
# RUN wget ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-3.6.3.tar.gz \
#    && gzip -c -d ./netcdf-3.6.3.tar.gz | tar -xo \
#    && cd netcdf-3.6.3 \
#    && ./configure --prefix=/soft/netcdf/3.6.3 \
#    && make && make check && make install && cd ..
#
# Build WRF first
# 
#RUN source /etc/bashrc
COPY wrf_options ./WRFV3/wrf_options
COPY wps_options ./WRFV3/wps_options
  ##&& ./configure <<< $'75\r1\r' \
  #&& ./configure <<< $'3\r1\r' 
  RUN cd ./WRFV3 \
  && sed -i -e 's/#ARCH    Linux x86_64 ppc64le, gfortran/#ARCH    Linux armv7l ppc64le, gfortran/' arch/configure_new.defaults  
  RUN export LDFLAGS="-lm" \
  && export NETCDF="/usr" \
  && export WRFIO_NCD_LARGE_FILE_SUPPORT=1 \
  && export LD_LIBRARY_PATH="/usr/lib/mpich/lib" \
  && export JASPERINC="/usr/include/" \
  && export JASPERLIB="/usr/lib/arm-linux-gnueabihf/"  \
  && export PATH="/usr/lib/mpich/bin:$PATH"  \
  && cd ./WRFV3 && ./configure < wrf_options \
  && sed -i -e '/^DM_CC/ s/$/ -DMPI2_SUPPORT/' ./configure.wrf \
  && sed -i 's/-lwrfio_nf/-lwrfio_nf -lnetcdf -lnetcdff/' ./configure.wrf \
  && echo y > wrf_comp_options  \
  && /bin/csh ./compile em_real < wrf_comp_options > compile.log
# #
# # Build WPS second
# #
# # input 1 to configure script
 RUN cd ./WPS \
  && export LDFLAGS="-lm" \
  && export NETCDF="/usr" \
  && export WRFIO_NCD_LARGE_FILE_SUPPORT=1 \
  && export LD_LIBRARY_PATH="/usr/lib/mpich/lib" \
  && export JASPERINC="/usr/include/" \
  && export JASPERLIB="/usr/lib/arm-linux-gnueabihf/"  \
  && export PATH="/usr/lib/mpich/bin:$PATH"  \
  && sed -i -e 's/#ARCH    Linux x86_64, gfortran/#ARCH    Linux armv7l, gfortran/' arch/configure.defaults \
  && echo 1 > wps_comp_options \
  && ./configure < wps_comp_options \
  && sed -i -e 's/-L$(NETCDF)\/lib/-L$(NETCDF)\/lib -lnetcdff -lnetcdf /' ./configure.wps \
  && sed -i 's/FFFFFFFF00000000/FFFFFFFF0000000/' ./ungrib/src/ngl/g2/intmath.f \
  && /bin/csh ./compile
# #

# set up ssh configuration
COPY ssh_config /root/.ssh/config
# RUN mkdir -p /root/.openmpi
# COPY default-mca-params.conf /root/.openmpi/mca-params.conf
RUN mkdir -p /var/run/sshd \
    && ssh-keygen -A \
    && sed -i 's/#PermitRootLogin yes/PermitRootLogin yes/g' /etc/ssh/sshd_config \
    && sed -i 's/#RSAAuthentication yes/RSAAuthentication yes/g' /etc/ssh/sshd_config \
    && sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/g' /etc/ssh/sshd_config \
    && ssh-keygen -f /root/.ssh/id_rsa -t rsa -N '' \
    && chmod 600 /root/.ssh/config \
    && chmod 700 /root/.ssh \
    && cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys 
RUN echo 'root:meso' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile
#



# # copy in a couple custom scripts
 COPY run-wrf /wrf
 COPY docker-clean /wrf
 RUN chmod +x /wrf/run-wrf \
  && chmod +x /wrf/docker-clean
# #
 RUN /wrf/docker-clean 
# #
 VOLUME /wrf
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
# CMD ["/wrf/run-wrf"]
