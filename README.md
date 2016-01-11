# README #

Docker+ - Adds a simple commands to a Dockerfile and another simple feature

1. SOURCE - Ability to source another docker file in your docker files. FROM and
MAINTAINER are stripped from sourced files.
2. Variable substitution - Variables following the pattern `[{VAR_NAME}]` will be
substituted using environment variables of the same name. If the environment
variable does not exit, `[{VAR_NAME}]` will be removed. A default value can be
declared using `[{VAR_NAME:DEFAULT_VALUE}]` notations

## Requirements ##

Currently bash, perl and sed are used. Future versions will by pure python.

## Usage ##

    cat Dockerfile.config | docker+.bsh > Dockerfile

or

    docker+.bsh Dockerfile.config > Dockerfile

## Justification ##

Doesn't this break the entire dockerhub pattern and repeatability? Well, YES!

But end of variables substitution is necessary for more complicated situations.
For example, a docker that matches the UID of the running user. Instead of hard
coding it in a docker files, a simple Makefile or script can be written to call
docker+ and create a Dockerfile at build time.

Code diversion is another issue I have with the current tag system for dockerhub.
Instead of maintaining multiple tags, I maintain the master branch, and use
docker+ to auto generate the differnt tags. 
[Example](https://github.com/andyneff/git-lfs_dockers/blob/52ef18235d54b60a9d4ad233a69a5c12b0107232/commit_tags.bsh)
