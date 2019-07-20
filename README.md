# Enterprise-DCOS-1.12-Install-Cheatsheet

This repository comprises my "cheatsheet" of steps to go from a "base Linux (Centos 7 or RHEL 7) to fully running DC/OS Cluster.

It is broken into three steps:
[1. Prerequisites:](https://github.com/jdyver/Enterprise-DC-OS-LATEST-Install-Cheatsheet/blob/master/1%20-%20Prerequisites.md): To be installed on all nodes including Boodstrap, Master, and Agents
2.  Bootstrap Preparation:  Prepares the installation and configuration files on the Bootstrap node.
3.  Install:  Deploys individual software to the nodes based on the cluster node type

FYI - I have used these instructions with Enterprise DC/OS 1.11 as well.  Please make note that Centos & RHEL 7.5 are not officially supported as operating systems for Enterprise DC/OS 1.11

these instructions are my personal condensation of the Advanced Installer from the Enterprise DC/OS 1.12 documents abnd a few other of my personal best practices.  this repo is not affiliated with Mesosphere, Inc and is not supported by Mesosphere, Inc. in any way.

