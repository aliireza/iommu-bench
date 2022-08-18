# iommu-bench

This repository contains information to benchmark IOMMU by (re)doing some of the experiments performed in the [paper][iommu-paper].


## Experiments

The experiments are located at `experiments/`. The folder has a `Makefile` and `README.md` that can be used to run the experiments.

**Note: Before running the experiments, you need to prepare your testbed according to the following guidelines.**


## Testbed

Our experiments mainly requires `npf`, `iperf`, `pcm`, and `perf`. Addditionally, you may need to install a custom Linux kernel and modify GRUB configuration file.



### Network Performance Framework (NPF) Tool

You can install `npf` via the following command:

```bash
python3 -m pip install --user npf
```

**Do not forget to add `export PATH=$PATH:~/.local/bin` to `~/.bashrc` or `~/.zshrc`. Otherwise, you cannot run `npf-compare` and `npf-run` commands.** 

NPF will look for `cluster/` and `repo/` in your current working/testie directory. We have included the required `repo` for our experiments and a sample `cluster` template, available at `experiment/`. To setup your cluster, please check the [guidelines][npf-setup] for our previous paper. Additionally, you can check the [NPF README][npf-readme] file.



### Building and Installing a Custom Kernel


The easiest way to build Linux kernel is to get your current configuration using the following command:

```bash
git clone git@github.com:aliireza/linux.git
cd linux
git checkout luigi-patch-v5.15
cp -v /boot/config-$(uname -r) .config
```

**You should change `CONFIG_SYSTEM_TRUSTED_KEYS` to `""`**

You need to install the following requirements before running `make`:

```bash
sudo apt-get install libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf
```

```bash
make -j32 bindeb-pkg EXTRAVERSION=-luigi-patch
cd ..
sudo dpkg -i *.deb
sudo reboot
```

**To check the order of the installed kernel in GRUB menu, you can look for `menuentry` in `/boot/grub/grub.cfg`. Check the [GRUB Reboot](#grub-reboot) for more info**

You can also install the kernel without using `dpkg`, as follows:

```bash
sudo make modules_install
sudo make install
```

### Removing a Custom Kernel

To remove a kernel installed via `dpkg`, run the following commands:

```bash
dpkg --list | grep linux-image 
sudo apt remove linux-{image,headers}-5.4.0-buffer-shuffling
```

To remove a manually installed kernel, run the following commands:

```bash
locate -b -e 5.4.0-luigi | xargs -I{} sudo rm -fr {}
sudo update-grub
```


**Note that you need to unistall existing Mellanox OFEDs if you want to rely on the default OFED shipped with the custom kernel. To do so, you can run `sudo /usr/sbin/ofed_uninstall.sh`**

### Enable/Disable IOMMU

To enable/disable IOMMU, you can modify the iommu options at `GRUB_CMDLINE` located in `/etc/default/grub`, as follows:

```bash
# Intel
GRUB_CMDLINE_LINUX="intel_iommu=on iommu=on"

# AMD
GRUB_CMDLINE_LINUX="amd_iommu=on iommu=on"
```

### GRUB Reboot

To switch a different kernel, you can set the GRUB default boot option from the terminal. To do so, you have to first set `GRUB_DEFAULT=saved` in `/etc/default/grub` and then run `sudo update-grub`. Later, you can reboot to your desired boot option via `sudo grub-reboot 0`. The following commands can help you to find your desired boot option.

```bash
export GRUB_CONFIG=`sudo find /boot -name "grub.cfg"`
sudo grep 'menuentry ' $GRUB_CONFIG | cut -f 2 -d "'" | nl -v 0
sudo grub-reboot 0
sudo reboot
```

### PCM Tools and Perf

We use PCM tool (i.e., `pcm-iio.x`) to measure the IOTLB metrics on Intel processors. 

```bash
git clone https://github.com/opcm/pcm.git
cd pcm
make
sudo make install
```


You can also use Perf tool to measure IOTLB metrics. To install Perf tool, you can run the following command:

```bash
sudo apt-get install linux-tools-$(uname -r) linux-cloud-tools-$(uname -r)
```

However, you may need to build Perf from source to be able to monitor the relevant events. 

### Perf commands

You can use the following commands to measure IOTLB metrics via Perf.

```bash
# Intel Processors

perf stat -a --pfm-events skx_unc_iio0::UNC_IO_VTD_ACCESS:L1_MISS,skx_unc_iio0::UNC_IO_VTD_ACCESS:L2_MISS,skx_unc_iio0::UNC_IO_VTD_ACCESS:L3_MISS,skx_unc_iio0::UNC_IO_VTD_ACCESS:L4_PAGE_HIT,skx_unc_iio0::UNC_IO_VTD_ACCESS:TLB1_MISS,skx_unc_iio0::UNC_IO_VTD_ACCESS:TLB_FULL,skx_unc_iio0::UNC_IO_VTD_ACCESS:TLB_MISS,skx_unc_iio0::UNC_IO_VTD_OCCUPANCY,skx_unc_iio0::UNC_IO_VTD_ACCESS:CTXT_MISS,skx_unc_iio1::UNC_IO_VTD_ACCESS:L1_MISS,skx_unc_iio1::UNC_IO_VTD_ACCESS:L2_MISS,skx_unc_iio1::UNC_IO_VTD_ACCESS:L3_MISS,skx_unc_iio1::UNC_IO_VTD_ACCESS:L4_PAGE_HIT,skx_unc_iio1::UNC_IO_VTD_ACCESS:TLB1_MISS,skx_unc_iio1::UNC_IO_VTD_ACCESS:TLB_FULL,skx_unc_iio1::UNC_IO_VTD_ACCESS:TLB_MISS,skx_unc_iio1::UNC_IO_VTD_OCCUPANCY,skx_unc_iio1::UNC_IO_VTD_ACCESS:CTXT_MISS sleep 300

# AMD Processors
perf stat -a -e 'amd_iommu/mem_pass_untrans/,amd_iommu/mem_pass_pretrans/, amd_iommu/mem_pass_excl/, amd_iommu/mem_target_abort/, amd_iommu/mem_trans_total/, amd_iommu/mem_iommu_tlb_pte_hit/, amd_iommu/mem_iommu_tlb_pte_mis/, amd_iommu/mem_iommu_tlb_pde_hit/, amd_iommu/mem_iommu_tlb_pde_mis/, amd_iommu/mem_dte_hit/, amd_iommu/mem_dte_mis/, amd_iommu/page_tbl_read_tot/, amd_iommu/page_tbl_read_nst/, amd_iommu/page_tbl_read_gst/, amd_iommu/int_dte_hit/, amd_iommu/int_dte_mis/, amd_iommu/cmd_processed/, amd_iommu/cmd_processed_inv/, amd_iommu/tlb_inv/' sleep 300

```

### iPerf

We mainly rely on `iPerf-2.0.14` to benchmark IOMMU. You may need to download and build iPerf manually. You can download the source from [here](https://sourceforge.net/projects/iperf2/files/iperf-2.0.14a.tar.gz/download) and then build it, as follows:

```bash
tar -xvf iperf-2.0.14a.tar.gz
cd iperf-2.0.14a
./configure
make
sudo make install
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
[npf-setup]: https://github.com/aliireza/ddio-bench/blob/master/TESTBED.md#network-performance-framework-npf-tool
[npf-readme]: https://github.com/tbarbette/npf/blob/master/README.md
[alireza-page]: https://www.kth.se/profile/farshin/
