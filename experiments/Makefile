# Change the number of runs per experiment
N_RUNS?=5

# To avoid running multiple experiments at once
MAKEFLAGS := -j 1

# NPF Clusters (dut.node and pkt-gen.node should be defined in cluster/)
NPF_CLUSTER?=client=client dut=dut server=dut #Icelake rack30/32

NPF_OUTPUT_NAME?=iommu-results.pdf
NPF_OUTPUT= --output --output-columns x all --max-results --graph-filename ${NPF_OUTPUT_NAME}
NPF_INT_FLAGS := --show-full --rand-env 65536 --build-folder ./npf_build/ ${NPF_OUTPUT} $(NPF_FLAGS)
NPF_INT_FLAGS := $(NPF_INT_FLAGS) #--force-retest #--no-test

# Tools path 
PCM_PATH=$(CURDIR)/../pcm/build/bin/ # PCM (pcm-iio)
EXP_PATH=$(CURDIR) # Current Directory
DUT_PMU_PATH=$(CURDIR)/../pmu-tools/ # pmu-tools 

TOOLS_PATH := 
TOOLS_PATH += PCM_PATH=${PCM_PATH}
TOOLS_PATH += EXP_PATH=${EXP_PATH}
TOOLS_PATH += DUT_PMU_PATH=${DUT_PMU_PATH}

.PHONY: RUN_BATCH clean

RUN_BATCH: 

# Run tests with variables rates (disable IOMMU from grub configuration file)
test_iperf_rate_iommu_off: 
	npf-run local --testie iommu.npf --cluster ${NPF_CLUSTER} --tags iperf pcm pps queue cpu ratetune --config n_runs=${N_RUNS} ${NPF_INT_FLAGS} --variable ${TOOLS_PATH} IOMMU=OFF
	rm -fr test_iperf_rate_iommu_off/
	mkdir -p test_iperf_rate_iommu_off
	mv iommu-results* test_iperf_rate_iommu_off/

# Run tests with variables rates (enable IOMMU from grub configuration file)
test_iperf_rate_iommu_on: 
	npf-run local --testie iommu.npf --cluster ${NPF_CLUSTER} --tags iperf pcm pps queue cpu ratetune --config n_runs=${N_RUNS} ${NPF_INT_FLAGS} --variable ${TOOLS_PATH} IOMMU=ON
	rm -fr test_iperf_rate_iommu_on/
	mkdir -p test_iperf_rate_iommu_on
	mv iommu-results* test_iperf_rate_iommu_on/

# Generate figures with variables rates (run the iommu_off and iommu_on rules first)
test_iperf_rate: 
	npf-run local --testie iommu.npf --cluster ${NPF_CLUSTER} --tags iperf pcm pps queue cpu  ratetune --config n_runs=${N_RUNS} ${NPF_INT_FLAGS} --no-test --variable ${TOOLS_PATH} IOMMU={OFF,ON}
	rm -fr test_iperf_rate/
	mkdir -p test_iperf_rate
	mv iommu-results* test_iperf_rate/


# Run tests with variables MTUs (disable IOMMU from grub configuration file)
test_iperf_mtu_iommu_off: 
	npf-run local --testie iommu.npf --cluster ${NPF_CLUSTER} --tags iperf pcm pps queue cpu mtutune --config n_runs=${N_RUNS} ${NPF_INT_FLAGS} --variable ${TOOLS_PATH} IOMMU=OFF
	rm -fr test_iperf_mtu_iommu_off/
	mkdir -p test_iperf_mtu_iommu_off
	mv iommu-results* test_iperf_mtu_iommu_off/

# Run tests with variables MTUs (enable IOMMU from grub configuration file)
test_iperf_mtu_iommu_on: 
	npf-run local --testie iommu.npf --cluster ${NPF_CLUSTER} --tags iperf pcm pps queue cpu mtutune --config n_runs=${N_RUNS} ${NPF_INT_FLAGS} --variable ${TOOLS_PATH} IOMMU=ON
	rm -fr test_iperf_mtu_iommu_on/
	mkdir -p test_iperf_mtu_iommu_on
	mv iommu-results* test_iperf_mtu_iommu_on/

# Generate figures with variables MTUs (run the iommu_off and iommu_on rules first)
test_iperf_mtu: 
	npf-run local --testie iommu.npf --cluster ${NPF_CLUSTER} --tags iperf pcm pps queue cpu mtutune --config n_runs=${N_RUNS} ${NPF_INT_FLAGS} --no-test --variable ${TOOLS_PATH} IOMMU={OFF,ON}
	rm -fr test_iperf_mtu/
	mkdir -p test_iperf_mtu
	mv iommu-results* test_iperf_mtu/

clean:
	rm -fr testie*
	rm -fr results/