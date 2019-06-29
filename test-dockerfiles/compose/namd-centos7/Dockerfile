# Standard NAMD for FEP on Multicore(no cuda):
FROM centos:7

# Install Basic Packages & libraries:
RUN yum install -y \
    wget \
    vim \
    git \
    csh \
    rlwrap \
    perl\
    redhat-lsb \
    lsof \
    libxml2-python \
    && rm -rf /var/cache/yum*

# Copy in latest version of NAMD for multicore
ADD NAMD_2.13_Nightly_64-bit_August-2018_multicore.tar.gz ./opt

# Rename directory for ease-of-use
RUN mv /opt/NAMD_Git-2018-08-07_Linux-x86_64-multicore /opt/namd

# Hard-coded default includes tests:
ARG test=True

COPY examples /benchmarks

RUN if [ "$test" == "True" ] ; \
    then echo "Tests activated" ; \
    else echo "Tests not activated" && rm -r /benchmarks ; fi

ENV PATH=/opt/namd:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

CMD ["bin/bash"]
