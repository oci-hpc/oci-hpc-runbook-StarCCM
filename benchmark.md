
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
