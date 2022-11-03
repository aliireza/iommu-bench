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
@inproceedings{farshin-iommu,
author = {Farshin, Alireza and Rizzo, Luigi and Elmeleegy, Khaled and Kostić, Dejan},
 title = {{Overcoming the IOTLB Wall for Multi-100-Gbps Linux-based Networking}},
 booktitle = {},
 series = {},
 year = {2023},
 isbn = {},
 location = {},
 pages = {},
 articleno = {},
 numpages = {},
 url = {},
 doi = {},
 acmid = {},
 publisher = {},
 address = {},
 keywords = {},
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
