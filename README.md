# <img src="https://github.com/oci-hpc/oci-hpc-runbook-HFSS/blob/master/Images/203-40.png" height="40"> Siemens Simcenter STAR-CCM+ Runbook
## Introduction
This Runbook will take you through the process of deploying one or multiple machines on Oracle Cloud Infrastructure, installing Simcenter STAR-CCM+, configuring the license, and then running a model.

Simcenter STAR-CCM+ is a complete multiphysics solution for the simulation of products and designs.

Running Simcenter STAR-CCM+ on Oracle Cloud Infrastructure is quite straightforward, follow along this guide for all the tips and tricks. 

![]( "Example Simcenter STAR-CCM+ simulation") 
 
## Architecture
The architecture for this runbook is as follow, we have one main machine (The headnode) that will start the jobs. Other machines (Compute Nodes) will be accessible from the headnode and STAR-CCM+ will distribute the jobs to the compute nodes. The headnode will be accesible through SSH from anyone with the key (or VNC if you decide to enable it) Compute nodes will only be accessible from inside the network. This is made possible with 1 Virtual Cloud Network with 2 subnets, one public and one private.   

![](https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/HPC_arch_draft.png "GPU Architecture for Running HFSS in OCI")

## Deployment
There are multiple options available to get started with STAR-CCM+ on OCI. The next 2 sections will show how to do it from the console in a webbrowser and using a Terraform script. Scripts are especially usefull with more complex architecture. For STARCCM+, 2 architectures are interesting. 
** Single HPC node
** Cluster with multiple compute nodes
### Console
#### Log In
You can start by logging in the Oracle Cloud console. If this is the first time, instructions to do so are available [here](https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/signingin.htm).
Select the region in which you wish to create your instance. Click on the current region in the top right dropdown list to select another one. 

<img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/Region.png" height="50">

#### Virtual Cloud Network
Before creating an instance, we need to configure a Virtual Cloud Network. Select the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/menu.png" height="20"> on the top left, then select Networking and Virtual Cloud Networks. <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/create_vcn.png" height="20">

On the next page, select the following: 
* Name of your VCN
* Compartment of your VCN
* Choose "CREATE VIRTUAL CLOUD NETWORK PLUS RELATED RESOURCES"

Scroll all the way down and <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/create_vcn.png" height="20">

Close the next window. 

#### Subnets
If you are using one compute node, the subnet was created during the VNC creation. If you want to create a cluster, we will generate a private subnet for the compute nodes, accessible only from the headnode.

Before we generate a private subnet, we will define a security rule to be able to access it from the headnode. We would also like to download packages on our compute nodes, we will create a NAT gateway to be able to access online repositories to update the machine. 

##### NAT Gateway
You have just created a VCN, click on the name.
In the ressource menu on the left side of the page, select NAT Gateways.

<img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/raw/master/images/resources_menu.png" height="200">

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/raw/master/images/NAT.png" height="20">

Choose a name (Ex:STARCCM_NAT) and click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/raw/master/images/NAT.png" height="20">


##### Security List
In the ressource menu on the left side of the page, select Security Lists.

Click on <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/raw/master/images/create_sl.png" height="20">

Select a name like STARCCM_Private_SecList

Add an Ingress Rule with CIDR 10.0.0.0/16 and select All Protocols

Add an Egress Rule with CIDR 0.0.0.0/0 and select All Protocols

Click on <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/raw/master/images/create_sl.png" height="20">

To allow the creation of a Network File System, we also need to add sa couple of ingress rules for the Default Security List for STARCCM_VCN. Click on "Default Security List for STARCCM_VCN" in the list. 

Add one ingress rules for all ports on TCP for NFS. 

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/addIngress.png" height="20">  

* CIDR : 10.0.0.0/16
* IP PROTOCOL: TCP
* Source Port Range: All
* Destination Port Range: All
Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/addIngress.png" height="20"> 

Add another ingress rule for UDP for NFS:

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/addIngress.png" height="20">  

* CIDR : 10.0.0.0/16
* IP PROTOCOL: UDP
* Source Port Range: All
* Destination Port Range:111,2049
Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/addIngress.png" height="20"> 


