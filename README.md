# Ansible VXLAN-EVPN for Campus

[![N|Solid](https://upload.wikimedia.org/wikipedia/commons/3/31/Juniper_Networks_logo.svg)](https://www.juniper.net/documentation/solutions/en_US/campus)

[![Build Status](https://travis-ci.org/packetferret/Ansible-VXLAN-EVPN-for-Campus.svg?branch=master)](https://travis-ci.org/packetferret/Ansible-VXLAN-EVPN-for-Campus)

## Overview

`Ansible-VXLAN-EVPN-for-Campus` is an easier way to deploy a VXLAN / EVPN fabric with Juniper's campus solutions. 

This project is based on *[Infrastructure-As-Code](https://dev.to/fedekau/infrastructure-as-code-a-beginners-perspective-2l8k)* concepts, where all elements of a device's configuration as stored in a format of `key:value` pairs and stored in a source-code management system such as Github or Gitlab. While not required for successful deployments, we at Juniper encourage network devops teams to *[leverage best practicies with git](https://dev.to/bholmesdev/git-github-best-practices-for-teams-opinionated-28h7)*.

The network designs available for you here are based upon *[Juniper's Validated Solutions](https://www.juniper.net/documentation/solutions/en_US/campus)*, feel free to take moment to review the *[Collapsed Core with EVPN Multihoming](https://www.juniper.net/documentation/en_US/release-independent/nce/information-products/pathway-pages/nce/nce-182-evpn-collapsed-core-evpn-multihoming-campus-pwp.html)* and *[EVPN-VXLAN Fabric](https://www.juniper.net/documentation/en_US/release-independent/nce/information-products/pathway-pages/nce/nce-172-evpn-vxlan-campus.html)* architectures.

## How to use this project

The crux of this project is executing the *[pb.configuration.network.yml](https://github.com/packetferret/Ansible-VXLAN-EVPN-for-Campus/blob/master/files/ansible/pb.configuration.network.yml)* playbook found in `/files/ansible/` directory. Taking a peek at this relatively simple playbook, we can see that there are just five tasks executed:

>1. validate that Ansible is running at least version 2.7.8
>2. remove and rebuild local directories that hold our generated configurations
>3. build files to host the various stanzas of a Juniper configuration 
>4. assemble all stanza files into a full device configuration
>5. push the configuration to the networking device

## New Features

- `Makefile` included to shortcut many of the commands
- Docker container provided for those that want execute in an isolated environment.
- EVE-NG lab for importing my toplogy has been added in `files/eve-ng_lab` 

## Topology

Here is a high level perspective of what the playbook will build without any adjustments made to the yaml files stored within the respective `group_vars` and `host_vars` directories

### Campus Collapsed Core Diagram

[![campus collapsed core](https://github.com/packetferret/Ansible-VXLAN-EVPN-for-Campus/blob/master/files/images/campus_collapsed_core_01.png)](https://github.com/packetferret/Ansible-VXLAN-EVPN-for-Campus/blob/master/files/images/campus_collapsed_core_01.png)


## Execution

The execution of this playbook can be acheived in multiple ways, experiment with each to find out your organization's preference. As you will see throughout the documentation, we have provided a `Makefile` to act as a shortcut for many of our project's commands.

### Running inside a Docker container

This is a personal preference, but this is a very clean method to run the playbook as it already manages all of your Python package dependencies. Within the root directory, you will run two commands to build and run the dedicated docker container.

```sh
make build
make ansible
```

For my visual friends, refer to this gif as an example:

[![make build](https://github.com/packetferret/Ansible-VXLAN-EVPN-for-Campus/blob/master/files/images/make_build.gif)](https://github.com/packetferret/Ansible-VXLAN-EVPN-for-Campus/blob/master/files/images/make_build.gif)


> *note: the make build command only needs to be ran once; subsequent executions of the command will not hurt anything, but will add a second or two of wasted time*


[![make ansible](https://github.com/packetferret/Ansible-VXLAN-EVPN-for-Campus/blob/master/files/images/make_ansible.gif)](https://github.com/packetferret/Ansible-VXLAN-EVPN-for-Campus/blob/master/files/images/make_ansible.gif)

### Running the Ansible playbook from your local workstation

If Docker isn't your thing, or your workplace just simply prevents you from having any fun and you need to run everything outside of containers, simply run the following command in the project's root directory:

```sh
make run
```

[![make run](https://github.com/packetferret/Ansible-VXLAN-EVPN-for-Campus/blob/master/files/images/make_run.gif)](https://github.com/packetferret/Ansible-VXLAN-EVPN-for-Campus/blob/master/files/images/make_run.gif)

## A quick word on the roles used within this playbook

This project is highly modular and will allow you adjust as needed to accomplish any design goal. For instance, if you wanted to use OSPF instead of BGP as the underlay, there is nothing that would stop you from updating the `protocols.yml` files to support that functionality. Because of this modularity, I felt it best to put all of the functionality of compiling these configurations within seperate roles, and then we can declare which types of devices should run through with roles (i.e. Firewalls should get the `security` stanza, but not data center switches)

> for the uninitiated, roles are found in the `roles/` directory of a project, and in our project, that directory will be `files/ansible/roles/`

Here is a quick breakdown of the formatting of roles used within this playbook:

### `juniper_stanza_` roles

As you will see during the playbook's execution, each device will have every configuration stanza generated as a seperate, temporary file. The creation of these files is the result of our roles that start with the name `juniper_stanza_`

Note that each one of these roles has a task in the `tasks/main.yml` file, and a templating file as `template/stanza_name.j2`. The former references the latter to produce a stanza of configuration, to be stored in the set of temporary directories `/tmp/ansible/{{ inventory_hostname }}/tmp`

### `juniper_assemble_config` role

This will be the role that compiles each individual stanza into a single, cohesive configuration. Unfortunately, we have to prepend an integer to the left side of the file name to force Ansible's module to load them in the correct sequence.

A little ugly, but happy with the result.

### `juniper_configuration_` roles

This small subset of roles allows for the checking, diff'ing, and application of configurations to the devices live on the network. Currently these are disabled, allowing the playbook to merely render the configuration.

If you'd like to apply the configurations to the devices at run time, then make sure all of the IP addresses and hostnames are correct in your `inventory.yml` file, or dynamic inventory source of truth (i.e. Netbox), and uncomment out these three roles in the main playbook `pb.configuration.network.yml` file.

### `juniper_jsnapy_` roles

These are roles that will help you perform network validation on the status of your VXLAN/EVPN enabled campus. Purely focused on validation operational state, but infinetly helpful in your day-to-day operations.

### `localhost-build-dirs` role

This role manages the local directories on your workstation. It will remove the previous `config/` directory and make sure an empty one replaces it before storing the completed/assembled configurations within it.

[![config_files](https://github.com/packetferret/Ansible-VXLAN-EVPN-for-Campus/blob/master/files/images/config_files.gif)](https://github.com/packetferret/Ansible-VXLAN-EVPN-for-Campus/blob/master/files/images/config_files.gif)

> Look in `config/` directory after your playbook is executed, this is where your assembled configurations are stored.

## Dependencies

The dependencies required come back to your preferred deployment method.

### Ansible on your local workstation

You will need to install the Python packages to your local workstation. We highly encourage you to *[leverage python virtual environments](https://realpython.com/python-virtual-environments-a-primer/)* to avoid situations where you may overwrite your system's local python packages.

Install the required python packages:

```sh
pip install -r files/docker/ansible/files/requirements.txt
```

### Ansible from within a Docker container

If using Docker as your execution engine, you may not be surprised to learn that you'll need Docker installed on your local workstation. That's it!

*[Docker](https://docs.docker.com/get-docker/)*


### Optional:

Protect your `secrets.yml` file by *[using Ansible-Vault to encrypt it with a password](https://docs.ansible.com/ansible/latest/user_guide/vault.html)*.

`ansible-vault encrypt group_vars/all/secrets.yml`

## Development

Want to contribute? Great!

Submit a PR and let's work on this together :D
