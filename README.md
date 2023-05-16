# iommu-bench

[![DOI](https://zenodo.org/badge/525288873.svg)](https://zenodo.org/badge/latestdoi/525288873)

This repository contains information to benchmark IOMMU by (re)doing some of the experiments performed in the [paper][iommu-paper].


## Experiments

The experiments are located at `experiments/`. The folder has a `Makefile` and `README.md` that can be used to run the experiments.

Note: Before running the experiments, you need to prepare your **testbed** according to the **[our guidelines](TESTBED.md)**.

## Custom Kernels

Our paper uses different Linux kernels to perform experiments, see `https://github.com/aliireza/linux`. The following list summarizes the relevant patches:

* [Aritifial TCP drops][luigi-patch]: This branch is based on a kernel patch developed by Luigi Rizzo (see [here][luigi-patch-info]). It artifically drops TCP packets to induce TCP re-transmissions and put more pressure on IOMMU/IOTLB. You need to set three parameters as follows:
  - **lossy_local_port**, **lossy_remote_port**: If non zero, they indicate the sockets matching one of these ports that will ignore drops.
  - **drop_freq**:  If non zero, the packets will be artificially dropped on the receive side one every `drop_freq`.
  
    ```bash
    # Example
    echo 2345 > /sys/module/tcp_input/parameters/lossy_local_port
    echo 10 > /sys/module/tcp_input/parameters/drop_freq # drop one in 10
    ```

* [Page Pool API with hugepage bulk allocation][page-pool-patch]: This branch contains changes to perform bulk allocations backed by hugepages within the Page Pool API in order to rely on hugepage IOTLB mappings. 
  - There is another [branch][page-pool-ethtool-patch] that provides an `ethtool` option to enable/disable the hugepage support, but it is not tested. 
    ```bash
    sudo ethtool -K <interface_name> rx-hp-alloc off
    ```





## Citing our paper

If you use PacketMill or X-Change in any context, please cite our [paper][iommu-paper]:

```bibtex
@article{farshin-iommu,
 title = {Overcoming the IOTLB wall for multi-100-Gbps Linux-based networking},
 author = {Farshin, Alireza and Rizzo, Luigi and Elmeleegy, Khaled and Kostić, Dejan},
 year = 2023,
 month = may,
 keywords = {200 Gbps, Hugepages, iPerf, IOMMU, IOTLB, Linux kernel, Packet processing},
 abstract = {
This article explores opportunities to mitigate the performance impact of IOMMU on high-speed network traffic, as used in the Linux kernel. We first characterize IOTLB behavior and its effects on recent Intel Xeon Scalable & AMD EPYC processors at 200 Gbps, by analyzing the impact of different factors contributing to IOTLB misses and causing throughput drop (up to 20% compared to the no-IOMMU case in our experiments). Secondly, we discuss and analyze possible mitigations, including proposals and evaluation of a practical hugepage-aware memory allocator for the network device drivers to employ hugepage IOTLB entries in the Linux kernel. Our evaluation shows that using hugepage-backed buffers can completely recover the throughput drop introduced by IOMMU. Moreover, we formulate a set of guidelines that enable network developers to tune their systems to avoid the “IOTLB wall”, \textit{i.e}., the point where excessive IOTLB misses cause throughput drop. Our takeaways signify the importance of having a call to arms to rethink Linux-based I/O management at higher data rates.
},
 volume = 9,
 pages = {e1385},
 journal = {PeerJ Computer Science},
 issn = {2376-5992},
 url = {https://doi.org/10.7717/peerj-cs.1385},
 doi = {10.7717/peerj-cs.1385}
}
```


## Getting Help

If you have any questions regarding our code or the paper, you can contact [Alireza Farshin][alireza-page] (farshin at kth.se).


[iommu-paper]: a
[alireza-page]: https://www.kth.se/profile/farshin/
[luigi-patch]: https://github.com/aliireza/linux/tree/luigi-patch-v5.15
[luigi-patch-info]: https://lists.bufferbloat.net/pipermail/bloat/2021-October/016693.html
[page-pool-patch]: https://github.com/aliireza/linux/tree/page-pool-bulk-5.15
[page-pool-ethtool-patch]: https://github.com/aliireza/linux/tree/page-pool-bulk-ethtool
