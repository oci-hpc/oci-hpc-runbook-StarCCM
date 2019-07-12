- [Installing STAR-CCM+](#running-the-application)
- [Running the Application](#running-the-application)
- [Benchmark Example](#benchmark-example)
  - [17 Millions Cells](#17-millions-cells)
  - [105 Millions Cells](#105-millions-cells)
# Installing STAR-CCM+
There are a couple of library that need to be added to the Oracle Linux image on the headnode and the compute nodes.

```
sudo yum -y install libSM libX11 libXext libXt
```

You can download the STAR-CCM+ installer from the Siemens PLM website or push it to your machine using scp. 
`scp /path/own/machine/ELECTRONICS_version.zip "opc@1.1.1.1:/home/opc/"`

Without a VNC connection, a silent installation needs to be done. 

```
mkdir /mnt/share/install
/path/own/machine/installscript.sh -i silent -DINSTALLDIR=/mnt/share/install/
```

If you would like to include the installation in the Resource Manager or terraform script. Unzip the files and edit the file hn-start-starccm.sh

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
