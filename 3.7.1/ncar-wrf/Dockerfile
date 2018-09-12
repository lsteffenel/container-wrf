from centos
MAINTAINER John Exby <exby@ucar.edu>
# 
# This Dockerfile compiles WRF from source during "docker build" step
ENV WRF_VERSION 3.7.1
RUN curl -SL https://ral.ucar.edu/sites/default/files/public/projects/ncar-docker-wrf/ucar-bsd-3-clause-license.pdf > /UCAR-BSD-3-Clause-License.pdf
#

#gcc gfortran g++ mpich  

COPY CentOS-armhfp-kernel.repo /etc/yum.repos.d/

RUN yum -y update \
  && yum -y install file gcc gcc-gfortran gcc-c++ glibc.i686 libgcc.i686 libpng-devel jasper jasper-devel hostname m4 make perl \ 
  tar tcsh time wget which zlib zlib-devel openssh-clients openssh-server net-tools epel-release \
  && yum clean all
#
# now get 3rd party EPEL builds of netcdf and openmpi dependencies
RUN yum -y install netcdf-openmpi-devel netcdf-fortran-openmpi-devel netcdf-fortran-openmpi hdf5-openmpi openmpi openmpi-devel \
  && yum clean all
#
WORKDIR /wrf
#
# Download original sources
#
RUN curl -SL http://www2.mmm.ucar.edu/wrf/src/WRFV$WRF_VERSION.TAR.gz | tar zxC /wrf \
 && curl -SL http://www2.mmm.ucar.edu/wrf/src/WPSV$WRF_VERSION.TAR.gz | tar zxC /wrf
#
# Set environment for interactive container shells
#
RUN echo export LDFLAGS="-lm" >> /etc/bashrc \
 && echo export NETCDF=/soft/netcdf/3.6.3 >> /etc/bashrc \
 && echo export JASPERINC=/usr/include/jasper/ >> /etc/bashrc \
 && echo export JASPERLIB=/usr/lib/ >> /etc/bashrc \
 && echo export LD_LIBRARY_PATH="/usr/lib/openmpi/lib" >> /etc/bashrc \
 && echo export PATH="/usr/lib/openmpi/bin:$PATH" >> /etc/bashrc \
 && echo setenv LDFLAGS "-lm" >> /etc/csh.cshrc \
 && echo setenv NETCDF "/soft/netcdf/3.6.3" >> /etc/csh.cshrc \
 && echo setenv JASPERINC "/usr/include/jasper/" >> /etc/csh.cshrc \
 && echo setenv JASPERLIB "/usr/lib/" >> /etc/csh.cshrc \
 && echo setenv LD_LIBRARY_PATH "/usr/lib/openmpi/lib" >> /etc/csh.cshrc \
 && echo setenv PATH "/usr/lib/openmpi/bin:$PATH" >> /etc/csh.cshrc
#
#
ENV LD_LIBRARY_PATH /usr/lib/openmpi/lib
ENV PATH  /usr/lib/openmpi/bin:$PATH
#
#
# set up ssh configuration
COPY ssh_config /root/.ssh/config
RUN mkdir -p /root/.openmpi
COPY default-mca-params.conf /root/.openmpi/mca-params.conf
RUN mkdir -p /var/run/sshd \
    && ssh-keygen -A \
    && sed -i 's/#PermitRootLogin yes/PermitRootLogin yes/g' /etc/ssh/sshd_config \
    && sed -i 's/#RSAAuthentication yes/RSAAuthentication yes/g' /etc/ssh/sshd_config \
    && sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/g' /etc/ssh/sshd_config \
    && ssh-keygen -f /root/.ssh/id_rsa -t rsa -N '' \
    && chmod 600 /root/.ssh/config \
    && chmod 700 /root/.ssh \
    && cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys
#
# Get and build netcdf first
#
RUN wget ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-3.6.3.tar.gz \
   && gzip -c -d ./netcdf-3.6.3.tar.gz | tar -xo \
   && cd netcdf-3.6.3 \
   && ./configure --prefix=/soft/netcdf/3.6.3 \
   && make && make check && make install && cd ..
#
# Build WRF first
# 
## input 75 and 1 to configure script alternative line = && echo -e "75\r1\r" | ./configure
RUN mkdir netcdf_links \
## && ln -sf /usr/include/openmpi-arm/ netcdf_links/include \
## && ln -sf /usr/lib/openmpi/lib netcdf_links/lib \
## && export NETCDF=/wrf/netcdf_links \
 && export NETCDF=/soft/netcdf/3.6.3/ \
 && export JASPERINC=/usr/include/jasper/ \
 && export JASPERLIB=/usr/lib/ \
 && export WRFIO_NCD_LARGE_FILE_SUPPORT=1 \
 && cd ./WRFV3 \
 && sed -i -e 's/#ARCH    Linux x86_64 ppc64le, gfortran/#ARCH    Linux armv7l ppc64le, gfortran/' arch/configure_new.defaults \
 ##&& ./configure <<< $'75\r1\r' \
 && ./configure <<< $'3\r1\r' \
 && sed -i -e '/^DM_CC/ s/$/ -DMPI2_SUPPORT/' ./configure.wrf \
 && /bin/csh ./compile em_real <<< y
#
# Build WPS second
#
# input 1 to configure script
RUN cd ./WPS \
 && export NETCDF=/soft/netcdf/3.6.3/ \
 && export JASPERINC=/usr/include/jasper/ \
 && export JASPERLIB=/usr/lib/ \
 && sed -i -e 's/#ARCH    Linux x86_64, gfortran/#ARCH    Linux armv7l, gfortran/' arch/configure.defaults \
 && ./configure <<< $'1\r' \
 ##&& sed -i -e 's/-L$(NETCDF)\/lib/-L$(NETCDF)\/lib -lnetcdff /' ./configure.wps \
 && /bin/csh ./compile
#
# copy in a couple custom scripts
COPY run-wrf /wrf
COPY docker-clean /wrf
RUN chmod +x /wrf/run-wrf \
 && chmod +x /wrf/docker-clean
#
RUN /wrf/docker-clean 
#
VOLUME /wrf
CMD ["/wrf/run-wrf"]
