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
- [Running the Application](#running-the-application)
- [Benchmark Example](#benchmark-example)
  - [17 Millions Cells](#17-millions-cells)
  - [105 Millions Cells](#105-millions-cells)
 
# Architecture
The architecture for this runbook is as follow, we have one main machine (The headnode) that will start the jobs. Other machines (Compute Nodes) will be accessible from the headnode and STAR-CCM+ will distribute the jobs to the compute nodes. The headnode will be accesible through SSH from anyone with the key (or VNC if you decide to enable it) Compute nodes will only be accessible from inside the network. This is made possible with 1 Virtual Cloud Network with 2 subnets, one public and one private.   

![](https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/HPC_arch_draft.png "GPU Architecture for Running HFSS in OCI")

# Deployment

Deploying this architecture on OCI can be done in different ways.
* The [resource Manager](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/ResourceManager.md#deployment-through-resource-manager) let you deploy it from the console. Only relevant variables are shown but others can be changed in the zip file. 
* [Terraform](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/terraform.md#deployment-through-terraform-script) is a scripting language for deploying resources. It is the foundation of the Resource Manager, using it will be easier if you need to make modifications to the terraform stack often. 
* The [web console](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Documentation/ManualDeployment.md#deployment-via-web-console) let you create each piece of the architecture one by one from a webbrowser. This can be used to avoid any terraform scripting or using existing templates. 

# Running the Application
Running Star-CCM+ is pretty straightforward: 
You can either start the GUI if you have a VNC session started with 
```
/mnt/share/install/version/STAR-CCM+version/star/bin/starccm+
```
To run on multiple nodes, place the model.sim in `/mnt/share/work/` and replace the number of cores used in total as the np argument. 

```
/mnt/share/install/14.04.011/STAR-CCM+14.04.011/star/bin/starccm+ -batch -power -licpath 1999@flex.cd-adapco.com -podkey ++AaAaaaAAaAAAAAAAAAaa -np 106 -machinefile /mnt/share/install/machinelist.txt /mnt/share/work/model.sim
```
## MPI implementations and RDMA

Performances can really differ depending on the MPI that you are using. 3 are supported by Star-CCM+ out of the box. 

* IBM Platform MPI: Default or flag `platform`
* Open MPI: Flag `intel`
* Intel MPI: Flag `openmpi3`

To specify options, you can use the flag `-mppflags`

When using OCI RDMA on a Cluster Network, you will need to specify these options: 

### Platform: 
For RDMA:
```
-intra=shm -e MPI_HASIC_UDAPL=ofa-v2-cma-roe-enp94s0f0 -UDAPL
```
For better performances:
```
-prot -aff:automatic:bandwidth
```
To pin on the first 36 threads:
```

```

### Intel: 
For RDMA: 
```
-mppflags "-iface enp94s0f0 -genv I_MPI_FABRICS=shm:dapl -genv DAT_OVERRIDE=/etc/dat.conf -genv I_MPI_DAT_LIBRARY=/usr/lib64/libdat2.so -genv I_MPI_DAPL_PROVIDER=ofa-v2-cma-roe-enp94s0f0 -genv I_MPI_FALLBACK=0"
```

Additionaly, instead of disabling hyper-threading, you can also force the MPI to pin it on the first 36 cores:  
```
-genv I_MPI_PIN_PROCESSOR_LIST=0-33 -genv I_MPI_PROCESSOR_EXCLUDE_LIST=36-71
```

### OpenMPI:



# Benchmark Example
<p align="center">
<img src="https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Images/lemans.png" height="300">
</p>
Performances of STAR-CCM+ are often measured using the LeMans benchmark with 17 and 105 Millions cells. The next graphs are showing how using more nodes impact the runtime, with a scaling really close to 100%. RDMA network, which has not been discussed in this runbook, only start to differentiate versus regular TCP runs if the Cells / Core ratio starts to go down.  

## 17 Millions Cells

<p align="center">
<img src="https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Images/RunTime_17M.png" height="350">
<img src="https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Images/scaling_17M.png" height="350">
</p>

## 105 Millions Cells

<p align="center">
<img src="https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Images/RunTime_105M.png" height="350">
<img src="https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/blob/master/Images/Scaling_105M.png" height="350">
</p>
