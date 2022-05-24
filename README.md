# [CLIJ](https://clij.github.io/) Benchmarking Workflow in [Bwb](https://github.com/BioDepot/BioDepot-workflow-builder)

This repository contains a workflow for the Biodepot-workflow-builder
(Bwb) implementing a benchmarking workflow for CLIJ, a library and
extension for the Fiji image processing suite allowing the use of GPUs
for accelerated processing (using OpenCL).

The workflow implemented is the one described in the Supplementary Materials
section of the original CLIJ paper (see [References](#References) below), with
some modifications to adapt it to the Bwb platform. Like all Bwb workflows, the
benchmarking workflow is containerized, which means it can be deployed on a more
powerful cloud server with minimal effort, and does not require installation of
anything besides Docker (and video drivers).

## Installation
Bwb requires that Docker be installed ([instructions
here](https://github.com/BioDepot/BioDepot-workflow-builder#installing-and-starting-docker)). Additionally,
you will need to have
the [NVIDIA Container Toolkit
installed](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) 
to make the GPU available to Docker containers. Note that AMD GPUs are not
currently supported, as Bwb uses the `--gpus` option in Docker, which does not
support AMD GPUs at time of writing (this is an open issue, see https://github.com/docker/cli/issues/2063).

### Installing GPU Drivers
The following instructions should work for an Ubuntu 20.04 system.

```bash
# Update software repositories and perform any necessary upgrades:
sudo apt update && sudo apt upgrade
# Install NVIDIA server driver 510:
sudo apt install nvidia-driver-510-server
```
You may need to **reboot your system** to finalize installation of the drivers.
Afterwards, check if your GPU is recognized by running
```bash
nvidia-smi
```

### Cloning repository
Clone this repository with
```bash
git clone https://github.com/PUBLIC_URL_HERE
```
*TODO*: Change this URL.

Navigate into the repository with
```bash
cd clij-benchmarking
```

### Building Docker Images
*TODO*: Remove this section if images are uploaded to DockerHub.

The widgets in this workflow use several custom Docker images that are not
available on the central Docker Hub repository, and need to be built on the host
machine; the most notable is the `fijiOCL` widget which contains GPU drivers and
OpenCL libraries for running Fiji with CLIJ. To build the Docker images, simply
navigate to the *root directory of the repository* and run
```bash
sudo ./build_docker_images.sh
```
Note that you will need root access on the host machine.

**NOTE:** It is important that this script is run from the root directory, as it
uses this assumption to navigate into directories containing relevant
`Dockerfile`s. The script will pause for 5 seconds to allow the user to confirm
that they are indeed in the root directory; to bypass this, run
```bash
sudo NOSLEEP=1 ./build_docker_images.sh
```

## Usage
### Start the Bwb server
Run the command

```bash
sudo docker run --rm \
 -p 5900:5900 -p 6080:6080 \
 -v ${PWD}/:/data \
 -v /var/run/docker.sock:/var/run/docker.sock \
 -v /tmp/.X11-unix:/tmp/.X11-unix \
 --privileged --group-add root \
 biodepot/bwb:imaging__latest
```

### Connect to Bwb with a browser or VNC client

To use Bwb, the user can use a browser (Chrome/Firefox/Safari) or a VNC client
(e.g. RealVNC). Instructions are given in [the Bwb documentation](https://github.com/biodepot/biodepot-workflow-builder#overview-running-bwb). In
most cases, the browser should be set to http://localhost:6080 if the Bwb server is
started on a laptop or the https://\<ip of the remote machine>:6080 if started on a
remote or cloud server. In addition, to connect to Bwb on the cloud, a port must
be opened and forwarded to allow browser and client to communicate with Bwb. The
exact methodology will depend on the cloud provider.  Some instructions for
Amazon web services are provided
[here](https://github.com/BioDepot/BioDepot-workflow-builder#how-do-i-run-bwb-on-the-cloud)

### Loading the workflow
1. From the Bwb menu bar, select `File > Load Workflow`
2. Navigate to `/data`, and select `clij_benchmarking_workflow`, then press
   "Choose".
3. Double click on the "Start" widget and press the blue "Start" button to start
   the workflow. 
   
The workflow will download images and benchmarking macros, and then perform the
same sequence of image processing operations on the dataset using both the
GPU and the CPU, and then compare the runtime and results of the two analyses.

Please note that the CPU benchmark may take a very long time to complete
depending on how many images are used - the runtime scales linearly with the
number of images used, since images are sequentially processed.
   
### Modifying parameters
By default, the workflow uses 300 images from the [dataset used in the CLIJ
paper](https://doi.org/10.17617/1.8J) for benchmarking; this can be adjusted by
double clicking on the "Download Images" widget and modifying the parameters.

The entire dataset consists of 607 images, numbered `000000` through `000606`;
the "Pattern" field of the "Download Images" widget contains a [`printf`-style format
string](https://en.wikipedia.org/wiki/Printf_format_string); by default it is
the URL of one of these images, with a `%06d` placeholder that will be replaced
by an image number, zero-padded to be 6 digits long. The number will range from
the value in the "Minimum Value" field to the value in the "Maximum Value"
field, inclusively, and files will be placed in the directory specified in the
"Output directory" field. Additionally, the "Replace Existing Files" checkbox
can be used to control whether existing files should be replaced; since the
dataset is very large, the user may choose to only download missing files and
skip existing ones.

Note that for simplicity, the benchmarking workflows will use **all** the images
in the chosen output directory; if you have already downloaded images once, and
wish to re-run the workflow with a smaller subset of images, either delete the
images you do not wish to use, or move them to another directory.

## References

* CLIJ
	> Robert Haase, Loic Alain Royer, Peter Steinbach, Deborah Schmidt,
	>     Alexandr Dibrov, Uwe Schmidt, Martin Weigert, Nicola Maghelli,
	>     Pavel Tomancak, Florian Jug, Eugene W Myers. CLIJ: GPU-accelerated
	>     image processing for everyone. Nat Methods 17, 5-6 (2020)
	>     doi:10.1038/s41592-019-0650-1
 
	 Workflow is taken from the Supplementary Materials section of this paper.
 
* Dataset used is the one given in the Supplementary Materials section of the
  CLIJ paper:
  https://doi.org/10.17617/1.8J

* Fiji:
  
    > Schindelin, J., Arganda-Carreras, I., Frise, E., Kaynig, V.,
	>   Longair, M., Pietzsch, T., … Cardona, A. (2012). Fiji: an
	>   open-source platform for biological-image analysis. Nature Methods,
	>   9(7), 676–682. doi:10.1038/nmeth.2019
