# Cuda-Enabled NAMD for Multicore:
FROM nvidia/cudagl:9.1-devel-centos7

# Install Basic Packages & libraries:
RUN yum install -y \
wget \
vim \
git \
csh \
rlwrap \
perl \
redhat-lsb \
lsof \
libxml2-python \
&& rm -rf /var/cache/yum*

# Copy in latest version of CUDA NAMD for multicore
# (ADD and .tar.gz file used for faster uploads and downloads to remote git
# repos)
ADD NAMD_2.13_Nightly_64bit-August2018-CUDA-9.1.tar.gz ./opt

# Rename directory for ease-of-use
RUN mv /opt/NAMD_Git-2018-08-04_Linux-x86_64-multicore-CUDA /opt/namd

# Hard coded default
ARG test=True

COPY examples /benchmarks

RUN if [ "$test" == "True" ] ; then echo "Tests activated" ; \
    else echo "Tests not activated" && rm -r /benchmarks ; fi

# Setting LD library path for 32-bit (lib) and 64-bit (lib64) systems is
# absolutely essential for cuda to run properly
ENV LD_LIBRARY_PATH=/opt/namd/:/usr/local/cuda/lib64/

ENV PATH=/opt/namd/:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin 

CMD ["bin/bash"]
