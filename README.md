# <img src="https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/raw/master/Images/STARCCM_logo.png" height="60"> Siemens Simcenter STAR-CCM+ Runbook

# Introduction
This Runbook will take you through the process of deploying one or multiple machines on Oracle Cloud Infrastructure, installing Simcenter STAR-CCM+, configuring the license, and then running a model.

Simcenter STAR-CCM+ is a complete multiphysics solution for the simulation of products and designs.

Running Simcenter STAR-CCM+ on Oracle Cloud Infrastructure is quite straightforward, follow along this guide for all the tips and tricks. 
<p align="center">
<img src="https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/raw/master/Images/Screenshot 2019-07-09 at 14.50.19.png" height="300" >
 </p>
 
**Table of Contents**
- [Introduction](#introduction)
- [Architecture](#architecture)
- [Deployment through Resource Manager](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/ResourceManager.md#deployment-through-resource-manager)
- [Deployment through Terraform Script](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/terraform.md#deployment-through-terraform-script)
- [Deployment via web console](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/ManualDeployment.md#deployment-via-web-console)
- [Installing STAR-CCM+](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/STAR-CCM%2B.md#installing-star-ccm)
- [Running the Application](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/STAR-CCM%2B.md#running-the-application)
- [Benchmark Example](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/STAR-CCM%2B.md#benchmark-example)
 
# Architecture
The architecture for this runbook is as follow, we have one main machine (The headnode) that will start the jobs. Other machines (Compute Nodes) will be accessible from the headnode and STAR-CCM+ will distribute the jobs to the compute nodes. The headnode will be accesible through SSH from anyone with the key (or VNC if you decide to enable it) Compute nodes will only be accessible from inside the network. This is made possible with 1 Virtual Cloud Network with 2 subnets, one public and one private.   

![](https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/HPC_arch_draft.png "GPU Architecture for Running HFSS in OCI")

# Deployment

Deploying this architecture on OCI can be done in different ways.
* The [resource Manager](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/ResourceManager.md#deployment-through-resource-manager) let you deploy it from the console. Only relevant variables are shown but others can be changed in the zip file. 
* [Terraform](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/terraform.md#terraform-installation) is a scripting language for deploying resources. It is the foundation of the Resource Manager, using it will be easier if you need to make modifications to the terraform stack often. 
* The [web console](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/ManualDeployment.md#deployment-via-web-console) let you create each piece of the architecture one by one from a webbrowser. This can be used to avoid any terraform scripting or using existing templates. 