##### Route Table
In the ressource menu on the left side of the page, select Route Tables.

Click on <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/raw/master/images/create_rt.png" height="20">

Change the name to STARCCM_Private_RT

Click + Additional Route Rule and select the settings:

* TARGET TYPE : NAT Gateway
* DESTINATION CIDR BLOCK : 0.0.0.0/0
* TARGET NAT GATEWAY : STARCCM_NAT

Click on <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/raw/master/images/create_rt.png" height="20">

##### Subnet
In the ressource menu on the left side of the page, select Subnets.
Click on <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/raw/master/images/create_subnet.png" height="20">

Choose the following settings:

* NAME : STARCCM_Private_Subnet
* TYPE: "REGIONAL"
* CIDR BLOCK: 10.0.3.0/24
* ROUTE TABLE: STARCCM_Private_RT
* SUBNET ACCESS: "PRIVATE SUBNET"
* SECURITY LIST: STARCCM_Private_SecList

Click on <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/raw/master/images/create_subnet.png" height="20">

#### Compute Instance
Create a new instance by selecting the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/menu.png" height="20"> on the top left, then select Compute and Instances. 

<img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/Instances.png" height="300">

On the next page, select <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/create_instance.png" height="25">

On the next page, select the following:
* Name of your instance
* Availibility Domain: Each region has multiple availability domain. Some instance shapes are only available in certain AD.
* Change the image source to CentOS 7.
* Instance Type: Select Bare metal
* Instance Shape: 
  * BM.HPC2.36
  * Other shapes are available as well, [click for more information](https://cloud.oracle.com/compute/bare-metal/features).
* SSH key: Attach your public key file. [For more information](https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/creatingkeys.htm).
* Virtual Cloud Network: Select the network that you have previsouly created. In the case of a cluster: Select the public subnet for the Headnode and the Private Subnet for the compute nodes.

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/create_instance.png" height="20">

After a few minutes, the instances will turn green meaning it is up and running. You can now SSH into it. After clicking on the name of the instance, you will find the public IP. You can now connect using `ssh opc@xx.xx.xx.xx` from the machine using the key that was provided during the creation. 

For a compute node to be able to access the NAT Gateway, select the compute node and in the Resources menu on the left, click on Attached VNICs. 

Hover over the three dots at the end of the line and select "Edit VNIC"

Uncheck "Skip Source/Destination Check"

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/updateVNIC.png" height="20">

Restart This section for each compute instance. Before you do that, we will create a ssh key specific for the cluster to allow all machines to talk to each other using ssh. Log on to the headnode you created and run the command `ssh-keygen`. Do not change the file location (/home/opc/.ssh/id_rsa) and hit enter when asked about a passphrase (twice). Or run this command:

```cat /dev/zero | ssh-keygen -q -N ""```

Add the content of id_rsa.pub into the file /home/opc/.ssh/authorized_keys. You will also use the content of id_rsa.pub as the public key when creating your compute nodes. 

```cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys```

#### Mounting a drive

HPC machines have local NVMe storage but it is not mounted by default. Let's take care of that! 

After logging in using ssh, run the command `lsblk`. 
The drive should be listed with the NAME on the left (Probably nvme0n1, if it is different, change it in the next commands)

The headnode will have the shared drive with the installation and the model. This will be shared between all the different compute nodes. Each compute node will also mount the drive to be running locally on a NVMe drive.  

If your headnode is also a compute node, you can partition the drive. 

Make sure gdisk is installed : ` sudo yum -y install gdisk `
Let's use it: 
```
sudo gdisk /dev/nvme0n1
> n      # Create new partition
> 1      # Partition Number
>        # Default start of the partition
> +500G  # Size of the shared partition
> 8300   # Type = Linux File System
> n      # Create new partition
> 2      # Partition Number
>        # Default start of the partition
>        # Default end of the partition, to fill the whole drive
> 8300   # Type = Linux File System
> w      # Write to file
> Y      # Accept Changes
```

Format the drive on the compute node:
```
sudo mkfs -t ext4 /dev/nvme0n1
```

Format the partitions on the headnode node:
```
sudo mkfs -t ext4 /dev/nvme0n1p1
sudo mkfs -t ext4 /dev/nvme0n1p2
```

Create a directory and mount the drive to it. For the headnode, select `/mnt/share` as the mount directory for the 500G partition and `/mnt/local` for the larger one. For compute node, select `/mnt/local` as the mount directory of the whole drive.

Compute Node:
```
sudo mkdir /mnt/local
sudo mount /dev/nvme0n1 /mnt/local
sudo chmod 777 /mnt/local
```

Head Node:
```
sudo mkdir /mnt/share
sudo mkdir /mnt/local
sudo mount /dev/nvme0n1p1 /mnt/share
sudo mount /dev/nvme0n1p2 /mnt/local
sudo chmod 777 /mnt/share
sudo chmod 777 /mnt/local
```


#### Creating a Network File System

##### Headnode

Since the headnode is in a public subnet, we will keep the firewall up and add an exception through. 
```
sudo firewall-cmd --permanent --zone=public --add-service=nfs
sudo firewall-cmd --reload
```
We will also activate the nfs-server:

```
sudo yum -y install nfs-utils
sudo systemctl enable nfs-server.service
sudo systemctl start nfs-server.service
```

Edit the file /etc/exports with vim or your favorite text editor. `sudo vi /etc/exports` and add the line `/mnt/share   10.0.0.0/16(rw)`

To activate those changes:

```sudo exportfs -a```

##### Compute Nodes
On the compute nodes, since they are in a private subnet with security list restricting access, we can disable it altogether. We will also install the nfs-utils tools and mount the drive. You will need to grab the private IP address of the headnode. You can find it in the instance details in the webbrowser where you created the instances, or find it by running the command `ifconfig` on the headnode. It will probably be something like 10.0.0.2, 10.0.1.2 or 10.0.2.2 depending on the CIDR block of the public subnet. 

```
sudo systemctl stop firewalld
sudo yum -y install nfs-utils
sudo mkdir /mnt/share
sudo mount 10.0.0.2:/mnt/share /mnt/share
```


#### Allow communication between machines
After creating the headnode, you generated a key for the cluster using `ssh.keygen`. We will need to send the file `~/.ssh/id_rsa` on all compute nodes. On the headnode, run ```scp /home/opc/.ssh/id_rsa 10.0.3.2:/home/opc/.ssh``` and run it for each compute node by changing the IP address. 


### Terraform Script
#### Terraform Installation

Download the binaries on the [terraform website](https://www.terraform.io/) and unzip the package. Depending on your Linux distribution, it should be similar to this:

```
tf_install_dir=~/tf_install_dir
cd $tf_install_dir
wget https://releases.hashicorp.com/terraform/0.12.0/terraform_0.12.0_linux_amd64.zip
unzip terraform_0.12.0_linux_amd64.zip
echo export PATH="\$PATH:$tf_install_dir" >> ~/.bashrc
source ~/.bashrc
```

To check that the installation was done correctly: `terraform -version` should return the version. 

#### Using terraform
##### Configure
Download the tar file and untar the content (Not Available yet):
* [HPC Shape](https://github.com/oci-hpc/oci-hpc-runbook-HFSS/raw/master/terraform_templates/clusterHPC.tar)

Edit the file terraform.tfvars for your settings, info can be found [on the terraform website](https://www.terraform.io/docs/providers/oci/index.html#authentication)

* Tenancy_ocid
* User_ocid
* Compartment_ocid
* Private_key_path
* Fingerprint
* SSH_private_key_path
* SSH_public_key
* Region

**Note1: For Compartment_ocid: To find your compartment ocid, go to the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/menu.png" height="20"> and select Identity, then Compartments. Find the compartment and copy the ocid.**

<img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/compartment_OCID.png" height="150">

**Note2: The private_key_path and fingerprint are not related to the ssh key to access the instance. You can create using those [instructions](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/apisigningkey.htm). The SSH public and private keys can be generated like [this](https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/creatingkeys.htm)**


In the variable.tf file, you can change the availability domain, and the number of compute nodes. 

##### Run
```
cd <folder>
terraform init
terraform plan
terraform apply
```
##### Destroy
```
cd <folder>
terraform destroy
```

## Installation
This guide will show the different steps for the CentOS 7 image available on Oracle Cloud Infrastructure. 
### Installing AEDT
There are a couple of library that need to be added to the CentOS image. 

```sudo yum -y install mesa-libGLU-devel mesa-libGL-devel libXp mesa-dri-drivers libXmu libXft giflib libXt libpng12.x86_64 mesa-libGL.i686 glibc.i686 bzip2-libs.i686 libpng.i686 libtiff.i686 libXft.i686```

You can download the AEDT installer from the Ansys website or push it to your machine using scp. 
`scp /path/own/machine/ELECTRONICS_version.zip "opc@1.1.1.1:/home/opc/"`

Without a VNC connection, a silent installation needs to be done. In our case, the license is pointing to a license server at IP 192.168.0.1. If you have different settings, you can record another option file with -options-record "/path/to/dir/filename"

```
mkdir /mnt/share/install
echo -W Agree.selection=1 > /home/opc/silent_cmd_aedt.txt
echo -P installLocation="/mnt/share/install/AnsysEM" >> /home/opc/silent_cmd_aedt.txt
echo -W TempDirectory.tempPath="/mnt/local/tmp" >> /home/opc/silent_cmd_aedt.txt
echo -W TempDirectory.ChangeTempPermission="1" >> /home/opc/silent_cmd_aedt.txt
echo -W LibraryOption.libraryOption=0 >> /home/opc/silent_cmd_aedt.txt
echo -W LibraryOption.libraryPath="" >> /home/opc/silent_cmd_aedt.txt
echo -W LicenseOption.licenseOption=2 >> /home/opc/silent_cmd_aedt.txt
echo -W LicenseOption.licenseFileName="" >> /home/opc/silent_cmd_aedt.txt
echo -W LicenseOption.serverCount=1 >> /home/opc/silent_cmd_aedt.txt
echo -W LicenseOption.serverName1="192.168.0.1" >> /home/opc/silent_cmd_aedt.txt
echo -W LicenseOption.serverName2="" >> /home/opc/silent_cmd_aedt.txt
echo -W LicenseOption.serverName3="" >> /home/opc/silent_cmd_aedt.txt
echo -W LicenseOption.tcpPort=1055 >> /home/opc/silent_cmd_aedt.txt
unzip /path/own/machine/ELECTRONICS_version.zip
/path/own/machine/Electronics_version/Linux/AnsysEM/disk1/setup.exe -options "/home/opc/silent_cmd_aedt.txt" -silent
```

### Installing Remote Solve Mamager
In case you will create multiple nodes, RSM will be used to submit the jobs to the different machines. We also need to add some ports through the firewall. 

```
echo -W Agree.selection=1 > /home/opc/silent_cmd_rsm.txt
echo -P installLocation="/mnt/share/install" >> /home/opc/silent_cmd_rsm.txt
/path/own/machine/Electronics_version/Linux/AnsysEM/disk1/setup.exe -options "/home/opc/silent_cmd_rsm.txt" -silent
sudo firewall-cmd --permanent --zone=public --add-port=11180/tcp
sudo firewall-cmd --permanent --zone=public --add-port=12180/tcp
sudo firewall-cmd --permanent --zone=public --add-port=13180/tcp
sudo firewall-cmd --permanent --zone=public --add-port=40000-59999/tcp
sudo firewall-cmd --permanent --zone=public --add-source=10.0.0.0/16
sudo firewall-cmd --reload
```

If the headnode will also be a compute node, start the RSM service. This will need to be done on each compute node as well.

```
sudo /mnt/share/install/rsm/Linux64/ansoftrsmservice startonboot -user opc
```

The Ansys EDT install need to be registered with RSM once, either on the headnode or on one compute node:

```
sudo /mnt/share/install/AnsysEM/AnsysEM19.4/Linux64/RegisterEnginesWithRSM.pl add
```

### Linking the /tmp directory on the headnode
If you have made multiple partition on your NVMe drive on the headnode, this will not be needed as you can write in /mnt/local/tmp.

Ansys AEDT will use the /mnt/local/tmp directory to run the calculation. This is currently on your boot volume which is not really efficient. We will link this directory 

```
mkdir /mnt/share/tmp
sudo mkdir /mnt/local
sudo ln -s /mnt/share/tmp /mnt/local/
```

### Connecting all compute node

Each compute node needs to be able to talk to each compute node. SSH communication works but RSM has some issue if you don't have each host in the known host file. If you used terraform, the headnode contains a file with all the ip address of the compute nodes. If not, you can compute it using the CIDR block of you private subnet

```
nmap -sn 10.0.3.0/24 | grep "scan report" | sed -e 's/.*(\(.*\)).*/\1/' > iplist.txt
```

Run those commands to download the script to register each node with all the other nodes.

```
wget https://github.com/oci-hpc/oci-hpc-runbook-HFSS/raw/master/scripts/generate_ssh_file.sh
chmod 777 generate_ssh_file.sh
./generate_ssh_file.sh
```

### (Optional) Set up a VNC
By default, the only access to the CentOS machine is through SSH in a console mode. If you want to see the Ansys EDT interface, you will need to set up a VNC connection. The following script will work for the default user opc. The password for the vnc session is set as "password" but it can be edited in the next commands. 

```
sudo yum -y groupinstall "Server with GUI"
sudo yum -y install tigervnc-server mesa-libGL
sudo systemctl set-default graphical.target
sudo cp /usr/lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:0.service
sudo sed -i 's/<USER>/opc/g' /etc/systemd/system/vncserver@:0.service
sudo mkdir /home/opc/.vnc/
sudo chown opc:opc /home/opc/.vnc
echo "password" | vncpasswd -f > /home/opc/.vnc/passwd
chown opc:opc /home/opc/.vnc/passwd
chmod 600 /home/opc/.vnc/passwd
sudo firewall-offline-cmd --zone=public --add-port=5900-5901/tcp
sudo systemctl restart firewalld
sudo systemctl daemon-reload
sudo systemctl start vncserver@:0.service
sudo systemctl enable vncserver@:0.service
```

In the console, we need to allow access to the different port to allow access.

In Networking/Virtual Cloud Networks, select your VCN and the subnet in which your machine is located and Select the security list associated with it.

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/addIngress.png" height="20"> 
* CIDR : 0.0.0.0/0
* IP PROTOCOL: TCP
* Source Port Range: All
* Destination Port Range:5900-5901

After logging in for the first time, go to application/System Tools/Settings and in Power, set the timer for Blank Screen in Power saving to Never.

<img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/CentOSSeetings.jpg" height="150"> 

## Running the Application
Running Ansys Electronic Desktop is pretty straightforward: 
You can either start the GUI if you have a VNC session started with 
```
/mnt/share/install/AnsysEM/AnsysEM19.4/Linux64/ansysedt
```

If you do not, you can run Ansys EDT in batch mode:
Copy the `/mnt/share/install/AnsysEM/AnsysEM19.4/Linux64/Examples/HFSS/RF\ Microwave/Tee.*` to `/mnt/share/work`

Edit the folowing command with your own IP's for the nodes. (In this case: 10.0.3.3 and 10.0.3.4)

``` shell
/mnt/share/install/AnsysEM/AnsysEM19.4/Linux64/ansysedt -jobid RSM_batchrun -distributed \
-machinelist list=10.0.3.3:-1:72:90%,10.0.3.4:-1:72:90% -auto -monitor -ng -batchoptions  \
"'HFSS/AllowOffCore'=1 'HFSS/DefaultProcessPriority'='Normal' 'HFSS/EnableGPU'=0 'HFSS/EnableGPUForSBR'=1\
'HFSS/HPCLicenseType'='Pack' 'HFSS/LegacyHFSS_TR'=0 'HFSS/LegacyIE'=0 'HFSS/MPIVendor'='Platform Computing'\
'HFSS/RemoteSpawnCommand'='SSH' 'HFSS/UseLegacyElectronicsHPC'=1" -batchsolve TeeModel:Nominal:Setup1 \
 /mnt/share/work/Tee.aedt
```

