# Experiments

This folder contains the NPF scripts to benchmark IOMMU/IOTLB.

Our NPF testie file/script contains all the required options to redo most of the paper experiments. However, we only provide a few Makefile rules to run a subset of experiments done in our paper. One can easily define new rules to perform other experiments.  

**Note that you need to setup your testbed before running any experiment.**

The details of the experiments are as follows:

* **iPerf Performance vs. Rate**: This experiment compares the performance of iPerf & IOTLB with/without IOMMU. We also report the CPU utilization of a CPU socket. To perform this experiment, you need to modify the IOMMU status in the GRUB configuration file, reboot the system, and then execute the appropriate Makefile rule.
  - `make test_iperf_rate_iommu_off`: You should disable IOMMU and then run this test to gather the data for IOMMU=OFF.
  - `make test_iperf_rate_iommu_on`: You should enable IOMMU and then run this test to gather the data for IOMMU=ON.
  - `make test_iperf_rate`: Run this after executing the two previous rules to generate a graph with both IOMMU=OFF and IOMMU=ON results. 