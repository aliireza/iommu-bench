# Experiments

This folder contains the NPF scripts to benchmark IOMMU/IOTLB.

Our NPF testie file/script contains all the required options to redo most of the paper experiments. However, we only provide a few Makefile rules to run a subset of experiments done in our paper. One can easily define new rules to perform other experiments.  

**Note that you need to setup your testbed and modify some variables in the npf testie file before running any experiment. The most important variables in the npf testie file are related to (i) the PCIe BDF (bus:device.function) address to gather IOTLB metrics via pcm, (ii) number of cores/queues & NUMA nodes, and (iii) iPerf configurations. Please go through them and modify them according to your testbed.** 

The details of the experiments are as follows:

* **iPerf Performance vs. Rate**: This experiment compares the performance of iPerf & IOTLB with/without IOMMU at different rates. We also report the CPU utilization of a CPU socket. To perform this experiment, you need to modify the IOMMU status in the GRUB configuration file, reboot the system, and then execute the appropriate Makefile rule.
  - `make test_iperf_rate_iommu_off`: You should disable IOMMU and then run this test to gather the data for IOMMU=OFF.
  - `make test_iperf_rate_iommu_on`: You should enable IOMMU and then run this test to gather the data for IOMMU=ON.
  - `make test_iperf_rate`: Run this after executing the two previous rules to generate a graph with both IOMMU=OFF and IOMMU=ON results. 

<!-- <p align="center">
<img src="images/test_iperf_rate.png"  alt="iPerf-Rate" width="50%">
</p> -->

* **iPerf Performance vs. MTU**: This experiment compares the performance of iPerf & IOTLB with/without IOMMU for different MTU sizes. Execute the appropriate Makefile rule, as follows:
  - `make test_iperf_mtu_iommu_off`: You should disable IOMMU and then run this test to gather the data for IOMMU=OFF.
  - `make test_iperf_mtu_iommu_on`: You should enable IOMMU and then run this test to gather the data for IOMMU=ON.
  - `make test_iperf_mtu`: Run this after executing the two previous rules to generate a graph with both IOMMU=OFF and IOMMU=ON results. 


* **Other experiments**: `iommu.npf` file contains many different tags, which you can enable/disable to create your custom benchmark via Makefile. The following list briefly describe the most important ones:
  - `freqtune`: Repeats the experiments for different CPU frequencies (FREQ variable). 
  - `uncoretune`: Repeats the experiments for different uncore CPU frequencies (UNCORE_FREQ variable). 
  - `ratetune`: Repeats the experiments for different rates frequencies (RATE variable). It uses `-b` option in iPerf.
  - `ddiotune`: Repeats the experiments for different number of DDIO ways (ddio_value variable). 
  - `pcm`: It uses PCM tool to gather IOTLB-related metrics. It reports a few metric for the duration of the experiment.  
  - `pcmtime`: It uses PCM tool to gather IOTLB-related metrics. It saves the values for every second. 
  - `perf`: It uses perf tool to gather IOTLB-related metrics. Note that the current perf events are specified for AMD processors, but you can extend it to cover Intel processors. 
  - `pps`: It captures the number of received/transmitted (RX/PX) packets per second on the DUT.
  - `ppstime`: It captures the number of received/transmitted (RX/PX) packets per second on the DUT. It saves the values for every second. 
  - `cpu`: It uses mpstat to measure the CPU utilization. 
  - `drop`: It uses ethtool to capture the number of dropped packets at the reception (i.e., `rx_packets_phy`, `rx_out_of_buffer`, and `rx_discards_phy` events reported by the Mellanox driver).
  - `queue`: It uses ethtool to capture the number of received packets per queue and report the load imbalance. 
  - `iperf`: It uses iperf.
  - `reverseiperf`: It uses iperf in a reverse setup where DUT generates packets. 
  - `iperftime`: It uses iperf. It reports the per-second statistics. 