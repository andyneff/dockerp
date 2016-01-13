# README #

Docker+ - Adds two simple features to a Dockerfile

1. SOURCE - A Dockerfile command that can source another dockerfile into your dockerfile. 
FROM and MAINTAINER lines are stripped from sourced files. By default, if the sourced
file is not an absolute path, the current working directory is used. If the file
can not be found, the docker+ directory is used to search for plugins. Plugins
are common featured dockerfile to do some complicated tasks automatically.
See [Plugins](#plugins) section. They often need environment variables or docker
options to run properly, so be sure to read the plugin file/section.
2. Variable substitution - Variables following the pattern `[{VAR_NAME}]` will be
substituted using environment variables of the same name. If the environment
variable does not exit, `[{VAR_NAME}]` will be removed. A default value can be
declared using `[{VAR_NAME:DEFAULT_VALUE}]` notations. These are environment variables
that have to be exported when `docker build` is executed.

## Requirements ##

Currently bash, perl and sed are used. It should be compatible with all docker
capable operating systems including: Linux, OSX, and Windows running MINGW32 
(and MINGW64, even though that is not supported yet by boot2docker)

## Usage ##

    cat Dockerfile.config | docker+.bsh > Dockerfile

or

    docker+.bsh Dockerfile.config > Dockerfile

## Justification ##

Doesn't this break the entire dockerhub pattern and repeatability? Well, YES!

But in the end, variables substitution is necessary for more complicated situations.
For example, a docker that matches the UID of the running user. Instead of hard
coding it in a docker files, a simple Makefile or script can be written to call
docker+ and create a Dockerfile at build time.

    build:
      env USER_UID=`id -u` USER_GID=`id -g` docker build -t my_image_name .

Code diversion is another issue I have with the current tag system for dockerhub.
I may have many Dockerfiles where ONLY one line would change... instead of switching
tags and updating them all, I just one ONE main dockerfile, and use docker+ to
create the multiple versions of the Dockerfile
[Example](https://github.com/andyneff/git-lfs_dockers/blob/52ef18235d54b60a9d4ad233a69a5c12b0107232/commit_tags.bsh)

## Plugins ##

### nvidia.plugin ###

Purpose: Installs the nvidia driver in the docker (with no kernel module)

Justification: This is needed to do any GPU work in the docker, but the version
of the driver in the docker must match EXACTLY the version of the host. Thanks


Variables: The docker environment variable `NVIDIA_VERSION` **must** be set before
`SOURCE nvidia.plugin`

Issues: Certain versions of the nvidia driver are not available for download. They
were distributed via cuda repo only. This includes all nvidia drivers install via
a repo before 352.55. To install these, look at one of the other nvidia-*.plugins

Does not work in boot2docker (unless you find someway of adding a GPU device to
the host os, which I don't even know if it is possible for TCL. Plus you would
have to have a quadro or electronically modified GTX card to get GPU passthrough
enabled... So IF you used a custom OS and quadro or modified tx card, I suppose 
it MIGHT be possible to pass a GPU device from the virtual machine to the 
docker... I have and never will test this. Good luck!)

### cuda.plugin ###

Purpose: Installs the cuda SDK

Justification: This is needed to do OpenCL and cuda processing in the docker

Variables: The docker environment variable `CUDA_VERSION` **must** be set before
`SOURCE cuda.plugin`

Issues: Every version of the Nvidia driver can support UP to a specific version
of the CUDA SDK. Attempting a version after that will result in a "CUDA Driver version"
mismatch with the "CUDA Runtime Version". 

The version of CUDA the video card is the "CUDA Driver Version", while the version
of CUDA being compiled and run against is the "CUDA Runtime Version". The version 
of CUDA the video card driver can support can be checked with the deviceQuery 
from the CUDA SDK Sampled (yeah, chicken before the egg problem). Usually, an 
older version of CUDA can be used, but there are exceptions. For example, 346.46 
does not work well with cuda 5.5.

Currently only versions 7.5 to 5.5 are configured to work.

### cuda-sample.plugin ###

Purpose: Same as `cuda.plugin`, except the samples are installed.


### cuda-env.plugin ###

Purpose: Setup paths for cuda

Justification: In order to use cuda easily, the ld path and path need to be 
modified for cuda

Variables: Uses `CUDA_VERSION` like `cuda.plugin`

### user.plugin ###

Purpose: Setup user in docker

Justification: A user account (often matching the running user's UID/GID) is
necessary for some docker tasks. For example, to access the X server, you must
pass /tmp/.X11-unix to the docker AND have the same UID/GID, thanks to 
[link](http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker/). This
is also important when modifying files and you want the uid/gid to match

Variables: The docker environment variables USER_NAME, USER_UID, and USER_GID
**may** be defined before `SOURCE user.plugin`. If they are not set, the defaults
`dev 1000:50` are used, which matches boot2docker's uid/gid.

Issues: Only supports one user and one gid. For more complicated situations, 
don't use this plugin. 

### user_pre.plugin and user_post.plugin ###

Purpose: Same as `user.plugin`, but split into two different steps

Justification: The caching feature in docker is extremely useful, especially for
images that take a considerable amount of time to `build`. Since there is a
possibility the uid/gid may change a lot in your image, the 
`SOURCE user_pre.plugin` can be used at the beginning of the Dockerfile to create
the dev user, and additional Dockerfile commands can use the dev user account. At 
the end, the environment variables can be set followed by `SOURCE user_post.plugin`
to change the user to your desired UID and GID (and username if you want). The
home directory is recursively `chown`ed, but nothing else.

Issues: Any files outside of `~dev` owned by dev will need to be `chown`ed
