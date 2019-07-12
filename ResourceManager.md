# <img src="https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/raw/master/Images/STARCCM_logo.png" height="60"> Siemens Simcenter STAR-CCM+ Runbook
 
**Table of Contents**
- [Deployment through Resource Manager](#deployment-through-resource-manager)
  - [Log In](#log-in)
  - [Resource Manager](#resource-manager)
  - [Select variables](#select-variables)
  - [Run the stack](#run-the-stack)
 
# Deployment through Resource Manager

## Log In
You can start by logging in the Oracle Cloud console. If this is the first time, instructions to do so are available [here](https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/signingin.htm).
Select the region in which you wish to create your instance. Click on the current region in the top right dropdown list to select another one. 

<img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/Region.png" height="50">

## Resource Manager
In the OCI console, there is a Resource Manager available that will create all the resources needed. 

Select the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/menu.png" height="20"> on the top left, then select Resource Manager and Stacks. 

Create a new stack: <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/stack.png" height="20">

Download the [ZIP file](https://github.com/oci-hpc/oci-hpc-runbook-StarCCM/raw/master/Terraform/starccm.zip) for STAR-CCM+

Add you private key to the zip file

Upload the ZIP file

Choose the Name and Compartment

## Select variables

Click on <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/next.png" height="20"> and fill in the variables. 

* AD: Availability Domain of the cluster (1,2 or 3)
* SSH_PRIVATE_KEY_PATH: Private key path. (Name of the file that you added to the zip)
* SSH_PUBLIC_KEY: Public key to access the cluster
* COMPUTENODE_COUNT: Number of compute machines (Integer)
* COMPUTE_SHAPE: Shape of the Compute Node (BM.HPC2.36)
* HEADNODE_SHAPE: Shape of the Head Node which is also a Compute Node in our architecture (BM.HPC2.36)
* GPUNODE_COUNT: Number of GPU machines for Pre/Post
* GPUPASSWORD: password to use the VNC session on the Pre/Post Node
* GPU_AD: Availability Domain of the GPU Machine (1,2 or 3)
* GPU_SHAPE: Shape of the Compute Node (VM.GPU2.1,BM.GPU2.2,...)

Click on <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/next.png" height="20">

Review the informations and click on <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/create.png" height="20">

## Run the stack

Now that your stack is created, you can run jobs. 

Select the stack that you created.
In the "Terraform Actions" dropdown menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/tf_actions.png" height="20">, run terraform apply to launch the cluster and terraform destroy to delete it. 

Once you have done this step, everything else in this runbook will be done except for [the download and instalation of the software](#installing-star-ccm).
