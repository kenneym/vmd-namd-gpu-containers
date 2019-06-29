# Cuda VMD and NAMD Container Images:

Included in this repository are several images for commonly used molecular dynamics pair VMD and NAMD. A VMD image based on ubuntu 16.04 can be found at the following [github page](https://github.com/kenneym/vmd-cudagl). All images contained in this repo, however, run on centos7 bases and sit in the directory titled "compose." Within this directory sits a cuda-enabled, fully fledged, visual/ Opengl rendering VMD build. This image will work on any machine, with or without cuda, and will take advantage of Nvidia GPUs if they are available. Likewise, in this directory, both a cuda-enabled, and a traditional CPU-based NAMD sits in this directory. As of the time of this writing, NAMD cannot run Free-Energy Perturbation (FEP) calcuations on cuda versions of NAMD software, so having both a CPU and GPU-based container for NAMD will be beneficial for those engaging in FEP (use the GPU container to run prep. simulations and CPU container for the FEP itself for quickest results). All three of these containers can be deployed anywhere, with a simple docker-compose command.

Each image was built by extending NVIDIA's CudaGl images (Cuda + Opengl). All cudagl images have dockerfiles hosted on [Gitlab](https://gitlab.com/nvidia/cudagl) and can be pulled from [Dockerhub](https://hub.docker.com/r/nvidia/cudagl/) 

These images include Cuda Version 9.1,  VMD version 1.9.4a12. You may either clone this repository and modify these images to your liking, or pull directly from Dockerhub. 

## Purpose

The purpose of these images  are to enable researchers to use either script-driven OR visual VMD depending upon their needs. While scripting in VMD is a very powerful tool, some researchers may find that certain tasks are best done visually. Further, they may find that some work-flows in VMD vary extensively enough experiment-to-experiment that scripting is not ideal. In our case, we need researchers who are not familiar with tcl/tk scripting to use some features of this container visually, enabling them to put forth their knowledge within their own realms of expertise without spending extensive time learning scripting. Containerized programs are attractive for a variety or reasons, including their ease-of-use, compatibility with powerful servers, and more. 

We should note that NVIDIA offers images of NAMD and VMD on their GPU cloud platform; however, there are two reasons we felt these offerings were inadequate. For one, there is no visual version of Visual Molecular Dynamics(VMD). Secondly, we wanted to be able to modify our containers to our liking, add additional scientific software, and understand the core strcuture of these dockerfiles. Because NVIDIA's offering did not offer copies of the Dockerfile, and we couldnt gain any insights into how the containers were strcutured, we preffered to build our own containers to be modified to our liking. We ran testbenches against the containers found on offer from NVIDIA, and the performance of the containers here matched or exceeded the NVIDIA containers in all cases.

**In short, these images are designed to make containerized NAMD and VMD as versatile as possible.**


## Directory Information

Inside the  `test-dockerfiles` directory:

1. `benchmarks` - described later on. Performance benchmarks run using these containers

2. `compose`- The directory containing all files central to the use of our cuda or CPU VMD/ NAMD containers.

## Running These Containers:

#### Requirements:

1. nvidia-docker2 installed
2. NVIDIA GPU on-board
3. NVIDIA drivers version 390 or greater
4. Have read and agreed to the University of Illinois Visual Molecular Dynamics Software License Agreement 

#### Obtaining the Binaries:

Redistribution of VMD & NAMD binaries is not permitted and therefore I am not able to actually include the neccessary binaries in this github repo. Unfortunately, you will have to head over the Theoretical and Computational Biophysics group page and download [VMD](https://www.ks.uiuc.edu/Development/Download/download.cgi?PackageName=VMD) and [NAMD](https://www.ks.uiuc.edu/Development/Download/download.cgi?PackageName=NAMD). Doing this can be a bit tricky, but following these steps will get you there.

**VMD:**
1. Download the latest VMD version 1.9.4 ALPHA build for linux (make sure it has cuda).
2. Add this binary to the `./test-dockerfiles/compose/visual-vmd-centos7/` directory.
3. Head over to the [Dockerfile](./test-dockerfiles/compose/visual-vmd-centos7/Dockerfile) and scroll down to lines 39-40 where we add and create a directory for the VMD program:

```Dockerfile
ADD vmd-1.9.4a12.bin.LINUXAMD64-CUDA9-OptiX411-OSPRay140.opengl.tar.gz /opt 
RUN ln -s  /opt/vmd-1.9.4a12 /opt/vmd
```
Edit the Dockerfile and change the name of the `.tar.gz` file after `ADD` to match the name of the latest version of this binary. Then, edit the directory name to match the current name of the binary (ex: `opt/vmd-1.9.4a20`, if we have reached alpha version a20).

5. We are now ready to go on the VMD side of things, but the same process needs to be repeated for both the CPU version of NAMD and the GPU version. Repeat the exact some process for each of these images as well. Drop the nightly build of the "multicore" NAMD image into the  `./test-dockerfiles/compose/namd-centos7` directory and the "cuda" NAMD image into the `./test-dockerfiles/compose/namd-centos7` directory, and repeat the editing process we just completed above in the corresponding Dockerfiles found in each directory. Note that the base image of the Cuda-Enabled NAMD image is `cudagl:9.1-devel-centos`. If the cuda version being used in the nighty build of the NAMD image changes to a higher version of cuda, you need to make sure to edit the `FROM` statement at the top of the Dockerfile to match the version of cuda required by the NAMD image. Check out the cudagl base-image offerings on [Gitlab](https://gitlab.com/nvidia/cudagl) and can be pulled from [Dockerhub](https://hub.docker.com/r/nvidia/cudagl/) if neccessary.

#### Modifying a Container:

If your computer or server's cuda version is higher or lower than 9.1, consider going to [Nvidia's CudaGL repository on Dockerhub](https://hub.docker.com/r/nvidia/cudagl/). This repo will have plenty more images that may fit your needs. 

To modify the cuda version used in the VMD container, simply change the "FROM" declaration at the top of the Dockerfile to whatever version of CudaGL you prefer. Ensure, however, that you are using the *development version* of whatever CudaGl image you choose. Any modifications to the "FROM" declaration should work just fine, as long as you retain the same OS (i.e. Ubuntu or Centos) and remember to use the development version CudaGL.


### Running the Containers - Step 1: Enable X-forwarding

Type the following commands directing into your bash command line. This code will allow docker containers that you select to access the X server on your host. 
```
XSOCK=/tmp/.X11-unix
XAUTH=/tmp/.docker.xauth
touch $XAUTH
xauth nlist $DISPLAY | sed -e 's/^..../ffff/' | xauth -f $XAUTH nmerge -
xhost +local:docker
```

Alternatively, simply `cd` to the `./test-dockerfile/compose` directory, and issue the following commands:

`chmod ug=wrx permissions.sh`

`./permissions.sh`

Using this automated process may be helpful, since these commands will need to be issued every single time the computer shuts down and restarts.

### Step 2: Install a Cuda Library:

The cuda library is the library of programs, executables, etc. that allows us to run code on NVIDIA GPUs. In order to benefit from the speedups associated with GPU computing, we need to install a library. The set of containers provided here port only to cuda version 9.1, however, if a later version of cuda is desired (9.2 when it finally releases on fedora, or cuda 10.0 when it realeases along with the new NVIDIA turing architechture in the coming months), then one can easily edit the docker files, such that the base images change from `nvidia/cudagl:9.1-devel-centos7` to `nvidia/cudagl:9.2-devel-centos7`, or another version of cudagl. Note that as long as the system's cuda library is equal to or higher than the cuda version in the dockerfiles, you will be set to go. Additionally, note that the cuda version in the Dockerfile must exactly match the cuda version of the corresponding NAMD or VMD binaries.


The installation of cuda differs from operating system to operating system. Consult the NVIDIA cuda download pages for instructions.

Note that the LD_LIBRARY_PATH must be properly set to the directory of your cuda installation, if GPU acceleration is to work properly.

Within these containers, LD_LIBRARY_PATH is correctly set. On a local machine, however, it is essential that LD_LIBRARY_PATH is set to the directory of your cuda Libraries (ex: LD_LIBRARY_PATH= /usr/local/cuda, on Centos, but definetely not the case in Fedora... you'll have to figure this one out). In Arch Linux, simply set LD_LIBRARY_PATH to `opt/cuda`.

### Step 3: Install NVIDIA-Docker

**NOTE!**  NVIDIA-Docker has been recently updated. Consult the NVIDIA-Docker page on github for additional information. Nvidia-Docker is now able to be used with Docker-CE, and now requires users to use:`docker run --runtime=nvidia` instead of `nvidia-docker run`. Please note these changes, as I have not gotten the time to alter this document to reflect them

Consult the additional documentation within this directory for information on installing nvidia docker.


## Running the Containers

Next up, we've actually got to run the containers themselves. We have two options here

### Option 1: Use Docker Compose

This option will perform a fully automated build and deployment of a cuda  VMD container, a cuda NAMD container, and a typical CPU-based NAMD container. This will allow for a wide range of work to be performed in all of these molecular dynamics programs.

1.  [Install](https://docs.docker.com/compose/install)�Docker Compose... *Note:* [double check](https://github.com/docker/compose/releases "Github Docker Compose Releases") for up-to-date version.

2.  From the toplevel directory of the `molecular dynamics` repo:

   `cd ./test-dockerfiles/compose`

3. Ensure that X-forwarding is enabled, the docker daemon is running, and you're opering on a device with cuda enabled (i.e. an NVIDIA GPU is on-board, and the cuda libraries are ready to go and compatable with the current dockerfiles)

4. Take a look at the directory structure now. The directory named `workspace` (under `./test-dockerfiles/compose` should include any .pdb files, genetic sequences, etc. that you would like to manipulate/ use with the set of VMD/NAMD containers. This directory will be bind-mounted when the containers are started. Note that, even after the containers are up and running, you may add new files to this directory and access them from within the workspace directory of any of the containers that will be deployed

5. Inside the `examples` directory are two performance test-sets. These test sets are used to test the performance of the containers once they are up and running. To take a look at how these containers performed against the offical NVIDIA ones, and against an NAMD program installed locally (as supposed to inside a container), `cd ../benchmarks` and look at the "Wallclock" time. All of the 4 tests (CPU-NAMD, Host-installed GPU NAMD, my GPU NAMD dockerfiles, and NVidia's official one) were run on the same APO1 test set.

6. The `docker-compose.yml` file contains all of the configurations for the docker compose builds. Note the `runtime:nvidia` declaration, which enables us to run these containers on GPU through docker-compose. Also note under the `args` sections of each container, there is an option named `test`, which is set to `false`. If you would like to include the aformentioned performance tests for usage inside your container, merely set these `test` variables to `true`

7.  Now, it's time to deploy. Run the command `docker-compose up` and all the containers will begin to build. If everything is in the right place, we should be able to build without error. 

8. Once the build of all three containers is complete, these containers will never need to be built again on the same machine, so long as they are not removed. Docker compose will recognize this upon the next calling of the command.

9. As soon as the building phase ends, VMD should put up it's own graphical window, and should give you acess to all files in the workspace and test directories (if you decided to add in the performance tests)

10. You won't see that the CPU and GPU NAMD containers are running, but indeed they are. To get into them, enter a new terminal window or tab, and issue the command `docker exec compose_CUDA-NAMD_1 /bin/bash `. No need for volumes, flags, anything. These were already created by docker compose, so they're already ready to go. To get into the CPU version of NAMD, simply use the same command with the name that compose gives the CPU-based container

11. To stop these containers from running any further, simply type `docker-compose down`. Now that you've built all these containers once, they'll be incredibly easy to launch again, since the images are now cached.

---

### Option 2: Build Invididual Images

In the case that you want to build only one image, for instance, you may only want to run the GPU version of NAMD, or the CPU version on a high powered server. Here's how you do it:

#### Step 2: Build the Container

Clone this repository. Then, cd to the `test-dockerfiles/compose` directory. Enter into the directory that contains your image of choice, and issue the following command:

    docker build -t your-container-name .

#### Step 3: Run the Container

 - Now `cd` back to the `compose/workspace` directory, or any other directory containing files relevant to your work with VMD or NAMD. To run a visual VMD container, issue the command below. This will enable your container to access files in your local directory (such as .pdb files) within the container's workspace directory, and enable to the container to access the host's X-server. Note that, in the case that you are running a GPU-less machine, of course, nvidia-docker would be replaced with plain old "docker."

	nvidia-docker run -it --rm -v $(pwd):/workspace \
	-v /tmp/.X11-unix:/tmp/.X11-unix 
	-v /tmp/.docker.xauth:/tmp/.docker.xauth \
	-e XAUTHORITY=/tmp/.docker.xauth -e DISPLAY \
	container-name

- Running the image in this way will cause VMD to launch directly. If you would instead prefer to enter the container's command line first, you can issue the same command with `bin/bash` appended to the end:

      nvidia-docker run -it --rm -v $(pwd):/workspace \
      -v /tmp/.X11-unix:/tmp/.X11-unix -v /tmp/.docker.xauth:/tmp/.docker.xauth \
      -e XAUTHORITY=/tmp/.docker.xauth -e DISPLAY \
      container-name /bin/bash

- To run NAMD containers, simply run

`nvidia-docker run -it --rm -v $(pwd):/workspace container-name /bin/bash`

- Note that, in these containers LD_LIBRARY_PATH is correctly set; however, to run NAMD outside of a container on your local machine, you must append the directory location of your NAMD install to the beginning of LD_LIBRARY_PATH. For instance, in arch linux, we would have `LD_LIBRARY_PATH='path/to/NAMD/install:opt/cuda'`

- To run NAMD to GPU nodes, note the flags in the following command:

  `./namd2 +p$(nproc) +setcpuaffinity +idlepoll <configfile.namd> > logfile.log`

  - +p - number of processors (CPU) on which to run

  - setcpuffinity - not totally sure what the purpose of this commmand is, but it is suggested by the creators of NAMD for both CPU and GPU runs
  
  - idlepoll - Ensures that NAMD is consistantly checking to see if it can run more code on the GPU

  - configfile.namd - the name of the NAMD configuration file that specifies what NAMD is actually computing

  - logfile - the file that NAMD will spit information to

- To check whether or not NAMD is running on GPU, run `nvidia-smi` while an intensive NAMD job is running, or launch the `nvidia x-sever` app. You should see the temp of the GPU go up slightly over time with smi, or should see the powerlevel rise in the x-server nvidia graphical app... the job has to be pretty extensive though to get significant temperature rise.

- VMD will tell you immediately upon launching if it has detected a GPU acceleration. It will spit this into stdout.
