##Fork notes
This is a fork from [storm-docker](https://github.com/viki-org/storm-docker) - at present the only change is that it runs Storm 0.9.5 instead of 0.9.2-incubating.

storm-docker
============

This repository makes it easy to run **distributed, multiple server**
(multiple Zookeeper, multiple Storm Supervisor)
[Storm](http://storm.incubator.apache.org/) topologies within
[Docker](https://www.docker.io/).

Read our [storm-docker blog post on the Viki Engineering blog](http://engineering.viki.com/blog/2014/announcing-storm-docker-making-distributed-multiple-server-storm-setups-easy-in-docker/)

More documentation on Github pages: http://dev.viki.com/storm-docker

## What's Special?

If we do a search of the word `storm` on the
[Docker Registry](https://registry.hub.docker.com/), we see several pages of
results. What makes our storm-docker special compared to similar offerings?

1. Supports multiple server Storm setups, in particular Zookeeper and
Storm Supervisor.
2. High level of configurability through **documented** sample configuration
files
3. Same configuration files used for **all** machines in the Storm cluster
4. Instructions on this `README` showing you how to setup the repository
5. Important parts of codebase is extensively documented to aid hacking.
No need to guess what the original author is thinking.
6. [Github pages](http://dev.viki.com/storm-docker) with more documentation;
a slight redesign and walkthrough is in the works
7. Tested and used in production.

Point 1 is probably the single biggest reason why you should use our
storm-docker, even if you're running everything on a single machine initially
(which storm-docker **does** support, btw).
When you need to add new machines to your Storm cluster, you'll find the
transition smooth and leverage on the work we did in figuring out how to run
multiple Zookeeper and Storm Supervisor in Docker.

At this point in time (23 June 2014), our storm-docker repository is probably
the first and only open source Docker setup that supports distributed,
multi-server Storm clusters out of the box.

## Important Note on machines in your Storm cluster

The storm-docker project is tested using Amazon EC2 instances; we make use of
the `curl` command to obtain the public and private IP addresses of the
machines. This may be a problem for machines which are not Amazon EC2 instances
(even though we have not faced any similar issues at Viki), as seen by this
Github issue:

https://github.com/viki-org/storm-docker/issues/1

As such, please use one of the following 2 setups for the machines in your
Storm cluster if you're using the storm-docker project:

1. None of the machines in your Storm cluster are Amazon EC2 instances
2. ALL the machines in your Storm cluster are Amazon EC2 instances in the same
security group

Refer to the documentation for the `all_machines_are_ec2_instances` key in the
`config/storm-setup.yaml.sample` file for more information.

## System Requirements

The following software is required for running the `storm-docker` repository.
In other words, for machines which are going to form your Storm cluster:

- GNU Make
- Docker
- python 2.7.x
- virtualenv

## Software Setup

**NOTE:** The steps here must be carried out for **all** machines in your
Storm cluster.

### Python setup

We will be making use of [virtualenv](http://virtualenv.readthedocs.org/en/latest/)
for some of the Python scripts in this repository. We also make use of the
[PyYAML](http://pyyaml.org/) library, and that requires some Python header
files.

On a Ubuntu-like system:

    sudo apt-get install python-virtualenv
    sudo apt-get install python-dev

### Install Docker

Use the following command from the top of the script at http://get.docker.io
to install Docker

    wget -qO- https://get.docker.io/ | bash

Verify your Docker installation:

    docker info

### Cloning this repository

Clone this repository, preferrably to `$HOME/workspace/storm-docker`.
At `$HOME/workspace`:

    git clone https://github.com/viki-org/storm-docker.git

The next few commands will be run from the `storm-docker` repository. Let us
go there:

    cd storm-docker

## Configuration

**NOTE:** The steps here must be carried out for **all** machines of your
Storm cluster unless otherwise stated.

### Configuring the Storm setup

**NOTE:** This step is **critical** to the correct functioning of the Storm
topology.

**NOTE:** storm-docker assumes that all machines in the Storm cluster make use
of the same configuration files.
As such, you can perform this step of editing the configuration files once
(on any machine) and copy the files to all the machines of your Storm cluster.

Copy the sample configuration files to concrete configuration files (the
`copy-sample-config.sh` script **does not** overwrite any existing concrete
configuration files):

    ./copy-sample-config.sh

Carry on by editing the following concrete configuration files:

- `config/storm-setup.yaml`
- `config/cluster.xml`
- `config/zoo.cfg`

Documentation is available in the copied concrete configuration files, except
the `config/cluster.xml` file used for logback configuration.
A default set of configuration has been set; for more fine-grained
configuration, we highly recommend reading
[The logback manual](http://logback.qos.ch/manual/).

Once done with your edits, we can continue with building the Docker images.

### Building the Docker images

**NOTE:** This step is necessary after making changes to **any** of the
configuration files in the above subsection.

Run the **GNU** `make` command. The default goal builds the Docker images:

    make

If this is the first time the Docker images are being built, this script will
take some time to complete.

## Running the Storm components

### Run the Docker containers

Before actually running the various Docker containers, you might want to verify
that all servers used in various sections of the `config/storm-setup.yaml` file
are listed under the `servers` dictionary (located in the same file) by running
the `scripts/verify_storm_setup_yaml.py` script:

    . venv/bin/activate
    python scripts/verify_storm_setup_yaml.py

Warnings will be printed to stderr should some servers be missing from the
`servers` dictionary.

As at 26 December 2014, it is highly recommended to run the `scripts/remote.py`
file to automatically spin up the various Docker containers on your Storm
cluster (instead of manually running the `start-storm.sh` script on each
server), like this:

    . venv/bin/activate
    python scripts/remote.py --all

[Fabric](http://fabfile.org) is used to run the `start-storm.sh` script on the
various servers and spin up the correct Docker containers, depending on your
configuration in the `config/storm-setup.yaml` file.

## Stopping Docker containers

To stop all running Docker containers:

    ./destroy-storm.sh

To stop individual containers, supply them as arguments to the
`destroy-storm.sh` script, for instance to stop the `ui` and `zookeeper`
containers:

    ./destroy-storm.sh ui zookeeper

## Motivation

This project was started to address the need to increase the scalability and
fault tolerance of Viki's Storm cluster. A lot of effort was spent figuring out
how to run multiple Storm Supervisor and Zookeeper in Docker.

This repository should be viewed more as a foundation on which you can build
on for running your Storm cluster in Docker, rather than as a defacto standard
for running Storm in Docker.

To better aid someone new to the codebase to understanding and subsequently
modifying the code, much of the core Python code contains rather extensive
inline documentation.

## Credits

This repository was originally based on
[wurstmeister/storm-docker](https://github.com/wurstmeister/storm-docker);
big thanks to wurstmeister for making his project open source.
