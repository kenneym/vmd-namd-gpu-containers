# Base image provides cuda 9.1 support and opengl rendering for glx and egl
# If you posess a older or newer version of cuda, feel free to change the 
# base image by consulting the nvidia/cudagl repo on Dockerhub
FROM nvidia/cudagl:9.1-devel-centos7

# Install basic packages:
RUN yum install -y \
      autoconf \
      make \
      automake \
      libtool \
      git \
      rsync \
      rlwrap \
      vim \
      csh \
      wget \
      && rm -rf /var/cache/yum*

# Install X11 packages and VMD dependencies
RUN yum install -y \

      # Used to render GLX graphics through the contianer:
      libXcomposite \
      libXinerama \
      libXi \
      xauth \ 
      xorg-x11-devel \
      \
      # Used to plot FEP data for analysis:
      xmgrace \
      ImageMagick \
      && rm -rf /var/cache/yum/*

# Provide NVIDIA driver all neccessary functions:
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility,graphics,video,compat32,display

# Copy in VMD and create symbolic link for convenience:
ADD vmd-1.9.4a12.bin.LINUXAMD64-CUDA9-OptiX411-OSPRay140.opengl.tar.gz /opt 
RUN ln -s  /opt/vmd-1.9.4a12 /opt/vmd

# Copy in test files unless declined by user
# Use --env test=F to decline
WORKDIR /

RUN echo $PWD

ARG test=False

COPY tests ./tests

RUN echo `ls`

RUN if [ $test = "True" ] ; \
    then echo "Tests activated" ; \
    else echo "Tests not activated" && rm -r tests ; \
    fi

# Add a bug fix to use FEP on Linux:
WORKDIR /opt/vmd/plugins/noarch/tcl/parsefep2.0
RUN echo `ls`
RUN sed -i '1845s/$/]/' ./parseFEP.tcl  \
    && sed -i '1846s/$/]/' ./parseFEP.tcl

# Install VMD binary:
WORKDIR /opt/vmd/

RUN echo `ls`

RUN ./configure \
    && cd src \
    && make install 

# Drop the user at the workspace directory
RUN mkdir /workspace \
    && cd /workspace

# Start VMD when the container is launched, by default
CMD ["vmd"]
